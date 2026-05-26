# 🛡️ LetsDefend — SOC168: Whoami Command Detected in Request Body

> **Plataforma:** [LetsDefend](https://app.letsdefend.io/)  
> **Categoria:** Blue Team / SOC Analysis  
> **Tipo de Ataque:** Command Injection  
> **Resultado:** ✅ True Positive — Escalado para Tier 2  
> **Data do Alerta:** 28 de Fevereiro de 2022, 04:12 AM  

---

## 📋 Sumário

- [Visão Geral](#visão-geral)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Detalhes do Alerta](#detalhes-do-alerta)
- [Processo de Investigação](#processo-de-investigação)
  - [1. Fila de Tickets SOC](#1-fila-de-tickets-soc)
  - [2. Assumir Responsabilidade](#2-assumir-responsabilidade)
  - [3. Criar Caso](#3-criar-caso)
  - [4. Utilizar o Playbook](#4-utilizar-o-playbook)
  - [5. Detecção](#5-detecção)
  - [6. Análise](#6-análise)
  - [7. Contenção](#7-contenção)
  - [8. Remediação](#8-remediação)
  - [9. Registro de Artefatos e IOCs](#9-registro-de-artefatos-e-iocs)
  - [10. Encerramento do Ticket](#10-encerramento-do-ticket)
- [Comandos Identificados](#comandos-identificados)
- [IOCs (Indicators of Compromise)](#iocs-indicators-of-compromise)
- [Conclusão](#conclusão)
- [Lições Aprendidas](#lições-aprendidas)
- [Referências](#referências)

---

## Visão Geral

Este writeup documenta a investigação do alerta **SOC168 — EventID 118**, disparado pela detecção do comando `whoami` no corpo de uma requisição HTTP. O alerta indicou uma possível tentativa de **Command Injection** em um servidor web interno.

A investigação confirmou que o ataque foi bem-sucedido: um agente externo, localizado na China, conseguiu executar comandos no servidor `WebServer1004` por meio de parâmetros POST manipulados.

---

## Conceitos Fundamentais

### O que é um SOC?

Um **Security Operations Center (SOC)** é o centro de comando da defesa cibernética de uma organização. Ele monitora atividades digitais, detecta e analisa ameaças, e toma ações para proteger e responder a incidentes de segurança — combinando tecnologia, processos e profissionais especializados.

### O que é Command Injection?

**Command Injection** é um tipo de ataque no qual o agente malicioso insere comandos do sistema operacional em campos de entrada de uma aplicação web vulnerável. Se a aplicação não faz a devida sanitização dos dados recebidos, esses comandos são executados diretamente no servidor.

### O que é o comando `whoami`?

O comando `whoami` é um utilitário de linha de comando disponível em sistemas Unix/Linux, macOS e Windows. Ele exibe o nome do usuário atualmente logado no sistema — sendo frequentemente usado por atacantes para fazer reconhecimento inicial após obter execução de código.

---

## Detalhes do Alerta

| Campo | Valor |
|---|---|
| **Event ID** | 118 |
| **Alerta** | Whoami Command Detected in Request Body |
| **Data/Hora** | 28 de Fevereiro de 2022, 04:12 AM |
| **IP de Origem** | `61.177.172[.]87` |
| **IP de Destino** | `172.16.17[.]16` (WebServer1004) |
| **URL Alvo** | `hxxps://172.16.17.16/video/` |
| **Método HTTP** | POST |
| **Endpoint Afetado** | WebServer1004 |

---

## Processo de Investigação

### 1. Fila de Tickets SOC

O primeiro passo é verificar a fila de tickets no **Main Channel** da seção de Monitoramento. A fila organiza alertas por severidade, garantindo triagem adequada e rastreamento de incidentes.

**Caminho:** `Practice → Monitoring → Main Channel`

---

### 2. Assumir Responsabilidade

O **EventID 118** foi selecionado para investigação. Assumir a propriedade do ticket garante accountability e evita sobreposição de trabalho entre analistas.

---

### 3. Criar Caso

Um caso formal foi criado para o EventID 118 clicando no ícone de ação correspondente. Criar um caso permite priorizar o incidente com base em sua severidade e impacto potencial.

---

### 4. Utilizar o Playbook

Os **Playbooks** garantem consistência e padronização nas respostas a incidentes. Seguindo o playbook definido pelo SOC, cada etapa da investigação é conduzida de forma estruturada e documentada.

---

### 5. Detecção

**Objetivo:** Determinar se o alerta é um **True Positive** ou **False Positive**.

**Ações realizadas:**

- Acesso ao **Log Management** e filtragem pelo IP de destino do alerta.
- Análise dos logs brutos do firewall direcionados ao IP de origem.
- Identificação de **POST Parameters** contendo comandos de sistema operacional (`whoami`, `ls`).

**Observação:** A variação no tamanho da resposta HTTP entre as requisições indicou que a manipulação dos parâmetros foi bem-sucedida.

**URL analisada:** `hxxps://172.16.17.16/video/`

**Query executada** retornou **5 eventos**, todos originados do IP:

```
61.177.172[.]87
```

---

### 6. Análise

#### Reputação do IP de Origem

O IP `61.177.172[.]87` foi verificado em fontes de threat intelligence:

- **VirusTotal:** 3 fornecedores de AV sinalizaram o IP como malicioso.
- **Cisco Talos:** Confirmou o IP como malicioso.

**Detalhes do IP:**

| Campo | Valor |
|---|---|
| **Tipo** | Endereço Estático |
| **ISP** | ChinaNet Jiangsu Province Network |
| **Uso** | Data Center |
| **Domínio** | chinatelecom.com.cn |
| **País** | China |
| **Cidade** | Lianyungang, Jiangsu |

#### Análise dos Logs Raw

A análise dos 5 logs brutos revelou uma sequência progressiva de comandos injetados nos POST Parameters:

| Requisição | Comando Injetado | Objetivo do Atacante |
|---|---|---|
| 1 | `whoami` | Identificar o usuário atual do sistema |
| 2 | `ls` | Listar arquivos do diretório corrente |
| 3 | `uname` | Obter informações do sistema operacional |
| 4 | `cat /etc/passwd` | Listar usuários do sistema |
| 5 | `cat /etc/shadow` | Acessar senhas criptografadas |

#### Verificação de Sucesso do Ataque

A busca por `WebServer1004` na seção **Endpoint Management** revelou:

- ✅ **Terminal History:** Todos os comandos estavam presentes no histórico do servidor.
- ✅ **Process History:** Os processos correspondentes foram confirmados.

**Conclusão:** O ataque foi bem-sucedido. Os comandos foram executados no servidor.

#### Determinações Finais

| Pergunta | Resposta |
|---|---|
| Tipo de ataque | Command Injection |
| Foi planejado internamente? | Não — IP externo malicioso |
| Origem | Externa (China → Servidor Web interno) |
| Ataque foi bem-sucedido? | ✅ Sim |

---

### 7. Contenção

Dado que o ataque foi confirmado como bem-sucedido, o endpoint `WebServer1004` foi **contido imediatamente** para prevenir danos adicionais.

**Status:** `WebServer1004 → CONTAINED`

A contenção limita o impacto do incidente, protege dados e operações, e preserva evidências para análise forense.

---

### 8. Remediação

As seguintes ações de remediação foram recomendadas:

- Escalar o ticket para um analista **Tier 2** para análise aprofundada.
- Resetar todas as credenciais de usuário potencialmente comprometidas.
- Implementar senhas fortes e políticas de autenticação robustas.
- Revisar e corrigir a vulnerabilidade de Command Injection na aplicação web.
- Implementar validação e sanitização de inputs no servidor.

---

### 9. Registro de Artefatos e IOCs

Os achados foram documentados na seção **Analyst Note** do caso, incluindo:

- Raciocínio da investigação
- Dados e observações relevantes
- Passos executados durante a análise

O ticket foi escalado para **Tier 2** devido ao sucesso confirmado do ataque.

---

### 10. Encerramento do Ticket

**Classificação Final:** ✅ **True Positive**

O alerta foi encerrado na aba **Investigation Channel** após a conclusão de todas as etapas do playbook. O ticket foi movido para **Closed Alerts**.

---

## Comandos Identificados

| Comando | Sistema | Descrição |
|---|---|---|
| `whoami` | Unix/Linux/Windows | Exibe o nome do usuário atualmente logado |
| `ls` | Unix/Linux | Lista os arquivos e diretórios do diretório atual |
| `uname` | Unix/Linux | Exibe informações do sistema operacional e hardware |
| `cat /etc/passwd` | Unix/Linux | Exibe o conteúdo do arquivo com registros de usuários do sistema |
| `cat /etc/shadow` | Unix/Linux | Exibe o conteúdo do arquivo com senhas criptografadas dos usuários (requer root) |

---

## IOCs (Indicators of Compromise)

| Tipo | Valor | Observação |
|---|---|---|
| IP Malicioso | `61.177.172[.]87` | China — Data Center — Flagged por múltiplos AVs |
| URL Alvo | `hxxps://172.16.17[.]16/video/` | Endpoint vulnerável à Command Injection |
| Servidor Comprometido | `WebServer1004` | IP interno: `172.16.17[.]16` |

> ⚠️ **Nota:** IPs e URLs foram defangeados (com `[.]`) para evitar cliques acidentais.

---

## Conclusão

Este alerta demonstrou um caso claro de **Command Injection bem-sucedido**. Um agente externo proveniente da China explorou um endpoint vulnerável no servidor web interno, executando uma série de comandos de reconhecimento — desde a simples identificação do usuário (`whoami`) até a tentativa de acessar o arquivo de senhas do sistema (`/etc/shadow`).

A investigação seguiu o fluxo completo do playbook SOC:

```
Detecção → Análise → Contenção → Remediação → Escalação → Encerramento
```

A presença dos comandos tanto no histórico de terminal quanto no histórico de processos do `WebServer1004` confirmou o sucesso do ataque, justificando a contenção imediata do endpoint e a escalação para Tier 2.

---

## Lições Aprendidas

- **Validação de inputs é crítica:** Aplicações web que recebem dados do usuário devem sempre sanitizar e validar os inputs antes de qualquer processamento no servidor.
- **Monitoramento de POST Parameters:** Ataques de Command Injection frequentemente ocorrem em parâmetros menos visíveis, como os do corpo de requisições POST.
- **Threat Intelligence é essencial:** A reputação do IP foi confirmada rapidamente usando VirusTotal e Cisco Talos, agilizando a tomada de decisão.
- **Histórico de comandos como evidência:** O Terminal History e o Process History do endpoint foram determinantes para confirmar o sucesso do ataque.
- **Contenção rápida reduz dano:** Isolar o servidor imediatamente após a confirmação do ataque é fundamental para limitar o impacto.

---

## Referências

- [LetsDefend Platform](https://app.letsdefend.io/)
- [OWASP — Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [Cisco Talos Intelligence](https://talosintelligence.com/)
- [VirusTotal](https://www.virustotal.com/)
- [MITRE ATT&CK — OS Command Injection (T1059)](https://attack.mitre.org/techniques/T1059/)

---

*Writeup baseado no artigo original de [Tijan Hydara (@topcyberdawg)](https://medium.com/@topcyberdawg) no Medium.*
