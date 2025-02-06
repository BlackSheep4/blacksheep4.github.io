---
layout: single
title: BountyHunter - Hack The Box
excerpt: "BountyHunter es una máquina de nivel medio, con un sistema operativo Linux, que se explota mediante un ataque XXE aprovechando la confiabilidad del servidor con el
input del usuario."
date: 2021-10-17
classes: wide
header:
  teaser: /assets/images/htb-writeup-bountyhunter/bountyhunter_logo.PNG
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
tags:
  - xxe
  - oscp style
  - http
  - medium
---

![](/assets/images/htb-writeup-bountyhunter/bountyhunter_logo.PNG)

BountyHunter es una máquina de nivel medio, con un sistema operativo Linux, que se explota mediante un ataque XXE aprovechando la confiabilidad del servidor con el
input del usuario.

## Ping

Lo primero es ver que podemos hacer ping a la máquina

![](/assets/images/htb-writeup-bountyhunter/1.PNG)

## Escaneo de puertos

Lo primero es lanzar un escáner para averiguar que puertos están abiertos por TCP:

``
nmap -sS --min-rate 5000 --open -vvv -Pn -p- 10.10.11.100 -oG allPorts
``

El resultado es el siguiente:

```
# Nmap 7.91 scan initiated Sun Oct 17 13:52:26 2021 as: nmap -sS --min-rate 5000 --open -vvv -Pn -p- -oN allPorts 10.10.11.100
Nmap scan report for 10.10.11.100
Host is up, received user-set (0.040s latency).
Scanned at 2021-10-17 13:52:26 CEST for 12s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
# Nmap done at Sun Oct 17 13:52:38 2021 -- 1 IP address (1 host up) scanned in 12.13 seconds
```

Tenemos dos puertos conocidos: HTTP y SSH. Con esta información vamos a tratar de enumerar versiones y obtener más información al respecto, antes de visitar la web.

``nmap -sC -sV -p22,80 10.10.11.100 -oN targeted``

El resultado es el siguiente:

```
# Nmap 7.91 scan initiated Sun Oct 17 13:53:30 2021 as: nmap -sC -sV -p22,80 -oN targeted 10.10.11.100
Nmap scan report for 10.10.11.100
Host is up (0.031s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
|   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
|_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Bounty Hunters
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Oct 17 13:53:39 2021 -- 1 IP address (1 host up) scanned in 8.79 seconds
```

En un primer momento, se me ocurre mirar con ``searchsploit`` alguna posible vulnerabilidad relacionada con el servicio Apache en la versión que corre (SPOILER: Por aquí no es).

``searchsploit apache httpd 2.4``

Los resultados son los siguientes:

```
Exploits: No Results
Shellcodes: No Results
Papers: No Results
```

## Recopilación de información de la web

Accedemos a la web via Firefox (o cualquier otro navegador), y vemos la primera toma de contacto:

![](/assets/images/htb-writeup-bountyhunter/2.PNG)

Como pequeño avance, se puede leer justo debajo de la fotografía *Can use Burp*. Bueno, si nos hablan de Burpsuite, podemos imaginarnos que probablemente haya que analizar las
cabeceras web en alguna petición web. Sigamos investigando. La web es bastante estática, a excepción de la sección "PORTAL". Si nos pasamos por ahñi, veremos que nos lleva a
otra sección de la web, donde se puede leer lo siguiente:

![](/assets/images/htb-writeup-bountyhunter/3.PNG)

Seguimos la dirección web que nos indica, y vemos una especie de programa beta que aún está en desarrollo:

![](/assets/images/htb-writeup-bountyhunter/4.PNG)

### Enumeración de rutas con WFUZZ

WFUZZ es una herramienta de fuzzing, para mi, de las mejores junto con dirbuster (aunque personalmente, prefiero WFUZZ). El problema que tiene es que no funciona bien con
páginas web que usan SSL, y esto da para explicar una herramienta muy parecida a WFUZZ pero funcional con SSL, pero esto lo veremos en otro artículo ;)

