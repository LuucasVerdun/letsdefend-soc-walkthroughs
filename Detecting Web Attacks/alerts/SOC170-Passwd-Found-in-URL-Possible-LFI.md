# 🛡️ LetsDefend — SOC170: Passwd Found in Requested URL — Possible LFI Attack

> **Plataforma:** [LetsDefend](https://app.letsdefend.io/)  
> **Categoria:** Blue Team / SOC Analysis  
> **Tipo de Ataque:** LFI (Local File Inclusion) — Tentativa  
> **Resultado:** ✅ True Positive — Ataque não bem-sucedido — Sem contenção necessária  
> **Event ID:** 120  
> **Data do Alerta:** 01 de Março de 2022, 10:10 AM  

---

## 📋 Sumário

- [Visão Geral](#visão-geral)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Detalhes do Alerta](#detalhes-do-alerta)
- [Processo de Investigação](#processo-de-investigação)
  - [1. Análise dos Logs — Log Management](#1-análise-dos-logs--log-management)
  - [2. Análise da URL Maliciosa](#2-análise-da-url-maliciosa)
  - [3. Verificação do Status HTTP](#3-verificação-do-status-http)
  - [4. Reputação do IP de Origem](#4-reputação-do-ip-de-origem)
  - [5. Determinações Finais](#5-determinações-finais)
  - [6. Encerramento do Ticket](#6-encerramento-do-ticket)
- [Anatomia do Ataque LFI](#anatomia-do-ataque-lfi)
- [IOCs (Indicators of Compromise)](#iocs-indicators-of-compromise)
- [Conclusão](#conclusão)
- [Lições Aprendidas](#lições-aprendidas)
- [Referências](#referências)

---

## Visão Geral

Este writeup documenta a investigação do alerta **SOC170 — EventID 120**, disparado pela detecção da string `passwd` na URL de uma requisição HTTPS. O alerta indicou uma possível tentativa de **LFI (Local File Inclusion)** contra o servidor `WebServer1006`.

A investigação confirmou que a tentativa de ataque foi real (**True Positive**), porém **não foi bem-sucedida**: o servidor retornou um erro HTTP 500 (Internal Server Error) sem nenhum dado na resposta, indicando que o arquivo `/etc/passwd` não foi entregue ao atacante.

---

## Conceitos Fundamentais

### O que é LFI (Local File Inclusion)?

**LFI (Local File Inclusion)** é uma vulnerabilidade em aplicações web que permite a um atacante forçar o servidor a incluir e expor arquivos locais do sistema de arquivos do servidor. O ataque ocorre quando a aplicação aceita parâmetros de entrada que referenciam arquivos sem validação adequada.

**Exemplo de exploração:**

```
URL normal:    https://site.com/?file=pagina.html
URL maliciosa: https://site.com/?file=../../../../etc/passwd
```

Utilizando **path traversal** (`../`), o atacante navega pelos diretórios do servidor para alcançar arquivos sensíveis fora do diretório raiz da aplicação.

### O que é o arquivo `/etc/passwd`?

O arquivo `/etc/passwd` é um arquivo de texto presente em sistemas Unix/Linux que contém informações sobre todos os usuários registrados no sistema, incluindo nome de usuário, UID/GID, diretório home e shell padrão. Expô-lo fornece ao atacante uma lista completa de usuários para ataques subsequentes.

### O que é Path Traversal?

**Path Traversal** é a técnica usada para navegar pela estrutura de diretórios de um servidor usando sequências `../`. Cada `../` sobe um nível na árvore de diretórios:

```
/var/www/html/    ← diretório raiz da aplicação
    ↑ ../
/var/www/
    ↑ ../
/var/
    ↑ ../
/                 ← raiz do sistema
  └── etc/
        └── passwd  ← arquivo alvo
```

---

## Detalhes do Alerta

| Campo | Valor |
|---|---|
| **Event ID** | 120 |
| **Regra** | SOC170 - Passwd Found in Requested URL - Possible LFI Attack |
| **Data/Hora** | 01 de Março de 2022, 10:10 AM |
| **Hostname** | WebServer1006 |
| **IP de Origem** | `106.55.45[.]162` |
| **IP de Destino** | `172.16.17[.]13` (WebServer1006) |
| **Método HTTP** | GET |
| **URL Requisitada** | `hxxps://172.16.17[.]13/?file=../../../../etc/passwd` |
| **User-Agent** | `Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 1.1.4322)` |
| **Gatilho do Alerta** | URL contém a string `passwd` |
| **Ação do Dispositivo** | Allowed (permitida pelo firewall) |

> ⚠️ **Nota sobre o User-Agent:** O User-Agent `MSIE 6.0` (Internet Explorer 6) indica um navegador de 2001. É comum em ferramentas automatizadas de ataque que usam user-agents antigos para tentar evadir detecções modernas.

---

## Processo de Investigação

### 1. Análise dos Logs — Log Management

O primeiro passo foi acessar o **Log Management** e inspecionar o endpoint `WebServer1006`, filtrando pelas comunicações entre o servidor e o IP de origem `106.55.45[.]162`.

A análise dos logs confirmou que o IP suspeito tentou acessar a URL maliciosa, navegando pela estrutura de diretórios com o objetivo de alcançar o arquivo `/etc/passwd`.

---

### 2. Análise da URL Maliciosa

A URL requisitada foi:

```
https://172.16.17.13/?file=../../../../etc/passwd
```

**Decomposição da técnica de Path Traversal:**

| Segmento | Significado |
|---|---|
| `?file=` | Parâmetro vulnerável que a aplicação usa para incluir arquivos |
| `../../../../` | Quatro níveis de path traversal para subir ao diretório raiz `/` |
| `etc/passwd` | Arquivo alvo contendo informações de usuários do sistema |

---

### 3. Verificação do Status HTTP

A análise dos logs revelou dois dados críticos sobre a resposta do servidor:

| Campo | Valor | Interpretação |
|---|---|---|
| **HTTP Response Size** | `0` | Nenhum dado foi retornado na resposta |
| **HTTP Response Status** | `500` | Internal Server Error — o servidor encontrou um erro ao processar a requisição |

**Conclusão:** O servidor não entregou o conteúdo do arquivo `/etc/passwd` ao atacante. O erro HTTP 500 indica que a tentativa **falhou**.

---

### 4. Reputação do IP de Origem

O IP `106.55.45[.]162` foi verificado em fontes de threat intelligence:

| Ferramenta | Resultado |
|---|---|
| **VirusTotal** | IP flagged como malicioso — origem: **China** |
| **Cisco Talos** | Confirmação adicional da natureza maliciosa do IP |

**Detalhes do IP:**

| Campo | Valor |
|---|---|
| **País de Origem** | China |
| **Classificação** | Malicioso |
| **Direção do tráfego** | Internet (externo) → Rede local (interno) |

---

### 5. Determinações Finais

| Pergunta | Resposta |
|---|---|
| Tipo de ataque | LFI (Local File Inclusion) com Path Traversal |
| É um True Positive? | ✅ Sim — a tentativa de ataque é real |
| O ataque foi bem-sucedido? | ❌ Não — HTTP 500, resposta vazia |
| É necessário conter o servidor? | ❌ Não — ataque não causou comprometimento |
| É necessário escalar para Tier 2? | ❌ Não — ataque bloqueado pelo servidor |
| Origem | Externa (China → rede local) |

---

### 6. Encerramento do Ticket

**Classificação Final:** ✅ **True Positive** (ataque real, porém não bem-sucedido)

O alerta foi encerrado após a confirmação de que a tentativa de LFI é legítima, o ataque não foi bem-sucedido (HTTP 500, sem dados expostos) e não há necessidade de contenção ou escalação.

---

## Anatomia do Ataque LFI

```
[Atacante - China]
        │
        │  GET /?file=../../../../etc/passwd
        │
        ▼
[WebServer1006 - 172.16.17.13]
        │
        ├── Recebe parâmetro: file=../../../../etc/passwd
        │
        ├── Tenta navegar: /var/www/html/ → /var/www/ → /var/ → / → /etc/passwd
        │
        └── ERRO 500 ❌ — arquivo não retornado
                │
                └── HTTP Response Size: 0
                    Nenhum dado exposto ao atacante
```

**Se o servidor fosse vulnerável, o fluxo seria:**

```
[Atacante]
        │
        │  GET /?file=../../../../etc/passwd
        ▼
[Servidor vulnerável]
        │
        └── HTTP 200 ✅
            Conteúdo do /etc/passwd retornado:
            root:x:0:0:root:/root:/bin/bash
            daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
            ...
```

---

## IOCs (Indicators of Compromise)

| Tipo | Valor | Observação |
|---|---|---|
| IP Malicioso | `106.55.45[.]162` | China — flagged no VirusTotal e Cisco Talos |
| URL Maliciosa | `hxxps://172.16.17[.]13/?file=../../../../etc/passwd` | Tentativa de LFI com path traversal |
| Arquivo Alvo | `/etc/passwd` | Arquivo de usuários do sistema Unix/Linux |
| Parâmetro Explorado | `?file=` | Parâmetro vulnerável a LFI |
| User-Agent Suspeito | `Mozilla/4.0 (MSIE 6.0; Windows NT 5.1)` | IE6 — indicativo de ferramenta automatizada |
| Servidor Alvo | `WebServer1006` — IP: `172.16.17[.]13` | Não comprometido — sem necessidade de contenção |

> ⚠️ **Nota:** IPs e URLs foram defangeados (com `[.]`) para evitar cliques acidentais.

---

## Conclusão

O alerta SOC170 demonstrou um caso importante: **um ataque pode ser True Positive sem ser bem-sucedido**. A tentativa de LFI foi real e deliberada — o atacante da China usou path traversal para tentar acessar o `/etc/passwd` do `WebServer1006`. No entanto, o servidor retornou HTTP 500 com resposta vazia, indicando que o arquivo não foi exposto.

Este cenário evidencia a importância de confirmar não apenas **se houve ataque**, mas **se o ataque teve sucesso** — e de basear as decisões de contenção e escalação no **impacto real**, não apenas na intenção.

O fluxo da investigação foi:

```
Alerta → Log Management → Análise da URL → HTTP Status → Threat Intel → Encerramento
```

---

## Lições Aprendidas

- **HTTP Status Code é evidência crítica:** A diferença entre HTTP 200 e HTTP 500 determina se um ataque LFI foi bem-sucedido. Sempre verificar o status code e o tamanho da resposta nos logs.
- **True Positive ≠ Ataque bem-sucedido:** Um alerta pode ser legítimo e ainda assim não resultar em comprometimento. A contenção deve ser proporcional ao impacto real.
- **Path Traversal é simples mas perigoso:** A sequência `../` é básica, mas devastadora em aplicações sem validação de input. Sempre sanitizar parâmetros de arquivo.
- **User-Agent como indicador:** User-Agents muito antigos (como IE6) em contextos modernos são fortes indicativos de ferramentas automatizadas de ataque.
- **Nem todo True Positive exige escalação:** Ataques bloqueados que não resultaram em comprometimento podem ser encerrados sem escalação para Tier 2, desde que bem documentados.

---

## Referências

- [LetsDefend Platform](https://app.letsdefend.io/)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [OWASP — Testing for Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)
- [VirusTotal](https://www.virustotal.com/)
- [Cisco Talos Intelligence](https://talosintelligence.com/)
- [MITRE ATT&CK — File and Directory Discovery (T1083)](https://attack.mitre.org/techniques/T1083/)

---

*Writeup baseado no artigo original de [2O0K](https://medium.com/@2O0K) no Medium.*
