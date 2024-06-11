---
layout: post
title: "s0ksy | Injection"
author: "s0ksy"
---
Estaremos resolviendo una máquina de Dockerlabs de dificultad muy fácil, donde veremos como podemos hacer explotaciones de SQL injection para obtener la contraseña y el usuario del ssh.

## Reconocimiento

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
* `-p-`: Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
* `--open`: Indicamos que solo queremos que nos muestre los puertos abiertos
* `--min-rate 5000`: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
* `-sS`: Para que nos haga un escaneo de tipo SYN, también para agilizar el escaneo y que sea sigiloso
* `-n`: Para que no se nos aplique resolución DNS (agilizar escaneo también)
* `-Pn`: Para que no nos lance PING, también para hacer el escaneo más rápido

Podemos observar que tenemos el puerto 22 (ssh) y 80 (http) abiertos:
![injection1](/assets/images/injection1.png)

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto).
![injection2](/assets/images/injection2.png)

Entramos en la página web y encontramos la página default de Apache, nada interesante
![injection3](/assets/images/injection3.png)

Procedemos a hacer fuzzing con gobuster para ver que directorios de interés podemos encontrar de esta página web, indicando con el parámetro -x que queremos que nos los busque con las extensiones indicadas (php, html, txt) y encontramos /index.php que puede ser bastante interesante:
![injection4](/assets/images/injection4.png)

Y encontramos un login en el directorio de /index.php, podriamos intentar una inyección SQL
![injection5](/assets/images/injection5.png)

## Explotación

Lo dicho, hacemos la SQLi usando `'or 1 = 1-- -` como user y como password podemos poner cualquier contraseña:

![injection6](/assets/images/injection6.png)

Perfecto!! Entramos y de primeras tenemos un usuario `Dylan` y una constraseña que podriamos usar para conectarnos por SSH:

![injection7](/assets/images/injection7.png)

Vamos a entrar con SSH, y estamos dentro!
![injection8](/assets/images/injection8.png)

## Escalada de privilegios

Intento ver que comandos puedo usar como sudoer pero no tiene el comando de sudo habilitado, por lo tanto lanzamos `find / -perm -4000 2>/dev/null` para poder encontrar archivos con permisos de SUID y de entre ellos observamos `/usr/bin/env` que destaca de entre los demás:
![injection9](/assets/images/injection9.png)

Nos dirigimos a https://gtfobins.github.io/ para poder ver que podemos hacer con ese bin para escalar privilegios y encontramos lo siguiente:
![injection10](/assets/images/injection10.png)

Bueno, lo copiamos en la terminal para escalar privilegios y obtenemos superusuario root!!
![injection11](/assets/images/injection11.png)