Con WFUZZ tratamos de enumerar las diferentes rutas que pueda tener la web. Es muy importante enumerar tanto rutas como posibles archivos:

```
wfuzz -c -t 200 --hc=404 --hh=25168 -L -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.11.100/FUZZ         
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.100/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                     
=====================================================================

000000291:   403        9 L      28 W       277 Ch      "assets"                                                                                                                                                                    
000000550:   403        9 L      28 W       277 Ch      "css"                                                                                                                                                                       
000000084:   200        25 L     152 W      2840 Ch     "resources"                                                                                                                                                                 
000000953:   403        9 L      28 W       277 Ch      "js"                                                                                                                                                                        
 /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:80: UserWarning:Finishing pending requests...

Total time: 0
Processed Requests: 2291
Filtered Requests: 2287
Requests/sec.: 0
```

Rápidamente aparecen las primeras rutas. Al parecer, la única ruta a la que podemos acceder es "/resources". El resto lo tenemos prohíbido (403 => Forbidden). Al acceder a la
ruta **http://10.10.11.100/resources** vemos:

![](/assets/images/htb-writeup-bountyhunter/5.PNG)

De todas las rutas, creo que la más llamativa e interesante (al menos a primera vista), es ese *README.txt* que tenemos ahí, esperando a ser leído :)

![](/assets/images/htb-writeup-bountyhunter/6.PNG)

Bueno, al parecer, es una lista de tareas, no saco algo muy claro de aquí.

### Enumeración de archivos web con WFUZZ

Antes de proceder creamos un archivo llamado `extensiones.txt` con el siguiente contenido:

```
php
js
html
txt
```

La idea es indicar algunas rutas comunes usadas en las aplicaciones web. Hecho esto volvemos con wfuzz y ponemos:

```
wfuzz -c -t 200 --hc=404 --hh=25168 -L -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -w extensions.txt http://10.10.11.100/FUZZ.FUZ2Z
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.100/FUZZ.FUZ2Z
Total requests: 882240

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                     
=====================================================================

000000056:   403        9 L      28 W       277 Ch      "html"                                                                                                                                                                      
000000053:   403        9 L      28 W       277 Ch      "php"                                                                                                                                                                       
000001469:   200        5 L      15 W       125 Ch      "portal - php"                                                                                                                                                              
000003389:   200        0 L      0 W        0 Ch        "db - php"                                                                                                                                                                  
```

Encontramos dos rutas, una ya la conocíamos */portal.php*. La otra parece indicar una base de datos, pero recordad, que nmap no había detectado ningún servicio de BBDD.
Tengamos este archivo en mente, lo usaremos próximamente. Vamos a volver con ese programa beta que habiamos visto antes. Aquí me acordé de la pista que nos dejaba el creador
de la máquina: *Can use Burp*.


## XXE Injection
Ponemos carácteres aleatorios, y capturamos la petición web con Burpsuite.

![](/assets/images/htb-writeup-bountyhunter/7.jpg)

Al parecer está haciendo una petición POST a una ruta, llamada **/tracker_diRbPr00f314.php** con un parámetro llamado **data=** que contiene texto URL Encodeado.
Además, parece ser que trata de envíar datos en formato XML. Lo primero es averiguar que contiene **data=**, así que copiamos el contenido del parámetro, y lo redigirimos
al encoder de Burpsuite. Lo desconvertimos de URL Encode, y obtenemos un texto codificado en base64, descodificamos el base64 y obtenemos el contenido descodificado, que resulta ser contenido XML.

![](/assets/images/htb-writeup-bountyhunter/8.PNG)

Otra manera de realizar la descodificación es con CyberChief, una navaja suiza del hacking. https://gchq.github.io/CyberChef/

Este contenido XML son los datos que procesa posteriormente el servidor web para devolvernos una respuesta. Esto implica que la máquina confía en el input del usuario
para enviar datos XML al servidor. Ahora que sabemos como podemos vulnerar el servidor, damos la bienvenida a... XXE Injection.

Para este ataque, he hecho uso de un cheatsheet, dónde se puede ver todas las posibilidades que se pueden hacer con XXE Injection.

