# 🛡️ LetsDefend — SOC165: Possible SQL Injection Payload Detected

> **Plataforma:** [LetsDefend](https://app.letsdefend.io/)  
> **Tipo de alerta:** SQL Injection  
> **Event ID:** 115  
> **Resultado:** True Positive ✅  
> **Dificuldade:** Intermediário

---

## 📋 Índice

1. [Visão Geral](#visão-geral)
2. [Conceitos Fundamentais](#conceitos-fundamentais)
3. [Fluxo de Investigação](#fluxo-de-investigação)
4. [Passo a Passo Detalhado](#passo-a-passo-detalhado)
   - [Task 1 – Verificar a fila de tickets do SOC](#task-1--verificar-a-fila-de-tickets-do-soc)
   - [Task 2 – Assumir responsabilidade pelo alerta](#task-2--assumir-responsabilidade-pelo-alerta)
   - [Task 3 – Criar o caso](#task-3--criar-o-caso)
   - [Task 4 – Utilizar o Playbook](#task-4--utilizar-o-playbook)
   - [Task 5 – Detecção](#task-5--detecção)
   - [Task 6 – Análise](#task-6--análise)
   - [Task 7 – Contenção](#task-7--contenção)
   - [Task 8 – Remediação](#task-8--remediação)
   - [Task 9 – Registrar Artefatos e IOCs](#task-9--registrar-artefatos-e-iocs)
   - [Task 10 – Encerrar o ticket](#task-10--encerrar-o-ticket)
5. [Artefatos e IOCs](#artefatos-e-iocs)
6. [Lições Aprendidas](#lições-aprendidas)
7. [Referências](#referências)

---

## Visão Geral

Este documento registra o walkthrough completo do alerta **SOC165 — Possible SQL Injection Payload Detected** na plataforma LetsDefend. O objetivo é demonstrar o processo de investigação de um analista SOC nível 1/2, desde a triagem do alerta até o encerramento do ticket.

O alerta foi gerado por requisições consecutivas e suspeitas sendo feitas ao mesmo endpoint de servidor, indicando uma possível tentativa de SQL Injection.

---

## Conceitos Fundamentais

### O que é um SOC?

Um **Security Operations Center (SOC)** é o centro de comando da defesa cibernética de uma organização. Ele monitora atividades digitais, detecta e analisa ameaças e toma medidas para proteger contra incidentes de segurança — combinando tecnologia, processos e profissionais especializados.

### O que é SQL?

**SQL (Structured Query Language)** é uma linguagem de programação padronizada utilizada para gerenciar e manipular bancos de dados relacionais. Permite realizar operações como consulta, inserção, atualização e exclusão de dados.

### O que é SQL Injection?

**SQL Injection** é uma vulnerabilidade de segurança web que ocorre quando um atacante consegue manipular uma query SQL injetando input malicioso através de campos de entrada do usuário (formulários, parâmetros de URL, etc.).

Isso acontece quando o input do usuário **não é devidamente sanitizado ou validado**, permitindo que comandos SQL arbitrários sejam executados no banco de dados da aplicação.

**Exemplo clássico de payload:**
```
' OR 1=1 -- -
```

Esse tipo de payload tenta fazer com que a condição da query sempre seja verdadeira, podendo bypassar autenticações ou extrair dados indevidos.

---

## Fluxo de Investigação

```
Fila de Tickets → Assumir Ticket → Criar Caso → Playbook
      ↓
  Detecção → Análise → Contenção → Remediação
      ↓
  Registrar IOCs → Encerrar Ticket
```

---

## Passo a Passo Detalhado

### Task 1 – Verificar a fila de tickets do SOC

A fila de tickets do SOC é componente crítico para gerenciar e responder a incidentes de segurança. Suas principais funções são:

- Rastreamento e gerenciamento de incidentes
- Priorização e triagem
- Accountability e responsabilidade
- Geração de relatórios
- Análise de tendências
- Conformidade e prontidão para auditoria

**Como acessar no LetsDefend:**
> `Practice` → `Monitoring` → `Main Channel`

Na fila, é possível visualizar alertas com diferentes níveis de severidade aguardando investigação.

---

### Task 2 – Assumir responsabilidade pelo alerta

Cada ticket na fila é atribuído a um analista ou equipe específica, garantindo responsabilidade clara pela resolução do incidente.

**Alerta selecionado:** `Event ID: 115 — Possible SQL Injection Payload Detected`

**Detalhes do alerta:**

| Campo | Valor |
|---|---|
| Event ID | 115 |
| Tipo | Possible SQL Injection |
| Ação | ALLOWED |
| Severidade | Alta |

---

### Task 3 – Criar o caso

A criação de um caso permite ao SOC priorizar incidentes com base na severidade e impacto potencial, otimizando a alocação de recursos e os tempos de resposta.

- Acesse o alerta Event ID: 115
- Clique no ícone de ação para oficializar a investigação
- O caso é criado e vinculado ao analista responsável

---

### Task 4 – Utilizar o Playbook

Os **playbooks** são essenciais para o SOC por garantirem:

- **Consistência:** respostas padronizadas a incidentes
- **Redução de erros:** passos claros e definidos
- **Eficiência:** analistas seguem o mesmo processo

Com o caso criado, o playbook é iniciado para guiar todas as etapas seguintes da investigação.

---

### Task 5 – Detecção

**Objetivo:** Determinar se o alerta é um **True Positive** ou **False Positive**.

**Achados iniciais:**

- A ação registrada no alerta foi **ALLOWED** (a requisição foi permitida pelo servidor)
- Isso indica que a requisição chegou ao servidor e recebeu uma resposta
- A URL suspeita precisava ser decodificada para análise

---

### Task 6 – Análise

#### 6.1 Decodificação da URL

A URL da requisição foi decodificada utilizando a ferramenta [URLDecoder.org](https://www.urldecoder.org/).

**URL decodificada (sanitizada):**
```
hxxps://172[.]16[.]17[.]18/search/?q=" OR 1 = 1 -- -
```

> ⚠️ A URL contém comandos SQL clássicos de SQL Injection (`OR 1=1 -- -`), confirmando a suspeita do alerta.

#### 6.2 Análise do IP de origem

O endereço IP de origem foi verificado nas seguintes plataformas de threat intelligence:

| Plataforma | Resultado |
|---|---|
| **VirusTotal** | 4 vendors sinalizaram o IP como malicioso |
| **AbuseIPDB** | IP externo à rede, origem: EUA, flagged como malicioso |

**Veredicto:** IP classificado como **Malicioso** ❌

#### 6.3 Análise dos logs de rede

Consultando os logs de rede filtrando pelo IP malicioso, foram encontrados **6 eventos**.

**Linha do tempo dos eventos:**

| Evento | Descrição | Código HTTP |
|---|---|---|
| 1 (inicial) | Primeira tentativa de acesso ao servidor | **200** (Sucesso) |
| 2–5 | Requisições subsequentes com payloads de injeção | Variados (sem sucesso) |
| 6 (alerta) | Requisição que disparou o alerta SOC | **500** (Erro interno) |

**Análise:**

- A **primeira requisição** retornou HTTP 200 — o atacante conseguiu acesso inicial ao servidor
- A requisição que **disparou o alerta** retornou HTTP 500 — o ataque de SQL Injection foi **mal-sucedido** no servidor
- Nenhum tráfego adicional do atacante foi encontrado na rede

#### 6.4 Outras verificações

| Verificação | Resultado |
|---|---|
| Ataque foi planejado/autorizado internamente? | ❌ Não |
| Ataque foi bem-sucedido? | ❌ Não (HTTP 500) |
| Classificação final | **True Positive — Ataque Malicioso** |

---

### Task 7 – Contenção

A contenção é fundamental para limitar o impacto de incidentes de segurança, proteger dados e operações, e preservar evidências para análise forense.

**Ação tomada:**

Com base nas evidências coletadas, o endpoint do servidor foi **contido/isolado** para prevenir danos adicionais.

> `Status do Endpoint: CONTAINED` 🔒

---

### Task 8 – Remediação

As seguintes ações de remediação foram recomendadas:

1. **Isolamento de rede:** Isolar a máquina comprometida para impedir que o atacante acesse outros recursos
2. **Validação de input:** Implementar sanitização e validação rigorosa de todos os inputs do usuário
3. **Tratamento de erros personalizado:** Não expor informações sensíveis do banco de dados em mensagens de erro (respostas HTTP 500 genéricas)
4. **Auditoria de segurança regular:** Realizar pentests e revisões de código periodicamente
5. **Uso de Prepared Statements / Parameterized Queries:** Substituir queries dinâmicas por queries parametrizadas

A implementação dessas medidas reduz significativamente o risco de ataques de SQL Injection.

---

### Task 9 – Registrar Artefatos e IOCs

Após a análise, todas as descobertas foram documentadas na seção **Analyst Note** do caso.

#### IOCs (Indicators of Compromise)

| Tipo | Valor | Observação |
|---|---|---|
| IP | `[REDACTED — ver alerta original]` | IP externo, flagged como malicioso em múltiplas plataformas |
| URL | `hxxps://172[.]16[.]17[.]18/search/?q=" OR 1 = 1 -- -` | Payload de SQL Injection |
| Payload | `" OR 1 = 1 -- -` | SQL Injection clássico |

#### Veredicto final

- **Tipo:** True Positive
- **Escalonamento:** Não necessário
- **Ataque bem-sucedido:** Não

---

### Task 10 – Encerrar o ticket

Com o playbook concluído e todas as etapas da análise realizadas, o ticket foi encerrado oficialmente.

**Como encerrar no LetsDefend:**
> `Monitoring` → `Investigation Channel` → Clicar no ícone de fechar (✔️)

O alerta encerrado pode ser encontrado na aba **Closed Alerts**.

**Resultado:** `Event ID 115: True Positive ✅`

---

## Artefatos e IOCs

```
Tipo:     IP Malicioso
Valor:    [IP do alerta — verificar no LetsDefend]
Fonte:    VirusTotal, AbuseIPDB

Tipo:     URL com SQL Injection
Valor:    hxxps://172[.]16[.]17[.]18/search/?q=" OR 1 = 1 -- -
Fonte:    Log de rede

Tipo:     Payload SQL
Valor:    " OR 1=1 -- -
Técnica:  SQL Injection (OWASP A03:2021)
```

---

## Lições Aprendidas

- ✅ Sempre decodificar URLs antes de analisar — payloads são frequentemente URL-encoded para evasão
- ✅ Verificar o IP de origem em múltiplas plataformas de threat intelligence (VirusTotal, AbuseIPDB, etc.)
- ✅ Checar o código de resposta HTTP — um HTTP 500 indica que o ataque não foi bem-sucedido no servidor
- ✅ Mesmo com ataque mal-sucedido, o endpoint deve ser contido como medida preventiva
- ✅ SQL Injection continua sendo uma das ameaças mais comuns (OWASP Top 10)
- ✅ Logs de rede são fundamentais para reconstruir a linha do tempo do ataque

---

## Referências

- [LetsDefend Platform](https://app.letsdefend.io/)
- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [VirusTotal](https://www.virustotal.com/)
- [AbuseIPDB](https://www.abuseipdb.com/)
- [URL Decoder](https://www.urldecoder.org/)
- [OWASP Top 10 — A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)

---

> **Nota:** Este walkthrough foi elaborado com fins educacionais para documentar o processo de investigação de um analista SOC. Os IPs e URLs foram sanitizados seguindo boas práticas de compartilhamento de IOCs.
