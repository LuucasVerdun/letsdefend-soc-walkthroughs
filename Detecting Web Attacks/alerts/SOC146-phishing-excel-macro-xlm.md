# SOC146 — Phishing Mail Detected — Excel 4.0 Macros (XLM)

> **Plataforma:** LetsDefend  
> **Alert ID / EventID:** #93  
> **Categoria:** Phishing / Malware  
> **Severidade:** Alta  
> **Veredicto:** ✅ True Positive  
> **Data do incidente:** Jun 13, 2021 — 02:11 PM  

---

## 📋 Sumário do Incidente

Um e-mail de phishing foi enviado para o usuário **Lars** (`lars@letsdefend.io`) contendo um anexo ZIP com um arquivo Excel malicioso. O arquivo `.xls` utilizava **Macros XLM (Excel 4.0)** para, ao ser aberto, executar comandos via `regsvr32.exe` e registrar duas DLLs maliciosas no sistema da vítima, estabelecendo comunicação com servidores de C2 externos.

O host da vítima (`LarsPRD`) foi confirmado comprometido após análise dos logs de endpoint e rede, com conexões reais aos IPs de C2 identificadas nos logs do LetsDefend.

---

## 🔍 Metadados do Alerta (SIEM)

| Campo                | Valor                                                        |
|----------------------|--------------------------------------------------------------|
| **EventID**          | 93                                                           |
| **Regra disparada**  | SOC146 — Phishing Mail Detected — Excel 4.0 Macros          |
| **Data/Hora**        | Jun 13, 2021 — 02:11 PM                                      |
| **Remetente**        | trenton@tritowncomputers.com                                 |
| **IP SMTP (origem)** | 24.213.228.54                                                |
| **Destinatário**     | lars@letsdefend.io                                           |
| **Assunto**          | RE: Meeting Notes                                            |
| **Anexo**            | 11f44531fb088d31307d87b01e8eabff.zip                         |
| **Host da vítima**   | LarsPRD                                                      |

---

## 📧 Análise do E-mail

O e-mail foi localizado no **Email Security** do LetsDefend pesquisando pelo endereço do remetente.

```
De:       trenton@tritowncomputers.com
Para:     lars@letsdefend.io
Assunto:  RE: Meeting Notes
Anexo:    11f44531fb088d31307d87b01e8eabff.zip (senha: "infected")
```

> ⚠️ **Red flag:** Anexo ZIP protegido com a senha padrão de malware `infected` — técnica usada para burlar inspeção automática por gateways de e-mail.

---

## 🗂️ Análise do Anexo

### Conteúdo do ZIP

Após extrair o arquivo em ambiente isolado (VM Linux / ParrotOS), foram encontrados **3 arquivos**:

| Arquivo                      | Tipo | SHA256                                                               |
|------------------------------|------|----------------------------------------------------------------------|
| `research-1646684671.xls`    | XLS  | `1df68d55968bb9d2db4d0d18155188a03a442850ff543c8595166ac6987df820`  |
| `iroto.dll`                  | DLL  | `055b9e9af987aec9ba7adb0eef947f39b516a213d663cc52a71c7f0af146a946`  |
| `iroto1.dll`                 | DLL  | `e05c717b43f7e204f315eb8c298f9715791385516335acd8f20ec9e26c3e9b0b`  |

> Hashes calculados com `sha256sum` em ambiente Linux isolado.

---

## 🦠 Análise de Malware

### VirusTotal

Todos os três arquivos foram verificados no VirusTotal e classificados como **maliciosos**:

| Arquivo                      | Detecções VT | Resultado     |
|------------------------------|-------------|---------------|
| `research-1646684671.xls`    | 37 / 72     | ⚠️ Malicioso  |
| `iroto.dll`                  | 19 / 72     | ⚠️ Malicioso  |
| `iroto1.dll`                 | 19 / 72     | ⚠️ Malicioso  |

### Any.run — Análise Dinâmica (Sandbox)

