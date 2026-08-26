
Primeiramente iniciamos com um scan via `nmap` para mapear o host dado (100.64.1.200)

![](Imagens/Pasted%20image%2020260824152539.png)

Como temos uma porta 80 aberta, podemos abrir no navegador e nos retorna esse site

![](Imagens/Pasted%20image%2020260824152609.png)

Observe que na página de login o CMS está exposto (CuteNews 2.1.2), vamos procurar um exploit que nos dê algum acesso inicial.

Após uma rápida busca na internet encontramos o `CVE-2019-11447`.

Link do exploit: https://www.exploit-db.com/exploits/48800

Após sua execução bem sucedida, temos um RCE, basta ir na pasta do usuário e achar a primeira flag.

![](Imagens/Pasted%20image%2020260824175839.png)

Temos acesso inicial via `www-data`, precisamos escalar privilégio, observem a pasta `/etc/passwd`

![](Imagens/Pasted%20image%2020260824181412.png)

A senha do usuário `gg`está encodada em MD5, após uma rápida quebra de senha via hashcat encontramos sua senha (gg:gg)

Para rodar o su precisamos de um terminal estável, portanto, o comando de `python3 -c 'import pty; pty.spawn("bin/bash")` fez-se necessário.

![](Imagens/Pasted%20image%2020260824182105.png)

Assim encerra-se a primeira máquina.