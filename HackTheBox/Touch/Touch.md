#ctfs 
![[Pasted image 20261010121325.png]]


Dados fornecidos:

| Endereço de IP         | 10.129.108.204                    |
| ---------------------- | ------------------------------- |
| Credenciais de reserva | **Jenny Crawford** / **KS7X2M** |

# Reconhecimento Inicial

Primeiro começamos com um scan `nmap`no IP fornecido

```shell
└─$ nmap 10.129.108.204 -Pn --open -sC -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-10 12:13 -03
Nmap scan report for 10.129.108.204
Host is up (0.15s latency).
Not shown: 65531 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
135/tcp  open  msrpc
3389/tcp open  ms-wbt-server
5985/tcp open  wsman
8443/tcp open  https-alt
|_http-cors: GET POST PUT OPTIONS
| http-title: Nexion DeviceHub - Login
|_Requested resource was /login

Nmap done: 1 IP address (1 host up) scanned in 353.33 seconds
```

Temos 4 portas abertas. Podemos realizar um outro scan a fim de verificarmos as versões dos serviços envolvidos.

```shell
└─$ nmap 10.129.108.204 -Pn --open -sV -p 135,3389,5985,8443
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-10 12:20 -03
Nmap scan report for 10.129.108.204
Host is up (0.27s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Service
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8443/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.56 seconds
```

Observe que se trata de uma máquina Windows. Com esse reconhecimento inicial já podemos avançar para a enumeração individual.

# Enumeração

## Serviço Web porta 8443

Ao abrirmos o link no navegador nos deparamos com a seguinte página:

`http://10.129.108.204:8443/login`

![[Pasted image 20261010122247.png]]

Podemos iniciar com um scan de diretórios a fim de ver pastas escondidas no endereço do site

```shell
└─$ dirsearch -u http://10.129.108.204:8443 -w /usr/share/wordlists/dirb/big.txt -x 404,500 -r   
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                    
 (_||| _) (/_(_|| (_| )                                                             
                                                                                    
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 20469

Output File: /home/pentecostes/reports/http_10.129.108.204_8443/_26-10-10_12-23-50.txt

Target: http://10.129.108.204:8443/

[12:23:50] Starting:                                                                
[12:24:11] 403 -   35B  - /api                                              
[12:25:04] 200 -  452B  - /favicon.ico                                      
[12:25:33] 200 -    3KB - /login  
```

Vamos aprofundar nosso scan no diretório `/api`

```shell
└─$ dirsearch -u http://10.129.108.204:8443/api/ -w /usr/share/wordlists/dirb/big.txt -x 404,500 -r 
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                    
 (_||| _) (/_(_|| (_| )                                                             
                                                                                    
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 20469

Output File: /home/pentecostes/reports/http_10.129.108.204_8443/_api__26-10-15_13-52-16.txt

Target: http://10.129.108.204:8443/

[13:52:16] Starting: api/                                                           
[13:54:24] 405 -   30B  - /api/scan                                         
[13:54:37] 200 -  116B  - /api/status                                       
                                                                             
Task Completed 
```

Observe que retornou dois resultados, `/api/scan`e `/api/status`.

Podemos analisar a /status que retornou 200 e verificar o que foi encontrado

`http://10.129.108.204:8443/api/status`

![[Pasted image 20261015141211.png]]

Temos aqui um numero de série (`NX-DH-2024-B7042`)! Podemos testá-lo na página inicial novamente e conseguiremos acesso ao dashboard

![[Pasted image 20261015141240.png]]

Observe que tem um nome de usuário e uma senha expostos (`KioskUser`, `[senha]`), podemos tentar um login no RDP com essas credenciais encontradas.

```shell
└─$ xfreerdp /v:10.129.108.204 /u:'KioskUser' /p:'[senha]'' +clipboard
```

Com isso, abre-se essa página no RDP:

![[Pasted image 20261015145743.png]]

Pede-se o código de reserva, o qual foi fornecido no enunciado do CTF

![[Pasted image 20261015145850.png]]

![[Pasted image 20261015235959.png]]

![[Pasted image 20261016000344.png]]

![[Pasted image 20261016000415.png]]