> 🔗 Relatório: [any.run — research-1646684671.xls](https://any.run/report/1df68d55968bb9d2db4d0d18155188a03a442850ff543c8595166ac6987df820/c4210bc2-1ada-411f-a98f-040ac7f3a6f6)

#### Fluxo de execução observado

```
excel.exe
  └─► regsvr32.exe -s iroto.dll      ← Living off the Land
  └─► regsvr32.exe -s iroto1.dll     ← Living off the Land
        └─► Conexão com C2 (HTTPS)
```

#### Por que `regsvr32.exe`?

O `regsvr32.exe` é um binário nativo e assinado do Windows, usado para registrar DLLs. Atacantes abusam dele (**técnica Squiblydoo**) porque:

- Está presente em allowlists de soluções de segurança
- Evita alertas de execução de processo desconhecido
- Permite carregar código malicioso de DLLs externas silenciosamente

#### Por que Macros XLM (Excel 4.0)?

- Tecnologia de **1992** ainda suportada pelo Excel moderno
- Armazenadas em **células de planilhas ocultas** — dificulta análise estática
- Não detectadas por ferramentas de análise VBA (`oletools`)
- Suportam execução automática via `Auto_Open` ao abrir o arquivo
- Usadas por campanhas como **Emotet, Dridex, IcedID e BazarLoader**

---

## 🌐 Indicators of Compromise (IoCs)

### Hashes SHA256

| Arquivo                      | SHA256                                                               |
|------------------------------|----------------------------------------------------------------------|
| `research-1646684671.xls`    | `1df68d55968bb9d2db4d0d18155188a03a442850ff543c8595166ac6987df820`  |
| `iroto.dll`                  | `055b9e9af987aec9ba7adb0eef947f39b516a213d663cc52a71c7f0af146a946`  |
| `iroto1.dll`                 | `e05c717b43f7e204f315eb8c298f9715791385516335acd8f20ec9e26c3e9b0b`  |

### IPs de C2

| IP               | Porta | Confirmado nos logs | Observação          |
|------------------|-------|---------------------|---------------------|
| `188.213.19.81`  | 443   | ✅ Sim              | Logs LetsDefend     |
| `192.232.219.67` | 443   | ✅ Sim              | Logs LetsDefend     |
| `2.16.186.56`    | 80    | Any.run             | Identificado sandbox|

### Domínios de C2

| Domínio                         | Status       |
|---------------------------------|--------------|
| `nws.visionconsulting[.]ro`     | ⚠️ Malicioso |
| `royalpalm.sparkblue[.]lk`      | ⚠️ Malicioso |

### URLs de C2

```
https://royalpalm.sparkblue[.]lk/vCNhYrq3Yg8/dot.html
https://nws.visionconsulting[.]ro/N1G1KCXA/dot.html
```

> ℹ️ Colchetes inseridos nas URLs e domínios para evitar resolução acidental de links.

---

## 💻 Análise do Endpoint — LarsPRD

Verificação realizada via **Endpoint Security** do LetsDefend.

### Processos suspeitos detectados

```
excel.exe → regsvr32.exe -s iroto.dll
excel.exe → regsvr32.exe -s iroto1.dll
```

> `excel.exe` gerando processo filho `regsvr32.exe` é **comportamento altamente anômalo**, indicativo de macro maliciosa em execução.

### Terminal History (comandos executados no host)

```bash
regsvr32.exe -s ../iroto.dll
regsvr32.exe -s ../iroto1.dll
```

### Network Action (conexões de rede confirmadas)

| IP de destino     | Porta | Status               |
|-------------------|-------|----------------------|
| `188.213.19.81`   | 443   | ✅ Conexão confirmada |
| `192.232.219.67`  | 443   | ✅ Conexão confirmada |

### Log Management — Confirmação de Comprometimento

As conexões com os IPs de C2 foram encontradas nos logs de rede do LetsDefend com **ação: allowed** — o tráfego malicioso não foi bloqueado automaticamente, confirmando que o host foi comprometido.

---

## 📋 Respostas ao Playbook

| Pergunta                                          | Resposta                          |
|---------------------------------------------------|-----------------------------------|
| Quando foi enviado o e-mail?                      | Jun 13, 2021, 02:11 PM            |
| Qual o endereço SMTP?                             | 24.213.228.54                     |
| Qual o endereço do remetente?                     | trenton@tritowncomputers.com      |
| Qual o endereço do destinatário?                  | lars@letsdefend.io                |
| O conteúdo do e-mail é suspeito?                  | ✅ Sim                            |
| Há anexos?                                        | ✅ Sim                            |
| O anexo é malicioso?                              | ✅ Sim — Malicioso                |
| O e-mail foi entregue ao usuário?                 | ✅ Sim — Delivered                |
| O usuário abriu o arquivo malicioso?              | ✅ Sim — Opened                   |

---

## 🛡️ Ações de Resposta e Contenção

| Ação                                                  | Status        |
|-------------------------------------------------------|---------------|
| Isolar host `LarsPRD` via Containment (Endpoint)      | ✅ Executado  |
| Bloquear remetente `trenton@tritowncomputers.com`      | ✅ Executado  |
| Bloquear IP SMTP de origem `24.213.228.54`             | ✅ Executado  |
| Adicionar hashes das DLLs e XLS à blocklist AV/EDR    | ✅ Executado  |
| Bloquear IPs de C2 (`188.213.19.81`, `192.232.219.67`) | ✅ Executado  |
| Bloquear domínios de C2 no DNS/proxy                  | ✅ Executado  |
| Registrar artefatos e IoCs no Case Management         | ✅ Executado  |
| Fechar alerta como True Positive                      | ✅ Executado  |

---

## ⚖️ Veredicto Final

```
✅ TRUE POSITIVE — Phishing confirmado com macro XLM maliciosa e host comprometido
```

**Justificativa:**
- E-mail com anexo ZIP protegido pela senha padrão de malware `infected`
- Arquivo `.xls` contém macros XLM que executam `regsvr32.exe` com DLLs maliciosas
- 37 engines do VirusTotal detectaram o arquivo como malicioso
- Sandbox Any.run confirmou comunicação ativa com dois servidores de C2
- Logs de rede do LetsDefend confirmaram conexões reais do host `LarsPRD` com os IPs de C2
- Terminal History do endpoint confirma execução dos comandos `regsvr32.exe -s iroto.dll`

---

## 🎓 Lições Aprendidas

### Técnicas MITRE ATT&CK identificadas

| ID          | Técnica                                          |
|-------------|--------------------------------------------------|
| T1566.001   | Phishing: Spearphishing Attachment               |
| T1137       | Office Application Startup (XLM Macro)           |
| T1218.010   | Signed Binary Proxy Execution: Regsvr32          |
| T1071.001   | Application Layer Protocol: Web Protocols (C2)  |
| T1055       | Process Injection via DLL registrada             |

### Como se proteger

- **Desabilitar macros XLM** via Group Policy (recomendado pela Microsoft desde 2021)
- Ativar regras de **Attack Surface Reduction (ASR)** no Microsoft Defender
- Bloquear macros em arquivos vindos da internet (**Mark of the Web**)
- Configurar gateway de e-mail para **bloquear `.xls` legados** de origem externa
- **Monitorar processos filhos do `excel.exe`** (regsvr32, cmd, powershell, wscript)
- Treinar usuários para nunca abrir anexos com senha `infected` ou similares

---

## 🔗 Ferramentas Utilizadas

| Ferramenta  | Uso                                          | Link                                   |
|-------------|----------------------------------------------|----------------------------------------|
| LetsDefend  | Plataforma SOC / EDR / Log Management        | https://letsdefend.io                  |
| VirusTotal  | Análise de hash dos 3 arquivos               | https://virustotal.com                 |
| Any.run     | Sandbox dinâmica do arquivo XLS              | https://any.run                        |
| AbuseIPDB   | Reputação dos IPs de C2                      | https://abuseipdb.com                  |
| URLScan.io  | Análise das URLs de C2                       | https://urlscan.io                     |
| sha256sum   | Cálculo dos hashes (Linux nativo)            | —                                      |

---

## 📚 Referências

- [MITRE ATT&CK T1566.001 — Spearphishing Attachment](https://attack.mitre.org/techniques/T1566/001/)
- [MITRE ATT&CK T1218.010 — Regsvr32](https://attack.mitre.org/techniques/T1218/010/)
- [MITRE ATT&CK T1137 — Office Application Startup](https://attack.mitre.org/techniques/T1137/)
- [MalwareBytes Labs — Excel 4.0 XLM Macros](https://www.malwarebytes.com/blog/news/2020/06/malspam-excel-4-macros)
- [Microsoft — Blocking Excel XLM macros by default](https://techcommunity.microsoft.com/t5/excel-blog/microsoft-excel-blocking-xll-add-ins-by-default/ba-p/3738690)
- [Any.run — Relatório da análise do research-1646684671.xls](https://any.run/report/1df68d55968bb9d2db4d0d18155188a03a442850ff543c8595166ac6987df820/c4210bc2-1ada-411f-a98f-040ac7f3a6f6)

---

*Writeup elaborado como parte do treinamento SOC no LetsDefend.*  
*Plataforma: LetsDefend.io | Alert: SOC146 — EventID #93*
