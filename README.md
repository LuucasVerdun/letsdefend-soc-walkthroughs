# 🛡️ LetsDefend — SOC Walkthroughs

Repositório com walkthroughs de alertas investigados na plataforma [LetsDefend](https://app.letsdefend.io/), simulando o ambiente de um Security Operations Center (SOC).

> 📌 Objetivo: documentar investigações práticas para fins de estudo e portfólio em Cibersegurança — Blue Team.

---

## 📂 Categorias

### 🌐 [Detecting Web Attacks](./Detecting%20Web%20Attacks/)
Ataques direcionados a aplicações e servidores web — SQL Injection, Command Injection, LFI, IDOR e similares.

| # | Event ID | Alerta | Tipo de Ataque | Resultado |
|---|----------|--------|----------------|-----------|
| 1 | SOC165 | [Possible SQL Injection Payload Detected](./Detecting%20Web%20Attacks/alerts/SOC165-SQL-Injection-Walkthrough.md) | SQL Injection | ✅ True Positive |
| 2 | SOC167 | [LS Command Detected in Requested URL](./Detecting%20Web%20Attacks/alerts/SOC167-LS-Command-Detected-in-URL.md) | Command Injection | ❌ False Positive |
| 3 | SOC168 | [Whoami Command Detected in Request Body](./Detecting%20Web%20Attacks/alerts/SOC168-Whoami-Command-Detected.md) | Command Injection | ✅ True Positive |
| 4 | SOC169 | [Possible IDOR Attack Detected](./Detecting%20Web%20Attacks/alerts/SOC169-Possible-IDOR-Attack-Detected.md) | IDOR | ✅ True Positive |
| 5 | SOC170 | [Passwd Found in Requested URL — Possible LFI Attack](./Detecting%20Web%20Attacks/alerts/SOC170-Passwd-Found-in-URL-Possible-LFI.md) | LFI / Path Traversal | ✅ True Positive (não bem-sucedido) |

---

### 🎣 [Phishing Email Analysis](./Phishing%20Email%20Analysis/)
Análise de e-mails e URLs de phishing, spear phishing e ameaças de engenharia social.

| # | Event ID | Alerta | Técnica | Resultado |
|---|----------|--------|---------|-----------|
| 1 | SOC141 | [Phishing URL Detected](./Phishing%20Email%20Analysis/alerts/SOC141-Phishing-URL-Detected.md) | URL Maliciosa / Spear Phishing | ✅ True Positive |

---

### 🦠 [Malware Analysis](./Malware%20Analysis/)
Análise de alertas relacionados a malware, arquivos suspeitos e comportamentos maliciosos em endpoints.

| # | Event ID | Alerta | Técnica | Resultado |
|---|----------|--------|---------|-----------|
| — | — | *Em breve* | — | — |

---

## 📊 Estatísticas

| Métrica | Valor |
|---|---|
| Total de alertas investigados | 6 |
| True Positives | 5 |
| False Positives | 1 |
| Categorias cobertas | 3 |

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Uso |
|---|---|
| [VirusTotal](https://www.virustotal.com/) | Análise de IPs, URLs e arquivos maliciosos |
| [AbuseIPDB](https://www.abuseipdb.com/) | Reputação de endereços IP |
| [Cisco Talos](https://talosintelligence.com/) | Inteligência de ameaças e reputação de IPs |
| [Hybrid Analysis](https://www.hybrid-analysis.com/) | Análise comportamental de URLs e arquivos |
| [URLScan.io](https://urlscan.io/) | Análise e screenshot de URLs suspeitas |
| [URLDecoder](https://www.urldecoder.org/) | Decodificação de URLs suspeitas com encoding |
| [LetsDefend](https://app.letsdefend.io/) | Plataforma SOC simulada |

---

## 📁 Estrutura do Repositório

```
letsdefend-soc-walkthroughs/
│
├── README.md
│
├── Detecting Web Attacks/
│   ├── README.md
│   └── alerts/
│       ├── SOC165-SQL-Injection-Walkthrough.md
│       ├── SOC167-LS-Command-Detected-in-URL.md
│       ├── SOC168-Whoami-Command-Detected.md
│       ├── SOC169-Possible-IDOR-Attack-Detected.md
│       └── SOC170-Passwd-Found-in-URL-Possible-LFI.md
│
├── Phishing Email Analysis/
│   ├── README.md
│   └── alerts/
│       └── SOC141-Phishing-URL-Detected.md
│
└── Malware Analysis/
    ├── README.md
    └── alerts/
        └── (em breve)
```

---

## 🔖 Tags

`blue-team` `soc` `letsdefend` `cybersecurity` `incident-response` `sql-injection` `command-injection` `idor` `lfi` `path-traversal` `phishing` `spear-phishing` `malware` `false-positive`
