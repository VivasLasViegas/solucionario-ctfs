#ctfs 

Comecei com um scan de portas abertas no host fornecido.

```shell
└─$ nmap 10.129.66.154 -Pn --open -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-06 09:24 -03
Nmap scan report for 10.129.245.214
Host is up (0.31s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 143.35 seconds
```

Depois fiz um scan mais aprofundado nas portas encontradas a fim de encontrarmos as versões dos serviços.

```shell
└─$ nmap 10.129.66.154 -Pn --open -p 22,3000 -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-06 09:33 -03
Nmap scan report for 10.129.245.214
Host is up (0.30s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
3000/tcp open  ppp?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.95%I=7%D=8/6%Time=6A747F13%P=x86_64-pc-linux-gnu%r(GetRequest,29B0,"HTTP/1\.1\x20200\x20OK\r\nVary:\x20RSC,\x20Next-Router-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch\r\nAccept-Encoding:\x20gzip\r\nx-nextjs-cache:\x20HIT\r\nx-nextjs-prerender:\x201\r\nx-nextjs-stale-time:\x204294967294\r\nX-Powered-By:\x20Next\.js\r\nCache-Control:\x20s-maxage=31536000,\x20\r\nETag:\x20\"p02u6gnhufd8t\"\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20171 75\r\nDate:\x20Thu,\x206\x20Aug\x202026\x2012:33:25\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html><html\x20lang=\"en\"><head><meta\x20charSet=\"utf-8\"/><meta\x20name=\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\"/><link\x20rel=\"stylesheet\"\x20href=\"/_next/static/css/414e1be982bc8557\.css\"\x20data-precedence=\"next\"/><link\x20rel=\"preload\"\x20as=\"script\"\x20fetchPriority=\"low\"\x20href=\"/_next/static/chunks/webpack-db0a529a99835594\.js\"/><script\x20src=\"/_next/static/chunks/4bd1b696-80bcaf75e1b4285e\.js\"\x20async=\"\"></script><script\x20src=\"/_next/static/chunks/517-d083b552e04dead1\.js\"\x20async=\"\"></script><script\x20s")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(NCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(HTTPOptions,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20RSC,\x20Next-Router-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x20private,\x20no-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r\nDate:\x20Thu,\x206\x20Aug\x202026\x2012:33:30\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20RSC,\x20Next-Router-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x20private,\x20no-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r\nDate:\x20Thu,\x206\x20Aug\x202026\x2012:33:31\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Observe que a resposta nos fornece um "200 OK", esse é o padrão de uma resposta HTTP, ou seja, é um site. Vamos abrir o site (`http://10.129.66.154:3000`) no navegador e ver o que encontramos.

![[Pasted image 20260806101437.png]]


Vamos dar um `feroxbuster` no site a fim de encontrarmos alguma info.

```shell
└─$ feroxbuster -u http://10.129.66.154:3000/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt  -C 404 
                                                                                    
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.66.154:3000/
 🚩  In-Scope Url          │ 10.129.66.154
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 💢  Status Code Filters   │ [404]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        1l      120w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
308      GET        1l        1w       13c http://10.129.66.154:3000/_next/static/ => http://10.129.66.154:3000/_next/static
308      GET        1l        1w       20c http://10.129.66.154:3000/_next/static/chunks/ => http://10.129.66.154:3000/_next/static/chunks
200      GET        1l        2w      463c http://10.129.66.154:3000/_next/static/chunks/main-app-4fbb4b1f318e39a0.js
200      GET        1l     2125w   112594c http://10.129.66.154:3000/_next/static/chunks/polyfills-42372ed130431b0a.js
308      GET        1l        1w        6c http://10.129.66.154:3000/_next/ => http://10.129.66.154:3000/_next
200      GET        1l       66w     3329c http://10.129.66.154:3000/_next/static/chunks/webpack-db0a529a99835594.js
200      GET        1l       85w     6707c http://10.129.66.154:3000/_next/static/css/414e1be982bc8557.css
308      GET        1l        1w       17c http://10.129.66.154:3000/_next/static/css/ => http://10.129.66.154:3000/_next/static/css
200      GET        2l     4694w   181180c http://10.129.66.154:3000/_next/static/chunks/517-d083b552e04dead1.js
200      GET        1l     2979w   166088c http://10.129.66.154:3000/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js
200      GET        1l      337w    17175c http://10.129.66.154:3000/
[####################] - 29m   220555/220555  0s      found:11      errors:0      
[####################] - 29m   220545/220545  129/s   http://10.129.66.154:3000/
```

Temos, de acordo com os diretórios enviados, um Next.js rodando, usando o Wappalyzer verificamos que a versão é 15.0.3

![[Pasted image 20260818224840.png]]

Vamos atrás de tal versão na internet a fim de encontrarmos vulnerabilidades conhecidas

```shell

```