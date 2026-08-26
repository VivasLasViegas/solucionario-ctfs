A máquina 2 estava no mesmo host das demais, entretanto, seu IP estava oculto. Precisamos escanear a rede a fim de encontrar seu IP.

![](Imagens/Pasted%20image%2020260824183023.png)

O IP `100.64.1.32`não pertence às demais máquinas, portanto, esse é o endereço que procuramos.

Faltou a imagem do scan `nmap`do host, entretanto, havia uma porta 80 e 22 abertas. Com isso, abri no navegador a fim de ver a página web.

![](Imagens/Pasted%20image%2020260826091639.png)

Temos um site vazio, o caminho é buscar diretórios, nesse caso, o `cgi-bin`é um ponto interessante de análise.

![](Imagens/Pasted%20image%2020260825220034.png)

Temos um potencial vetor de ataque, a pasta `/cgi-bin/resources.pl`. Após pesquisar na internet, descobrimos que o servidor é vulnerável a `ShellShock`. Com isso, basta executar o exploit no `curl`

![](Imagens/Pasted%20image%2020260825220009.png)

Com isso, temos um acesso inicial à máquina, basta pesquisa a flag de usuário.

![](Imagens/Pasted%20image%2020260825231128.png)

Queremos agora escalar privilégio, vamos pesquisar arquivos com o bit `SUID` ativo.

![](Imagens/Pasted%20image%2020260825231219.png)

Temos o `pkexec`com bit ativo, existe um exploit bem conhecido a respeito do `pkexec`, o [CVE-2021-4034](https://www.exploit-db.com/exploits/50689)

Após sua execução, temos `root`.

![](Imagens/Pasted%20image%2020260825231027.png)

Assim, encerra-se a segunda máquina.