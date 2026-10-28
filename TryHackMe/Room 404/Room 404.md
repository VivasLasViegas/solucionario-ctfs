#ctfs

| IP inicial | 10.67.169.101 |
| ---------- | ------------- |
# Reconhecimento inicial

## Scan completo de portas

Comecei com um scaneamento completo das portas TCP do IP fornecido, utilizando o comando:

```shell
nmap -Pn --open -p- 10.67.169.101
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-27 21:17 -03
Nmap scan report for 10.67.169.101
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 45.00 seconds
```

O resultado revelou duas portas abertas: `22/tcp`, referente ao SSH, e `8080/tcp`, identificada inicialmente como HTTP proxy.

## Identificação dos serviços

Com as portas encontradas, realizei um novo scan para identificar as versões dos serviços e executar os scripts padrão do nmap:

```shell
nmap -Pn --open -p 22,8080 -sVC 10.67.169.101
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-27 21:18 -03
Nmap scan report for 10.67.169.101
Host is up (0.13s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 9f:e2:1d:cd:6f:24:e6:d7:70:a7:1c:57:3f:bd:ac:da (ECDSA)
|_  256 ab:df:a1:db:38:2a:83:fb:0a:85:2e:ac:db:e0:a4:d4 (ED25519)
8080/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
| http-git:
|   10.67.169.101:8080/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: initial Byte Lotus guest platform
|_http-title: Byte Lotus &mdash; Stay Noticed
|_http-server-header: Werkzeug/3.0.1 Python/3.12.3
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.67 seconds
```

O SSH utiliza `OpenSSH 9.6p1 Ubuntu 3ubuntu13.16`. Na porta `8080`, há uma aplicação HTTP servida por `Werkzeug 3.0.1` em Python `3.12.3`, cujo título é **Byte Lotus — Stay Noticed**. O nmap também identificou um repositório Git exposto em `/.git/`.

# Enumeração

## Serviço Web

### Identificação do .git exposto

Observe que no scan `nmap` inicial temos um diretório `.git` exposto. Ao acessarmos ele no navegador nos revela a seguinte página

`http://10.67.169.101:8080/.git`

![Listagem do diretório .git](Imagens/Pasted%20image%2020261027232928.png)

Podemos reconstruir o git através da ferramenta `git-dumper`!

```shell
└─$ git-dumper http://10.67.169.101:8080/.git ./repo
[-] Testing http://10.67.169.101:8080/.git/HEAD [200]
[-] Testing http://10.67.169.101:8080/.git/ [200]
[-] Fetching .git recursively
[-] Fetching http://10.67.169.101:8080/.git/ [200]
[-] Fetching http://10.67.169.101:8080/.gitignore [404]
[-] http://10.67.169.101:8080/.gitignore responded with status code 404
[-] Fetching http://10.67.169.101:8080/.git/logs/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/ [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/ [200]
[-] Fetching http://10.67.169.101:8080/.git/config [200]
[-] Fetching http://10.67.169.101:8080/.git/HEAD [200]
[-] Fetching http://10.67.169.101:8080/.git/description [200]
[-] Fetching http://10.67.169.101:8080/.git/branches/ [200]
[-] Fetching http://10.67.169.101:8080/.git/COMMIT_EDITMSG [200]
[-] Fetching http://10.67.169.101:8080/.git/info/ [200]
[-] Fetching http://10.67.169.101:8080/.git/index [200]
[-] Fetching http://10.67.169.101:8080/.git/refs/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/0a/ [200]
[-] Fetching http://10.67.169.101:8080/.git/logs/HEAD [200]
[-] Fetching http://10.67.169.101:8080/.git/logs/refs/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/fa/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/0f/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/25/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/info/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/a5/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/pack/ [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/applypatch-msg.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/commit-msg.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/fsmonitor-watchman.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/post-update.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/pre-applypatch.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/pre-push.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/pre-receive.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/pre-merge-commit.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/pre-commit.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/pre-rebase.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/prepare-commit-msg.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/update.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/sendemail-validate.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/hooks/push-to-checkout.sample [200]
[-] Fetching http://10.67.169.101:8080/.git/info/exclude [200]
[-] Fetching http://10.67.169.101:8080/.git/refs/heads/ [200]
[-] Fetching http://10.67.169.101:8080/.git/refs/tags/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/0a/12caa4e52a965e89e5eccf5760924b21aacbf7 [200]
[-] Fetching http://10.67.169.101:8080/.git/logs/refs/heads/ [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/fa/45dbd69394ea9e13683d9efb6a0220daac59d4 [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/25/75ab073f67615a27135663ed36794c2d2584fb [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/a5/965c580fee91d852e5b19a8290da02d2926523 [200]
[-] Fetching http://10.67.169.101:8080/.git/objects/0f/13550b4cb13e9f30c61d5b342c532d21e45bda [200]
[-] Fetching http://10.67.169.101:8080/.git/refs/heads/main [200]
[-] Fetching http://10.67.169.101:8080/.git/logs/refs/heads/main [200]
[-] Sanitizing .git/config
[-] Running git checkout .
Updated 3 paths from the index
```

Temos o repositório reconstruído, vamos ver o que tinha nele.

```shell
└─$ ls
app.js  index.html  README.md
```

Temos aqui 3 arquivos, podemos ler o `README.md`

```shell
└─$ cat README.md
# Byte Lotus — Guest Experience Platform

Internal staging repository for the guest app and concierge personalization
service. Do not deploy this folder to production.

Staging flag (remove before launch): THM{FLAG}
```

Aqui está nossa flag, e assim termina-se nosso CTF.
