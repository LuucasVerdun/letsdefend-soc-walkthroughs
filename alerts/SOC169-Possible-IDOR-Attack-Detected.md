# 🛡️ LetsDefend — SOC169: Possible IDOR Attack Detected

> **Plataforma:** [LetsDefend](https://app.letsdefend.io/)  
> **Categoria:** Blue Team / SOC Analysis  
> **Tipo de Ataque:** IDOR (Insecure Direct Object Reference)  
> **Resultado:** ✅ True Positive — Escalado para Tier 2  
> **Event ID:** 119  

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
- [Como o Ataque IDOR Funciona](#como-o-ataque-idor-funciona)
- [IOCs (Indicators of Compromise)](#iocs-indicators-of-compromise)
- [Conclusão](#conclusão)
- [Lições Aprendidas](#lições-aprendidas)
- [Referências](#referências)

---

## Visão Geral

Este writeup documenta a investigação do alerta **SOC169 — EventID 119**, disparado pela detecção de requisições consecutivas ao mesmo endpoint com parâmetros manipulados, indicando uma possível tentativa de **IDOR (Insecure Direct Object Reference)**.

A investigação confirmou que o ataque foi bem-sucedido: um agente externo proveniente dos EUA realizou múltiplas requisições ao endpoint `/get_user_info/`, alterando o parâmetro `user_id` para acessar dados de diferentes usuários sem autorização.

---

## Conceitos Fundamentais

### O que é um SOC?

Um **Security Operations Center (SOC)** é o centro de comando da defesa cibernética de uma organização. Ele monitora atividades digitais, detecta e analisa ameaças, e toma ações para proteger e responder a incidentes de segurança — combinando tecnologia, processos e profissionais especializados.

### O que é IDOR?

**IDOR (Insecure Direct Object Reference)** é uma vulnerabilidade comum em aplicações web. Funciona de forma similar a um armário de academia com cadeados numerados:

> Imagine que cada usuário tem um armário com número único e uma chave especial. Se alguém descobrir que pode **alterar o número do armário na chave**, consegue abrir o armário de outra pessoa e acessar seus pertences.

Em aplicações web, quando um usuário faz uma requisição (como acessar informações de conta), o sistema usa **identificadores** (como números) para determinar quais dados exibir. Se a aplicação não verifica corretamente se o usuário tem permissão para acessar aquele identificador, um atacante pode manipulá-lo para visualizar ou roubar dados de outra pessoa.

**Exemplo prático:**

```
Requisição legítima:   GET /get_user_info/?user_id=123
Requisição maliciosa:  GET /get_user_info/?user_id=456
                                                   ^^^
                                          ID alterado pelo atacante
```

---

## Detalhes do Alerta

| Campo | Valor |
|---|---|
| **Event ID** | 119 |
| **Alerta** | Possible IDOR Attack Detected |
| **Gatilho** | Requisições consecutivas ao mesmo endpoint com parâmetros variados |
| **IP de Origem** | Externo — EUA (flagged como malicioso) |
| **IP de Destino** | `172.16.17[.]15` (servidor interno) |
| **URL Alvo** | `hxxps://172.16.17[.]15/get_user_info/` |
| **Parâmetro Manipulado** | `user_id=X` (iterado pelo atacante) |
| **Eventos Encontrados** | 5 eventos nos logs |

---

## Processo de Investigação

### 1. Fila de Tickets SOC

O primeiro passo é verificar a fila de tickets no **Main Channel** da seção de Monitoramento. A fila organiza alertas por severidade, garantindo triagem adequada e rastreamento de incidentes.

**Caminho:** `Practice → Monitoring → Main Channel`

---

### 2. Assumir Responsabilidade

O **EventID 119** foi selecionado para investigação. Assumir a propriedade do ticket garante accountability e evita sobreposição de trabalho entre analistas.

---

### 3. Criar Caso

Um caso formal foi criado para o EventID 119 clicando no ícone de ação correspondente. Criar um caso permite priorizar o incidente com base em sua severidade e impacto potencial.

---

### 4. Utilizar o Playbook

Os **Playbooks** garantem consistência e padronização nas respostas a incidentes. Seguindo o playbook definido pelo SOC, cada etapa da investigação é conduzida de forma estruturada e documentada.

---

### 5. Detecção

**Objetivo:** Compreender o que disparou o alerta e verificar se é um True Positive ou False Positive.

O alerta foi disparado por **requisições consecutivas ao mesmo endpoint**:

```
hxxps://172.16.17[.]15/get_user_info/
```

A repetição de requisições com variações no parâmetro `user_id` é um padrão clássico de enumeração em ataques IDOR.

---

### 6. Análise

#### Reputação do IP de Origem

O IP de origem foi verificado em fontes de threat intelligence:

| Ferramenta | Resultado |
|---|---|
| **VirusTotal** | 1 fornecedor de AV sinalizou o IP como malicioso |
| **AbuseIPDB** | IP flagged como malicioso |

**Detalhes do IP:**

| Campo | Valor |
|---|---|
| **Origem** | Externa — EUA |
| **Classificação** | Malicioso (confirmado por múltiplas fontes) |
| **Direção do tráfego** | EUA → Rede interna da empresa |

#### Análise dos Logs de Rede

A query nos logs de rede pelo IP malicioso retornou **5 eventos**. A análise dos logs revelou o padrão de ataque:

| Evento | Parâmetro Enviado | Status HTTP | Observação |
|---|---|---|---|
| 1 | `user_id=1` | 200 OK | ✅ Sucesso — dados retornados |
| 2 | `user_id=2` | 200 OK | ✅ Sucesso — dados retornados |
| 3 | `user_id=3` | 200 OK | ✅ Sucesso — dados retornados |
| 4 | `user_id=4` | 200 OK | ✅ Sucesso — dados retornados |
| 5 | `user_id=X` | — | ⚠️ Evento que disparou o alerta |

O atacante realizou **enumeração de IDs de usuário**, obtendo respostas HTTP 200 (sucesso) em múltiplas requisições — confirmando que dados de diferentes usuários foram acessados sem autorização.

#### Verificação de Ataque Planejado

Foi verificado na **Email Security** se havia alguma comunicação indicando que o teste foi planejado internamente. Nenhuma informação nesse sentido foi encontrada.

#### Determinações Finais

| Pergunta | Resposta |
|---|---|
| Tipo de ataque | IDOR (Insecure Direct Object Reference) |
| Foi planejado internamente? | Não — nenhuma evidência encontrada |
| Origem | Externa (EUA → servidor web interno) |
| A requisição foi bem-sucedida? | ✅ Sim — múltiplas respostas HTTP 200 |
| Ataque confirmado? | ✅ Sim — True Positive |

---

### 7. Contenção

Dado que o ataque foi confirmado como bem-sucedido, o endpoint do servidor afetado foi **contido imediatamente** para prevenir danos adicionais.

**Status:** `Servidor → CONTAINED`

A contenção limita o impacto do incidente, protege dados e operações, e preserva evidências para análise forense.

---

### 8. Remediação

As seguintes ações de remediação foram recomendadas:

- **Implementar verificações de autorização** para garantir que o usuário solicitante tem permissão para acessar o recurso referenciado.
- **Substituir IDs previsíveis** por referências indiretas ou tokens opacos que não possam ser facilmente enumerados ou manipulados.
- **Implementar validação de input** e mecanismos de controle de acesso no nível do servidor.
- **Proteger adequadamente as aplicações web** com revisão de código focada em controles de acesso.
- **Implementar rate limiting** para detectar e bloquear padrões de enumeração automatizada.
- Escalar o ticket para **Tier 2** para análise aprofundada e verificação dos dados expostos.

---

### 9. Registro de Artefatos e IOCs

Os achados foram documentados na seção **Analyst Note** do caso, incluindo:

- Raciocínio da investigação
- Evidências dos logs (5 eventos, múltiplas respostas HTTP 200)
- Confirmação do sucesso do ataque via análise do status code
- Passos executados durante a análise

O ticket foi escalado para **Tier 2** devido ao sucesso confirmado do ataque e à exposição de dados de múltiplos usuários.

---

### 10. Encerramento do Ticket

**Classificação Final:** ✅ **True Positive**

O alerta foi encerrado na aba **Investigation Channel** após a conclusão de todas as etapas do playbook. O ticket foi movido para **Closed Alerts**.

---

## Como o Ataque IDOR Funciona

```
[Atacante]
    │
    │  GET /get_user_info/?user_id=1  → HTTP 200 ✅ (dados do usuário 1)
    │  GET /get_user_info/?user_id=2  → HTTP 200 ✅ (dados do usuário 2)
    │  GET /get_user_info/?user_id=3  → HTTP 200 ✅ (dados do usuário 3)
    │  GET /get_user_info/?user_id=4  → HTTP 200 ✅ (dados do usuário 4)
    │                ...
    ▼
[Servidor sem controle de autorização]
    │
    └── Retorna dados de qualquer user_id solicitado
        sem verificar se o solicitante tem permissão
```

**Fluxo correto (com proteção):**

```
[Atacante]
    │
    │  GET /get_user_info/?user_id=456
    ▼
[Servidor com controle de autorização]
    │
    ├── Verifica: "O usuário autenticado é o dono do user_id=456?"
    │
    └── Não → HTTP 403 Forbidden ❌
```

---

## IOCs (Indicators of Compromise)

| Tipo | Valor | Observação |
|---|---|---|
| IP Malicioso | Externo — EUA | Flagged no VirusTotal e AbuseIPDB |
| URL Alvo | `hxxps://172.16.17[.]15/get_user_info/` | Endpoint vulnerável a IDOR |
| Parâmetro Explorado | `user_id=X` | Enumerado sequencialmente pelo atacante |
| Servidor Comprometido | IP interno: `172.16.17[.]15` | Contido após confirmação do ataque |

> ⚠️ **Nota:** IPs e URLs foram defangeados (com `[.]`) para evitar cliques acidentais.

---

## Conclusão

Este alerta demonstrou um caso claro de **IDOR bem-sucedido**. Um agente externo proveniente dos EUA explorou um endpoint da aplicação web que não implementava verificações de autorização adequadas, conseguindo acessar dados de múltiplos usuários simplesmente alterando o parâmetro `user_id` nas requisições.

A presença de múltiplas respostas HTTP 200 nos logs confirmou que os dados foram efetivamente entregues ao atacante para cada ID enumerado, justificando a contenção imediata do servidor e a escalação para Tier 2.

O fluxo completo da investigação seguiu o playbook SOC:

```
Detecção → Análise de IP → Log Management → Confirmação → Contenção → Remediação → Escalação → Encerramento
```

---

## Lições Aprendidas

- **Controle de autorização é inegociável:** Toda requisição a dados sensíveis deve verificar se o usuário autenticado tem permissão para acessar o recurso específico — não apenas se está autenticado.
- **IDs sequenciais são um risco:** Utilizar IDs numéricos previsíveis facilita a enumeração por atacantes. Prefira UUIDs ou tokens opacos.
- **HTTP 200 é uma evidência crítica:** O status code de resposta nos logs é determinante para confirmar se um ataque IDOR foi bem-sucedido.
- **Rate limiting detecta enumeração:** Múltiplas requisições consecutivas ao mesmo endpoint com parâmetros variados são um padrão identificável que pode ser bloqueado automaticamente.
- **Threat Intelligence acelera decisões:** A combinação de VirusTotal e AbuseIPDB confirmou rapidamente a natureza maliciosa do IP de origem.
- **Email Security como fonte de contexto:** Verificar se o teste foi planejado internamente é uma etapa essencial para evitar containment desnecessário.

---

## Referências

- [LetsDefend Platform](https://app.letsdefend.io/)
- [OWASP — Insecure Direct Object Reference (IDOR)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [OWASP Top 10 — Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [VirusTotal](https://www.virustotal.com/)
- [AbuseIPDB](https://www.abuseipdb.com/)
- [MITRE ATT&CK — Exploitation for Credential Access (T1212)](https://attack.mitre.org/techniques/T1212/)

---

*Writeup baseado no artigo original de [Tijan Hydara (@topcyberdawg)](https://medium.com/@topcyberdawg) no Medium.*
