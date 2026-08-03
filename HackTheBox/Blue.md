#ctfs 

Comecei com um scaneamento do IP fornecido via nmap `nmap 10.129.53.163 --open -p-`

![[Pasted image 20260803120550.png]]

O que nos forneceu a primeira flag.

Logo em seguida executei um scaneamento direcionado às portas TCP com menos de 5 dígitos.

![[Pasted image 20260803120728.png]]

O que nos forneceu a segunda e terceira flags.

Na sequência executei um scan sem autenticação via `smbclient` a fim de verificar os shares disponíveis `smbclient -L \\10.129.53.163 -N`

![[Pasted image 20260803120946.png]]

O que nos forneceu a quarta flag.

A quinta flag nos solicita fazer uma pequena busca na internet a fim de encontrar a vulnerabilidade que possibilita um RCE no SMB. O resultado já nos dá a resposta da sexta flag, a qual pergunta o nome do worm associado a essa falha que se popularizou em 2017.

Próximo passo é explorarmos tal falha. Com isso, abri o console do Metasploit e pesquisei "EternalBlue" nos módulos disponíveis

![[Pasted image 20260803121423.png]]

O que após dar exploit estabelece-se conexão

![[Pasted image 20260803122230.png]]

Dando um `whoami /user` para ver o usuário utilizado conseguimos a sétima flag

Entramos na pasta do usuário harris e vemos a flag de user.txt

![[Pasted image 20260803122538.png]]

Como somos um usuário de alto privilégio (nt authority/system) basta somente ir a pasta do "Administrator"

![[Pasted image 20260803123055.png]]

Assim termina-se o CTF.