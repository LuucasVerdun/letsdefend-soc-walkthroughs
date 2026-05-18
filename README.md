# 🛡️ LetsDefend — SOC Walkthroughs

Repositório com walkthroughs de alertas investigados na plataforma [LetsDefend](https://app.letsdefend.io/), simulando o ambiente de um Security Operations Center (SOC).

> 📌 Objetivo: documentar investigações práticas para fins de estudo e portfólio em Cibersegurança — Blue Team.

---

## 📋 Alertas Resolvidos

| # | Event ID | Alerta | Categoria | Resultado |
|---|----------|--------|-----------|-----------|
| 1 | SOC146 | [Phishing — Excel 4.0 Macro (XLM)](./alerts/SOC146-phishing-excel-macro-xlm.md) | Phishing / Malware | ✅ True Positive |
| 2 | SOC165 | [Possible SQL Injection Payload Detected](./alerts/SOC165-SQL-Injection-Walkthrough.md) | Web Attack | ✅ True Positive |

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Uso |
|---|---|
| [VirusTotal](https://www.virustotal.com/) | Análise de IPs, URLs e arquivos maliciosos |
| [AbuseIPDB](https://www.abuseipdb.com/) | Reputação de endereços IP |
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
    └── SOC165-SQL-Injection-Walkthrough.md
```

---

## 🔖 Tags

`blue-team` `soc` `letsdefend` `cybersecurity` `sql-injection` `phishing` `incident-response`
