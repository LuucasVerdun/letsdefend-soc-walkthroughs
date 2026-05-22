# 🛡️ Web Attack Detection Cheat Sheet
## XSS, SQL Injection, Command Injection, LFI & Directory Traversal

---

# 🔥 XSS (Cross-Site Scripting)

## 📌 Tags HTML suspeitas

```text
<script
</script>
<img
<svg
<iframe
<body
<input
<video
<audio
<object
<embed
<link
<meta
<style
<form
```

---

## 📌 Eventos JavaScript

```text
onerror=
onload=
onclick=
onmouseover=
onfocus=
onmouseenter=
onmouseleave=
onmousemove=
onkeydown=
onkeyup=
onsubmit=
onanimationstart=
```

---

## 📌 Funções JavaScript suspeitas

```text
alert(
prompt(
confirm(
eval(
setTimeout(
setInterval(
document.cookie
document.domain
document.location
window.location
fetch(
XMLHttpRequest
atob(
btoa(
```

---

# 💉 SQL Injection (SQLi)

## 📌 Operadores e caracteres suspeitos

```text
'
"
--
#
/*
*/
;
```

---

## 📌 Payloads clássicos SQLi

```sql
' OR 1=1 --
" OR "1"="1
admin'--
' UNION SELECT NULL--
' OR 'a'='a
```

---

## 📌 Comandos SQL perigosos

```sql
SELECT
UNION
INSERT
UPDATE
DELETE
DROP
CREATE
ALTER
EXEC
EXECUTE
xp_cmdshell
```

---

# 💻 Command Injection

## 📌 Caracteres suspeitos

```bash
;
&&
||
|
`
$()
```

---

## 📌 Comandos perigosos

```bash
whoami
id
uname
pwd
cat /etc/passwd
curl
wget
nc
bash
sh
python
perl
php
powershell
cmd.exe
```

---

# 📂 LFI / RFI

## 📌 Indicadores

```text
../
..%2f
/etc/passwd
boot.ini
win.ini
proc/self/environ
php://input
```

---

# 📁 Directory Traversal

```text
../../
..\
%2e%2e%2f
%252e%252e%252f
```

---

# 🤖 Ferramentas automatizadas (User-Agent)

```text
sqlmap
Nikto
Nmap Scripting Engine
curl/
Wget/
python-requests
Go-http-client
```

---

# 🔎 Regex para SIEM / Wazuh / Splunk

## XSS

```regex
(<script|onerror=|onload=|javascript:|alert\(|document\.cookie)
```

## SQL Injection

```regex
(UNION SELECT|OR 1=1|SLEEP\(|WAITFOR DELAY|xp_cmdshell|--|#)
```

## Command Injection

```regex
(;|&&|\|\||\$\(|`|whoami|id|curl|wget|nc)
```

---

# 📊 Campos importantes para analisar

- URI
- Query string
- POST body
- User-Agent
- Referrer
- Response code
- Response size

---

# 🚨 Status HTTP suspeitos

| Código | Possível significado |
|---|---|
| 200 | Payload aceito |
| 500 | Backend/SQL quebrou |
| 403 | WAF bloqueou |
| 302 | Redirecionamento suspeito |

---

# 🎯 O que normalmente indica ataque real

## SQL Injection

- Muitas tentativas seguidas
- Mudança constante de payload
- Uso de UNION/SLEEP
- Erros SQL no response

## XSS

- HTML/JS dentro da URL
- Payload refletido na resposta
- Uso de document.cookie

---

# 📚 Fluxo rápido de análise SOC

## 1️⃣ Verificar URL

```text
<script
OR 1=1
UNION
../
```

## 2️⃣ Verificar resposta HTTP

```text
500
SQL syntax error
alert(
```

## 3️⃣ Verificar User-Agent

```text
sqlmap
curl
python-requests
```

## 4️⃣ Verificar repetição

- Muitas tentativas
- Mesmo IP
- Payloads variados
- Enumeração automatizada
