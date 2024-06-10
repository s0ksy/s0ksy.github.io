---
layout: post
title: "Injection"
author: "s0ksy"
---

## Reconocimiento

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
* `-p-`: Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
* `--open`: Indicamos que solo queremos que nos muestre los puertos abiertos
* `--min-rate 5000`: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
* `-sS`: Para que nos haga un escaneo de tipo SYN, también para agilizar el escaneo y que sea sigiloso
* `-n`: Para que no se nos aplique resolución DNS (agilizar escaneo también)
* `-Pn`: Para que no nos lance PING, también para hacer el escaneo más rápido

Podemos observar que tenemos el puerto 22 (ssh) y 80 (http) abiertos:
![[Pasted image 20240605165921.png]]

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto).
![[Pasted image 20240605170003.png]]

Entramos en la página web y encontramos la página default de Apache, nada interesante
![[Pasted image 20240605170042.png]]

Procedemos a hacer fuzzing con gobuster para ver que directorios de interés podemos encontrar de esta página web, indicando con el parámetro -x que queremos que nos los busque con las extensiones indicadas (php, html, txt) y encontramos /index.php que puede ser bastante interesante:
![[Pasted image 20240605170243.png]]

Y encontramos un login en el directorio de /index.php, podriamos intentar una inyección SQL
![[Pasted image 20240605170306.png]]

## Explotación

Lo dicho, hacemos la SQLi usando `'or 1 = 1-- -` como user y como password podemos poner cualquier contraseña:
![[Pasted image 20240605170546.png]]

Perfecto!! Entramos y de primeras tenemos un usuario `Dylan` y una constraseña que podriamos usar para conectarnos por SSH:
![[Pasted image 20240605170604.png]]

Vamos a entrar con SSH, y estamos dentro!
![[Pasted image 20240605170934.png]]

## Escalada de privilegios

Intento ver que comandos puedo usar como sudoer pero no tiene el comando de sudo habilitado, por lo tanto lanzamos `find / -perm -4000 2>/dev/null` para poder encontrar archivos con permisos de SUID y de entre ellos observamos `/usr/bin/env` que destaca de entre los demás:
![[Pasted image 20240605171305.png]]

Nos dirigimos a https://gtfobins.github.io/ para poder ver que podemos hacer con ese bin para escalar privilegios y encontramos lo siguiente:
![[Pasted image 20240605171631.png]]

Bueno, lo copiamos en la terminal para escalar privilegios y obtenemos superusuario root!!
![[Pasted image 20240605171641.png]]

