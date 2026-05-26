# 🎣 LetsDefend — Phishing Walkthroughs

Repositório com walkthroughs de alertas de **Phishing** investigados na plataforma [LetsDefend](https://app.letsdefend.io/), simulando o ambiente de um Security Operations Center (SOC).

> 📌 Objetivo: documentar investigações práticas de phishing para fins de estudo e portfólio em Cibersegurança — Blue Team.

---

## 📋 Alertas Resolvidos

| # | Event ID | Alerta | Técnica | Resultado |
|---|----------|--------|---------|-----------|
| 1 | SOC141 | [Phishing URL Detected](./alerts/SOC141-Phishing-URL-Detected.md) | URL Maliciosa / Spear Phishing | ✅ True Positive |

---

## 🧠 O que é Phishing?

**Phishing** é um tipo de ataque de engenharia social em que o atacante engana a vítima para que ela clique em links maliciosos, forneça credenciais ou execute arquivos perigosos. Os ataques chegam geralmente por e-mail, SMS ou mensagens instantâneas, se passando por comunicações legítimas.

### Tipos Comuns de Phishing

| Tipo | Descrição |
|---|---|
| **Phishing** | Campanha em massa sem alvo definido |
| **Spear Phishing** | Ataque direcionado a uma pessoa ou organização específica |
| **Whaling** | Spear phishing voltado a executivos de alto nível (C-level) |
| **Smishing** | Phishing via SMS |
| **Vishing** | Phishing via chamada de voz |

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Uso |
|---|---|
| [VirusTotal](https://www.virustotal.com/) | Análise de IPs, URLs e arquivos maliciosos |
| [AbuseIPDB](https://www.abuseipdb.com/) | Reputação de endereços IP |
| [Hybrid Analysis](https://www.hybrid-analysis.com/) | Análise comportamental de URLs e arquivos |
| [URLScan.io](https://urlscan.io/) | Análise e screenshot de URLs suspeitas |
| [LetsDefend](https://app.letsdefend.io/) | Plataforma SOC simulada |

---

## 📁 Estrutura do Repositório

```
letsdefend-phishing-walkthroughs/
│
├── README.md
│
└── alerts/
    └── SOC141-Phishing-URL-Detected.md
```

---

## 🔖 Tags

`blue-team` `soc` `letsdefend` `cybersecurity` `phishing` `spear-phishing` `email-security` `incident-response` `endpoint-containment`
