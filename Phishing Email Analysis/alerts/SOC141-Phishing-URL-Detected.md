# 🛡️ LetsDefend — SOC141: Phishing URL Detected

> **Plataforma:** [LetsDefend](https://app.letsdefend.io/)  
> **Categoria:** Blue Team / SOC Analysis  
> **Tipo de Ataque:** Phishing — URL Maliciosa  
> **Resultado:** ✅ True Positive — Host comprometido e contido  
> **Origem da URL Maliciosa:** Rússia  

---

## 📋 Sumário

- [Visão Geral](#visão-geral)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Detalhes do Alerta](#detalhes-do-alerta)
- [Processo de Investigação](#processo-de-investigação)
  - [1. Leitura e Registro do Alerta](#1-leitura-e-registro-do-alerta)
  - [2. Criar Caso e Abrir Playbook](#2-criar-caso-e-abrir-playbook)
  - [3. Verificação no Log Management](#3-verificação-no-log-management)
  - [4. Análise de Reputação — URL e IP](#4-análise-de-reputação--url-e-ip)
  - [5. Verificação de Acesso à URL Maliciosa](#5-verificação-de-acesso-à-url-maliciosa)
  - [6. Contenção do Host Comprometido](#6-contenção-do-host-comprometido)
  - [7. Registro de Artefatos e IOCs](#7-registro-de-artefatos-e-iocs)
  - [8. Relatório Final e Encerramento](#8-relatório-final-e-encerramento)
- [IOCs (Indicators of Compromise)](#iocs-indicators-of-compromise)
- [Conclusão](#conclusão)
- [Lições Aprendidas](#lições-aprendidas)
- [Referências](#referências)

---

## Visão Geral

Este writeup documenta a investigação do alerta **SOC141 — Phishing URL Detected**, disparado pela detecção de acesso a uma URL maliciosa de origem russa. O usuário `ellie@letsdefend[.]io` acessou o link de phishing duas vezes via porta 80 (HTTP), confirmando o comprometimento do host.

A investigação confirmou que o acesso foi bem-sucedido (**True Positive**) e o host foi contido imediatamente para evitar propagação na rede.

---

## Conceitos Fundamentais

### O que é Phishing?

**Phishing** é um tipo de ataque de engenharia social no qual o agente malicioso engana a vítima para que ela clique em um link malicioso, forneça credenciais ou faça download de malware. Os ataques de phishing geralmente chegam por e-mail, SMS ou mensagens instantâneas, se passando por comunicações legítimas.

### O que é uma URL de Phishing?

Uma **URL de phishing** é um endereço web controlado pelo atacante, projetado para:
- Coletar credenciais (páginas de login falsas)
- Distribuir malware (downloads automáticos)
- Redirecionar para outros sites maliciosos

### O que é Endpoint Containment?

**Contenção de endpoint** é o processo de isolar um dispositivo comprometido da rede corporativa para:
- Impedir a propagação de malware para outros dispositivos
- Preservar evidências para análise forense
- Limitar o impacto do incidente enquanto a remediação é executada

---

## Detalhes do Alerta

| Campo | Valor |
|---|---|
| **Alerta** | SOC141 — Phishing URL Detected |
| **Usuário Afetado** | `ellie[@]letsdefend[.]io` |
| **IP de Destino (C2)** | `91.189.114[.]8` |
| **URL Maliciosa** | `hxxp://mogagrocol[.]ru/wp-content/plugins/akismet/fv/index[.]php?email=ellie[@]letsdefend[.]io` |
| **Porta** | 80 (HTTP) |
| **Nº de Acessos** | 2 conexões registradas nos logs |
| **Ação do Dispositivo** | Allowed (acesso permitido) |
| **Origem da Ameaça** | Rússia |

---

## Processo de Investigação

### 1. Leitura e Registro do Alerta

O primeiro passo é ler com atenção **todas as informações do alerta** e registrá-las em um notepad para fácil consulta durante a investigação. Pontos de atenção inicial:

- URL contém o e-mail da vítima como parâmetro (`?email=ellie@letsdefend.io`) — técnica comum para personalizar páginas de phishing
- Domínio russo (`.ru`) não relacionado ao negócio
- Porta 80 (HTTP) — sem criptografia, facilitando interceptação
- Plugin WordPress (`akismet`) no caminho da URL — pode indicar site legítimo comprometido usado como infraestrutura de phishing

---

### 2. Criar Caso e Abrir Playbook

Um caso foi criado para o alerta, abrindo a tela de detalhes do incidente e o início do playbook. O playbook é o guia passo a passo para conduzir a investigação de forma estruturada.

> **Dica operacional:** Mantenha múltiplas abas abertas — uma para o playbook, outra para o Log Management e outra para as ferramentas de Threat Intelligence. Isso agiliza a investigação sem perder o contexto do caso.

---

### 3. Verificação no Log Management

**Objetivo:** Verificar se o host do usuário efetivamente fez contato com o IP/URL maliciosa.

**Ação:** Busca no **Log Management** pelo IP de destino `91.189.114[.]8`.

**Resultado:** O host do usuário `ellie@letsdefend.io` fez **2 conexões** à URL maliciosa via **porta 80 (HTTP)**, com acesso **permitido** pelo firewall.

**Conclusão:** O acesso ocorreu — o usuário visitou a URL de phishing.

---

### 4. Análise de Reputação — URL e IP

**Ferramentas utilizadas:** AbuseIPDB, VirusTotal e Hybrid Analysis.

> **Boas práticas:** Utilizar múltiplas ferramentas de Threat Intelligence reduz a chance de perder indicadores importantes, pois cada plataforma pode apresentar resultados diferentes. Analisar em profundidade: hosts contatados, domínios, IPs relacionados e localização geográfica.

| Ferramenta | Resultado |
|---|---|
| **VirusTotal** | URL e IP flagged como maliciosos |
| **AbuseIPDB** | IP `91.189.114[.]8` classificado como malicioso — origem: **Rússia** |
| **Hybrid Analysis** | Confirmação de comportamento suspeito |

**Classificação:** ✅ **Malicioso**

---

### 5. Verificação de Acesso à URL Maliciosa

**Objetivo:** Confirmar se o usuário de fato acessou a URL maliciosa (não apenas se o alerta disparou).

**Resultado:** Confirmado via Log Management — a conexão foi **Allowed** (permitida) e o acesso ocorreu **2 vezes**.

**Resposta no Playbook:** ✅ **Accessed** — o usuário acessou a URL maliciosa.

---

### 6. Contenção do Host Comprometido

Como o usuário acessou a URL maliciosa e o host está potencialmente comprometido, a contenção foi executada imediatamente.

**Ferramenta:** **Endpoint Security** — permite localizar o dispositivo do usuário, visualizar logs de atividade e acionar a contenção.

**Status:** `Host de ellie@letsdefend.io → CONTAINED`

> **Observação:** Os logs de conexão à URL/IP maliciosa deveriam estar visíveis no Endpoint Security, mas não foram exibidos neste caso — um comportamento reportado na plataforma.

---

### 7. Registro de Artefatos e IOCs

Os seguintes artefatos foram adicionados ao relatório do caso:

| Tipo | Valor |
|---|---|
| Domínio malicioso | `mogagrocol[.]ru` |
| IP do atacante | `91.189.114[.]8` |
| URL completa | `hxxp://mogagrocol[.]ru/wp-content/plugins/akismet/fv/index[.]php?email=ellie[@]letsdefend[.]io` |

---

### 8. Relatório Final e Encerramento

Um relatório detalhado foi elaborado cobrindo:

- **Quem:** Usuário `ellie@letsdefend.io` (host comprometido)
- **O quê:** Acesso a URL de phishing de origem russa
- **Onde:** Rede interna — via porta 80
- **Quando:** Registrado no alerta SOC141
- **Como:** 2 conexões permitidas pelo firewall ao IP `91.189.114[.]8`
- **Sugestões:** Contenção do host, reset de credenciais, treinamento de conscientização de phishing para o usuário

**Classificação Final:** ✅ **True Positive**

O alerta foi encerrado em **Case Management** e movido para **Closed Alerts**.

---

## IOCs (Indicators of Compromise)

| Tipo | Valor | Observação |
|---|---|---|
| IP Malicioso | `91.189.114[.]8` | Rússia — flagged no AbuseIPDB e VirusTotal |
| Domínio | `mogagrocol[.]ru` | Domínio russo — infraestrutura de phishing |
| URL Maliciosa | `hxxp://mogagrocol[.]ru/wp-content/plugins/akismet/fv/index[.]php?email=ellie[@]letsdefend[.]io` | URL de phishing personalizada com e-mail da vítima |
| Usuário Comprometido | `ellie[@]letsdefend[.]io` | Host contido após confirmação do acesso |
| Porta Utilizada | `80` (HTTP) | Sem criptografia |

> ⚠️ **Nota:** IPs, domínios e URLs foram defangeados (com `[.]` e `hxxp`) para evitar cliques acidentais.

---

## Conclusão

O alerta SOC141 demonstrou um caso clássico de phishing bem-sucedido. O usuário `ellie@letsdefend.io` acessou uma URL maliciosa de origem russa duas vezes, com a conexão permitida pelo firewall corporativo. A URL utilizava o e-mail da vítima como parâmetro — técnica comum para personalizar a experiência de phishing e aumentar a credibilidade da página maliciosa.

O fluxo completo da investigação seguiu o playbook SOC:

```
Alerta → Log Management → Threat Intel (URL + IP) → Confirmação de Acesso → Contenção → Artefatos → Relatório → Encerramento
```

A contenção imediata do host foi a ação crítica para limitar o impacto do incidente e prevenir a propagação de eventual malware na rede.

---

## Lições Aprendidas

- **Múltiplas ferramentas de Threat Intel:** Usar AbuseIPDB, VirusTotal e Hybrid Analysis em conjunto aumenta a cobertura de detecção — cada plataforma pode ter informações que as outras não têm.
- **Parâmetros de URL revelam intenção:** A presença do e-mail da vítima como parâmetro (`?email=`) indica uma campanha de phishing direcionada (spear phishing), não genérica.
- **Porta 80 é um sinal de alerta:** Conexões HTTP (sem criptografia) a domínios externos suspeitos merecem atenção especial, especialmente quando o domínio é de país de alto risco.
- **Log Management confirma o acesso real:** O alerta dispara, mas é o Log Management que confirma se o acesso de fato ocorreu — dado essencial para decidir sobre contenção.
- **Contenção proporcional ao risco:** Quando o acesso à URL maliciosa é confirmado, a contenção imediata do host é a resposta correta para limitar o blast radius do incidente.
- **Relatório estruturado (5W+H):** Um bom relatório de incidente responde Who, What, Where, When, Why e How — essencial para escalação e aprendizado organizacional.

---

## Referências

- [LetsDefend Platform](https://app.letsdefend.io/)
- [OWASP — Phishing](https://owasp.org/www-community/attacks/Phishing)
- [AbuseIPDB](https://www.abuseipdb.com/)
- [VirusTotal](https://www.virustotal.com/)
- [Hybrid Analysis](https://www.hybrid-analysis.com/)
- [MITRE ATT&CK — Phishing (T1566)](https://attack.mitre.org/techniques/T1566/)

---

*Writeup baseado no artigo original de [Giovanni Isola (@giovanni.isola1)](https://medium.com/@giovanni.isola1) no Medium.*
