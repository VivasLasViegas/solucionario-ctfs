#ctfs 

Primeiro comecei com um scan `nmap`em todas as portas

```shell
$ nmap 10.129.234.54 --open -p- -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-04 21:19 -03
Nmap scan report for 10.129.234.54
Host is up (0.21s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 47.07 seconds
```

Temos um site no ar. Ao tentar abrir a página no navegador encontramos um domínio a ser declarado na tabela de roteamento estático do computador, basta editar no `/etc/hosts`
e o site aparece.

![[Pasted image 20260804212349.png]]

Após uma rápida exploração da página principal encontramos o endereço de email associado a um dos staffers do site. O que nos traz a segunda flag.

Podemos iniciar agora uma varredura de diretórios.

```shell
$ feroxbuster -u http://nexus.htb/ -w /usr/share/wordlists/dirb/big.txt -x txt,json,env,config,php -C 404

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://nexus.htb/
 🚩  In-Scope Url          │ nexus.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/dirb/big.txt
 💢  Status Code Filters   │ [404]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, json, env, config, php]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        7l       12w      162c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET     1267l     3775w    49296c http://nexus.htb/
[####################] - 8m   122814/122814  0s      found:1       errors:2
[####################] - 8m   122814/122814  252/s   http://nexus.htb/
```

Observe que não nos retornou nada, vamos inicair uma varredura de subdomínios associados a tal host.

```shell
$ ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
       -u "http://10.129.234.54" \
       -H "Host: FUZZ.nexus.htb" \
       -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.234.54
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.nexus.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

billing                [Status: 302, Size: 390, Words: 60, Lines: 12, Duration: 407ms]
git                     [Status: 200, Size: 14472, Words: 1195, Lines: 242, Duration: 152ms]
:: Progress: [19966/19966] :: Job [1/1] :: 170 req/sec :: Duration: [0:01:36] :: Errors: 0 ::
```

Agora tivemos o retorno de dois subdomínios (`billing.nexus.htb`e `git.nexus.htb`). Após configurar a tabela de roteamento conseguimos acessar os dois sites.

![[Pasted image 20260804213635.png]]
![[Pasted image 20260804213654.png]]

Vamos explorar o `git.nexus.htb` a fim de seguir o roteiro do CTF.

Após uma breve procura pelos repositórios abertos encontrei a seguinte credencial de banco de dados:

![[Pasted image 20260805161312.png]]

Conseguimos a senha do banco de dados. Vamos tentar utilizar o e-mail encontrado no site principal e testar as credenciais obtidas do DB_PASSWORD.

Continuar...