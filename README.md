# 🛡️ LetsDefend — SOC Walkthroughs

Repositório com walkthroughs de alertas investigados na plataforma [LetsDefend](https://app.letsdefend.io/), simulando o ambiente de um Security Operations Center (SOC).

> 📌 Objetivo: documentar investigações práticas para fins de estudo e portfólio em Cibersegurança — Blue Team.

---

## 📋 Alertas Resolvidos

| # | Event ID | Alerta | Categoria | Resultado |
|---|----------|--------|-----------|-----------|
| 1 | SOC146 | [Phishing — Excel 4.0 Macro (XLM)](./alerts/SOC146-phishing-excel-macro-xlm.md) | Phishing / Malware | ✅ True Positive |
| 2 | SOC165 | [Possible SQL Injection Payload Detected](./alerts/SOC165-SQL-Injection-Walkthrough.md) | Web Attack | ✅ True Positive |
| 3 | SOC168 | [Whoami Command Detected in Request Body](./alerts/SOC168-Whoami-Command-Detected.md) | Web Attack / Command Injection | ✅ True Positive |

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Uso |
|---|---|
| [VirusTotal](https://www.virustotal.com/) | Análise de IPs, URLs e arquivos maliciosos |
| [AbuseIPDB](https://www.abuseipdb.com/) | Reputação de endereços IP |
| [Cisco Talos](https://talosintelligence.com/) | Inteligência de ameaças e reputação de IPs |
| [URLDecoder](https://www.urldecoder.org/) | Decodificação de URLs suspeitas |
| [LetsDefend](https://app.letsdefend.io/) | Plataforma SOC simulada |

---

## 📁 Estrutura do Repositório

```
letsdefend-soc-walkthroughs/
│
├── README.md
│
└── alerts/
    ├── SOC146-phishing-excel-macro-xlm.md
    ├── SOC165-SQL-Injection-Walkthrough.md
    └── SOC168-Whoami-Command-Detected.md
```

---

## 🔖 Tags

`blue-team` `soc` `letsdefend` `cybersecurity` `sql-injection` `phishing` `command-injection` `incident-response`