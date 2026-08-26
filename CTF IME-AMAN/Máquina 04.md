Iniciei com um scan no host.

![](Pasted%20image%2020260824183311.png)

Porta 80 aberta, abramos no navegador.

![](Pasted%20image%2020260824183211.png)

Veja que é uma página de login customizada, a princípio sem CMS conhecido, podemos tentar registrar um usuário para ver o que acontece.

![](Pasted%20image%2020260824183746.png)

Esse "Bem vindo, 123" me fez acreditar por um tempo que seria um `SSTI`, mas a resposta não vem daí.

Tentei fazer um `IDOR`no host mas também estava limitado.

![](Pasted%20image%2020260824183910.png)

Algo interessante que acontecia ao se abrir o site foi a importação de uma imagem através de um parâmetro `?get_picture`.

![](Pasted%20image%2020260826093216.png)

A importação da imagem é através de um link externo, ou seja, é vulnerável a `LFI`. Podemos acessar a `/etc/passwd` e a flag (o qual não tirei print) via navegador com a utilização certa do parâmetro vulnerável.

![](Pasted%20image%2020260824223614.png)

Agora é que o CTF fica bem mais difícil. Precisamos escalar privilégios e só temos um `LFI`, não há `RCE`, e também não é possível um `Log Poisoning`. Resolvemos da seguinte forma:

A chain é um `SSRF` usando `gopher://` para mandar pacotes para o MySQL rodando interno que é possível acessar com root e sem senha (permissão vulnerável), e lá ele consegue criar um arquivo em `/tmp`, criei um `webshell` em `/tmp`.

Após isso usei um `parameter injection` naquele `curl` para copiar do `/tmp/` para o `recipes` (um dos diretórios descobertos via `dirsearch`)

![](Pasted%20image%2020260826094331.png)

Com o `-K` e um arquivo de config que copia um arquivo `.php` la no `/var/www/html`. A partir dai, com o `into dumpfile`, criavam-se os arquivos pelo MySQL. (made by Hilux)

![](Pasted%20image%2020260826094319.png)


