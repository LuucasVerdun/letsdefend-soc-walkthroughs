# 🌐 Detecting Web Attacks — SOC Walkthroughs

Walkthroughs de alertas de **ataques web** investigados na plataforma [LetsDefend](https://app.letsdefend.io/), simulando o ambiente de um Security Operations Center (SOC).

> 📌 Foco: identificar, analisar e responder a ataques direcionados a aplicações e servidores web — SQL Injection, Command Injection, LFI, IDOR e similares.

---

## 📋 Alertas Resolvidos

| # | Event ID | Alerta | Tipo de Ataque | Resultado |
|---|----------|--------|----------------|-----------|
| 1 | SOC165 | [Possible SQL Injection Payload Detected](./alerts/SOC165-SQL-Injection-Walkthrough.md) | SQL Injection | ✅ True Positive |
| 2 | SOC167 | [LS Command Detected in Requested URL](./alerts/SOC167-LS-Command-Detected-in-URL.md) | Command Injection | ❌ False Positive |
| 3 | SOC168 | [Whoami Command Detected in Request Body](./alerts/SOC168-Whoami-Command-Detected.md) | Command Injection | ✅ True Positive |
| 4 | SOC169 | [Possible IDOR Attack Detected](./alerts/SOC169-Possible-IDOR-Attack-Detected.md) | IDOR | ✅ True Positive |
| 5 | SOC170 | [Passwd Found in Requested URL — Possible LFI Attack](./alerts/SOC170-Passwd-Found-in-URL-Possible-LFI.md) | LFI / Path Traversal | ✅ True Positive (não bem-sucedido) |

---

## 🧠 Tipos de Ataque Cobertos

| Ataque | Descrição |
|---|---|
| **SQL Injection** | Inserção de código SQL malicioso em campos de entrada para manipular o banco de dados |
| **Command Injection** | Execução de comandos do sistema operacional via parâmetros vulneráveis da aplicação |
| **IDOR** | Manipulação de identificadores para acessar recursos de outros usuários sem autorização |
| **LFI / Path Traversal** | Navegação por diretórios do servidor para acessar arquivos locais sensíveis |

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Uso |
|---|---|
| [VirusTotal](https://www.virustotal.com/) | Análise de IPs e URLs maliciosas |
| [AbuseIPDB](https://www.abuseipdb.com/) | Reputação de endereços IP |
| [Cisco Talos](https://talosintelligence.com/) | Inteligência de ameaças e reputação de IPs |
| [URLDecoder](https://www.urldecoder.org/) | Decodificação de URLs suspeitas com encoding |
| [LetsDefend](https://app.letsdefend.io/) | Plataforma SOC simulada |

---

## 📁 Estrutura

```
Detecting Web Attacks/
│
├── README.md
│
└── alerts/
    ├── SOC165-SQL-Injection-Walkthrough.md
    ├── SOC167-LS-Command-Detected-in-URL.md
    ├── SOC168-Whoami-Command-Detected.md
    ├── SOC169-Possible-IDOR-Attack-Detected.md
    └── SOC170-Passwd-Found-in-URL-Possible-LFI.md
```

---

## 🔖 Tags

`web-attacks` `sql-injection` `command-injection` `idor` `lfi` `path-traversal` `blue-team` `soc` `letsdefend`
