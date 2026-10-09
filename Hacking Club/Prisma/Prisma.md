#ctfs 

Primeiro começo com uma scan voa `nmap`

```shell
└─$ nmap 172.16.5.108 -Pn --open -sC -p-  
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-05 18:38 -03
Nmap scan report for 172.16.5.108
Host is up (0.15s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-title: Prisma Digital Print

Nmap done: 1 IP address (1 host up) scanned in 54.79 seconds
```

Agora de forma mais aprofundada:

```shell
└─$ nmap 172.16.5.108 -Pn --open -sCV -p 22,80   
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-05 18:39 -03
Nmap scan report for 172.16.5.108
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.28.3 (Ubuntu)
|_http-title: Prisma Digital Print
|_http-server-header: nginx/1.28.3 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.29 seconds
```

Temos duas portas abertas. SSH E HTTP. Vamos iniciar nossa fase de enumeração
# Enumeração

## Serviço Web

Ao abrirmos o navegador e digitarmos o IP `172.16.5.108` nos deparamos com a seguinte página

![[Pasted image 20261005183854.png]]

Observe que podemos anexar um arquivo em `http://172.16.5.108/upload`

![[Pasted image 20261005184137.png]]

```shell
└─$ dirsearch -u http://172.16.5.108/ -w /usr/share/wordlists/dirb/big.txt -x 404,500 -r
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                    
 (_||| _) (/_(_|| (_| )                                                             
                                                                                    
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 20469

Output File: /home/pentecostes/reports/http_172.16.5.108/__26-10-05_18-49-50.txt

Target: http://172.16.5.108/

[18:49:50] Starting:                                                                
[18:50:26] 200 -   16KB - /contact                                          
[18:50:45] 200 -   14KB - /gallery                                          
[18:51:31] 200 -   15KB - /quote                                            
[18:51:42] 200 -   18KB - /services                                         
[18:52:03] 200 -   17KB - /upload                                           
                                                                             
Task Completed 
```