https://github.com/payloadbox/xxe-injection-payload-list

Al capturar la petición que se le hace al servidor web, podemos modificarla, para envíar el contenido que nosotros queramos que interprete, en este caso inyectamos nuestro
código malicioso XML.

![](/assets/images/htb-writeup-bountyhunter/9.PNG)

En este caso, tenemos que volver a codificarlo en el mismo proceso que lo hemos descodificado, primero lo pasamos a base64, y luego lo URL Encodeamos.

Lo que estamos haciendo es declarar una entidad, donde de alguna manera le estamos diciendo: "Oye, declaro una entidad que se llame "blacksheep", y que cuando te llame
quiero que me sustituyas "&blacksheep;" con el fichero que contiene las cuentas del sistema". Para llamar a esta entidad, vemos como en mi caso lo llamo en la primera
etiqueta, pero lo puedes hacer en cualquiera de las otras. Para llamar a esta entidad se le indica con: *&llamando_entidad;*.

Al envíar esta petición, el servidor nos responde con lo siguiente:

![](/assets/images/htb-writeup-bountyhunter/10.PNG)

De esta manera, podemos buscar usuarios existentes en el sistema, encontramos al usuario "development", que cuenta con una */bin/bash* y su directorio */home/development*.
Posible usuario potencial identificado.

Os acordáis de **db.php**? Si tratamos de acceder directamente tal y como hemos hecho con */etc/passwd*, no conseguiremos ver nada, ya que al parecer, el fichero está encriptado
y no se puede interpretar. Entonces, hacemos uso de un wrapper PHP para bypassear al control de acceso, y poder acceder al contenido:

![](/assets/images/htb-writeup-bountyhunter/11.PNG)

URL Encodeamos el base64, y lo ponemos como contenido a enviar dentro del parámetro, y PUM! El fichero *db.php* URL Encodeado.

![](/assets/images/htb-writeup-bountyhunter/12.PNG)

Ahora únicamente lo descodificamos el base64, y nos dará unas credenciales de una base de datos.

![](/assets/images/htb-writeup-bountyhunter/13.PNG)

## Acceso a la máquina

Pero... NMAP nos indicaba que no estaba abierto ningún servicio de BBDD. ¿Qué otro servicio teniamos disponible aparte de HTTP?
Correcto SSH. Nos conectamos via SSH con el usuario que habiamos descubierto anteriormente y la contraseña que hemos conseguido.

![](/assets/images/htb-writeup-bountyhunter/14.PNG)

Y ahora podemos listar la flag de usuario:

![](/assets/images/htb-writeup-bountyhunter/15.PNG)

## Escalada de Privilegios

Al hacer un `ls`, veo un fichero llamado *contract.txt*

```
Hey team,
I'll be out of the office this week but please make sure that our contract with Skytrain Inc gets completed.
This has been our first job since the "rm -rf" incident and we can't mess this up. Whenever one of you gets on please have a look at the internal tool they sent over. There have been a handful of tickets submitted that have been failing validation and I need you to figure out why.
I set up the permissions for you to test this. Good luck.
-- John
```

Tratamos de ver si podemos ejecutar como sudo, algún binario: `sudo -l`

```
Matching Defaults entries for development on bountyhunter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
User development may run the following commands on bountyhunter:
    (root) NOPASSWD: /usr/bin/python3.8 
```

Parece que podemos ejecutar como sudo el binario *Python3.8*. Sencillamente ejecutamos el script que dice la nota, que se encuentra en */opt/skytrain_inc/ticketValidator.py*:

![](/assets/images/htb-writeup-bountyhunter/16.PNG)

Al indicarnos la ruta al ticket, le indicamos el archivo .md que se encuenra en */home/development/shell.md*

Y automáticamente nos dará root:

![](/assets/images/htb-writeup-bountyhunter/17.PNG)
 
Rooteado! Gracias por leerme! =)

## Qué se ha aprendido?

- XXE Injection
- Uso de Burpsuite Decoder
- Uso de CyberChief
