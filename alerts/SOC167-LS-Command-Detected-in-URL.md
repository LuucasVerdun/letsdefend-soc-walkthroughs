# 🛡️ LetsDefend — SOC167: LS Command Detected in Requested URL

> **Plataforma:** [LetsDefend](https://app.letsdefend.io/)  
> **Categoria:** Blue Team / SOC Analysis  
> **Tipo de Ataque:** Command Injection (suspeito)  
> **Resultado:** ❌ False Positive — Alerta encerrado  
> **Event ID:** 117  

---

## 📋 Sumário

- [Visão Geral](#visão-geral)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Detalhes do Alerta](#detalhes-do-alerta)
- [Processo de Investigação](#processo-de-investigação)
  - [1. Análise Inicial do Alerta](#1-análise-inicial-do-alerta)
  - [2. Iniciar o Playbook](#2-iniciar-o-playbook)
  - [3. Pergunta 1 — A requisição é maliciosa?](#3-pergunta-1--a-requisição-é-maliciosa)
  - [4. Pergunta 2 — Existem outras requisições?](#4-pergunta-2--existem-outras-requisições)
  - [5. Pergunta 3 — As demais requisições são maliciosas?](#5-pergunta-3--as-demais-requisições-são-maliciosas)
  - [6. Nota de Análise](#6-nota-de-análise)
  - [7. Encerramento do Ticket](#7-encerramento-do-ticket)
- [Por que o Alerta Disparou?](#por-que-o-alerta-disparou)
- [Conclusão](#conclusão)
- [Lições Aprendidas](#lições-aprendidas)
- [Referências](#referências)

---

## Visão Geral

Este writeup documenta a investigação do alerta **SOC167 — EventID 117**, disparado pela detecção do padrão `ls` em uma URL requisitada. O alerta apontava para uma possível tentativa de **Command Injection**, mas a análise revelou que se tratava de um **False Positive**.

A string `ls` foi detectada como parte da palavra `skills` em uma URL de busca legítima no site do LetsDefend, sem qualquer intenção maliciosa.

---

## Conceitos Fundamentais

### O que é Command Injection?

**Command Injection** é um tipo de ataque em que o objetivo é a execução de comandos arbitrários no sistema operacional do servidor por meio de uma aplicação vulnerável. O atacante insere comandos maliciosos em campos de entrada que são processados sem a devida sanitização.

> Fonte: [OWASP — Command Injection](https://owasp.org/www-community/attacks/Command_Injection)

### O que é o comando `ls`?

O comando `ls` lista todos os arquivos, programas e diretórios no local onde é executado em sistemas Unix/Linux. Por exemplo:

```bash
$ ls /secret/
secret_sub/   super_secret.txt
```

Se um atacante conseguir executar `ls` em um diretório do servidor, ele pode mapear a estrutura de arquivos, identificar dados sensíveis e planejar ataques subsequentes.

### O que é um False Positive?

Um **False Positive** ocorre quando um sistema de detecção dispara um alerta para uma atividade que, após investigação, se revela legítima e inofensiva. Falsos positivos mal gerenciados podem gerar fadiga de alertas nos analistas SOC, comprometendo a eficácia do time.

---

## Detalhes do Alerta

| Campo | Valor |
|---|---|
| **Event ID** | 117 |
| **Alerta** | LS Command Detected in Requested URL |
| **Hostname** | EliotPRD |
| **IP de Origem** | `172.16.17[.]46` (interno) |
| **IP de Destino** | `188.114.96[.]15` |
| **URL Requisitada** | `hxxps://letsdefend[.]io/blog/?s=skills` |
| **Padrão Detectado** | `ls` (contido na palavra `skills`) |

---

## Processo de Investigação

### 1. Análise Inicial do Alerta

Ao revisar o alerta, os dados iniciais chamam atenção:

- A URL requisitada é `hxxps://letsdefend[.]io/blog/?s=skills`
- O parâmetro de busca é `?s=skills` — uma pesquisa simples pelo termo "skills" no blog do LetsDefend
- O padrão `ls` foi detectado **dentro da palavra `skills`**, não como um comando isolado

Apesar de aparentemente inofensiva, a investigação deve seguir o processo completo do playbook para garantir que não há atividade maliciosa associada.

---

### 2. Iniciar o Playbook

O playbook foi iniciado clicando em **"Start Playbook!"**, seguindo o fluxo padronizado de investigação.

---

### 3. Pergunta 1 — A requisição é maliciosa?

**Objetivo:** Determinar se a URL requisitada representa uma ameaça real.

**Ações realizadas:**

1. **Verificação de reputação do IP de destino** no VirusTotal:
   - O IP `188.114.96[.]15` retornou resultado **limpo** — nenhum fornecedor sinalizou como malicioso.

2. **Análise no Log Management:**
   - Filtragem por IP de origem (`172.16.17[.]46`) e IP de destino.
   - Verificação de todas as URLs requisitadas pelo host `EliotPRD`.
   - Todas as requisições encontradas eram **buscas simples e legítimas** relacionadas a conteúdo de SOC Analyst no blog do LetsDefend.

3. **Análise da URL alvo:**
   - `?s=skills` é um parâmetro de busca padrão WordPress.
   - Não há regex, ofuscação, encoding malicioso ou tentativa de injeção de comandos.
   - O `ls` detectado é apenas uma **substring da palavra `skills`**.

**Resposta:** ✅ **Non-Malicious**

---

### 4. Pergunta 2 — Existem outras requisições?

**Objetivo:** Verificar se o host realizou outras requisições além da que disparou o alerta.

**Ação:** Análise completa dos logs filtrados por IP de origem no Log Management.

**Resultado:** Sim — foram encontradas **outras requisições** do mesmo host.

**Resposta:** ✅ **Sim**

---

### 5. Pergunta 3 — As demais requisições são maliciosas?

**Objetivo:** Determinar se as outras requisições do host `EliotPRD` representam ameaças.

**Análise:** Todas as demais requisições encontradas eram buscas simples no blog do LetsDefend, relacionadas a conteúdo de capacitação em Blue Team — sem qualquer indício de malícia.

**Resposta:** ✅ **Non-Malicious**

---

### 6. Nota de Análise

A seguinte nota foi registrada no campo **Analyst Note** do caso:

> *"Todas as requisições do usuário são inofensivas e relacionadas ao aprendizado de habilidades de Blue Team. O alerta foi disparado pelo termo 'skills', que contém a substring 'ls', mas não se trata de um comando de injeção. O alerta pode ter sido gerado por uma regra mal configurada. Recomenda-se revisar e corrigir a regra de detecção de 'ls' para reduzir falsos positivos."*

---

### 7. Encerramento do Ticket

**Classificação Final:** ❌ **False Positive**

O alerta foi encerrado após a conclusão de todas as etapas do playbook. O ticket foi movido para **Closed Alerts**.

---

## Por que o Alerta Disparou?

O alerta foi disparado por uma **regra de detecção mal configurada** que buscava pela string `ls` em URLs sem considerar o contexto completo da palavra.

```
URL: https://letsdefend.io/blog/?s=skills
                                      ^^
                               "ls" detectado aqui
                          (substring de "skills")
```

Esse é um exemplo clássico de como regras de detecção muito genéricas geram **falsos positivos em larga escala**. Uma regra mais precisa deveria:

- Buscar por `ls` como **parâmetro isolado** (ex: `?cmd=ls`, `?exec=ls`)
- Considerar o contexto da requisição (método HTTP, estrutura da URL, outros parâmetros)
- Utilizar expressões regulares com delimitadores de palavra (`\bls\b`)

---

## Conclusão

O alerta SOC167 demonstrou um cenário importante para qualquer analista SOC: **nem todo alerta representa uma ameaça real**. A investigação confirmou que:

- A URL era uma busca legítima no blog do LetsDefend
- O IP de destino não possuía histórico malicioso
- Todas as demais requisições do host eram inofensivas
- O padrão `ls` foi detectado como **substring**, não como comando injetado

O fluxo completo da investigação foi:

```
Alerta → Verificação de Reputação → Log Management → Análise de URLs → False Positive → Encerramento
```

A recomendação final foi revisar a regra de detecção responsável pelo alerta, adicionando contexto e precisão para evitar novos falsos positivos do mesmo tipo.

---

## Lições Aprendidas

- **Regras de detecção precisam de contexto:** Detectar substrings sem considerar o contexto da requisição gera ruído desnecessário e fadiga de alertas.
- **False Positives têm custo real:** Cada falso positivo consome tempo de analistas que poderiam estar investigando ameaças reais.
- **Log Management é essencial:** Analisar o comportamento completo do host — não apenas o evento isolado — é fundamental para uma conclusão correta.
- **Reputação de IPs e URLs:** Verificar fontes externas (VirusTotal, Talos) é um passo obrigatório antes de qualquer conclusão.
- **Documentação e notas:** Registrar o raciocínio da investigação garante rastreabilidade e aprendizado para o time.

---

## Referências

- [LetsDefend Platform](https://app.letsdefend.io/)
- [OWASP — Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [VirusTotal](https://www.virustotal.com/)
- [MITRE ATT&CK — OS Command Injection (T1059)](https://attack.mitre.org/techniques/T1059/)

---

*Writeup baseado no artigo original de [Emre Can Erol (@erolemrecan)](https://medium.com/@erolemrecan) no Medium.*
