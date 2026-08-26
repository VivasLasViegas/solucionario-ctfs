
Começamos com um rápido scan no IP fornecido.

![](Imagens/Pasted%20image%2020260824190744.png)

Diferente das demais, esta é uma máquina Windows, conforme a ferramenta acusou.

![](Imagens/Pasted%20image%2020260824191123.png)

Abrindo o IP no navegador chegamos a esse site fornecido, após uma rápida pesquisa, é apenas um template estático, o que queremos está oculto não nesta página.

![](Imagens/Pasted%20image%2020260824191408.png)

Façamos um scan via `dirsearch`a fim de encontrarmos diretórios ocultos.

Aqui o print não acusa, entretanto, após isso, fiz um scan com busca em arquivos `.aspx`pois o servidor possui `ASP.NET` em seus frameworks.

![](Imagens/Pasted%20image%2020260825101603.png)

Chegamos a essa página de upload suspeita. Basta fazer upload de um arquivo .aspx com código malicioso.

![](Imagens/Pasted%20image%2020260826091743.png)

Temos um RCE.

Nesse ponto, basta pesquisarmos o local da flag e a leitura será feita.

![](Imagens/Pasted%20image%2020260825102352.png)

Assim, temos a primeira flag encontrada.

Escalei de RCE para um RevShell na minha máquina, de modo a fazer uma escalação de privilégios mais rápida do host

![](Imagens/Pasted%20image%2020260825103712.png)

Aqui faltou um print, mas ao dar `whoami /priv`achamos uma permissão chamada `SeImpersonatePrivilege` = Enabled, ou seja, vulnerável a `GodPotato`.

Basta executar o exploit e obter privilégio. (`net authority\system`)

![](Imagens/Pasted%20image%2020260825104444.png)

Executando mais uma vez a fim de ler a flag root.

![](Imagens/Pasted%20image%2020260825105138.png)

Assim encerra-se a máquina 3.