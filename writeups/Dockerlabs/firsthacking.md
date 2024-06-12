---
layout: post
title: "FirstHacking"
author: "s0ksy"
---
Estaremos resolviendo una máquina de Dockerlabs de dificultad muy fácil, donde veremos como podemos hacer explotaciones de SQL injectfgfion para obtener la contraseña y el usuario del ssh.

## Reconocimiento

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
* `-p-`: Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
* `--open`: Indicamos que solo queremos que nos muestre los puertos abiertos
* `--min-rate 5000`: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
* `-n`: Para que no se nos aplique resolución DNS (agilizar escaneo también)
* `-Pn`: Para que no nos lance PING, también para hacer el escaneo más rápido

Y encontramos el puerto 21 (ftp) abierto:
![first1](/assets/images/first1.png)

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto). Encontramos la versión del `ftp` que esta corriendo en ese puerto, pero me parece raro que no tengamos acceso por Anonymous
![first2](/assets/images/first2.png)

Por lo tanto, buscamos la versión del servicio en searchsploit para ver si se puede vulnerar eso de alguna forma y encontramos 2 formas de hacerlo (ejecutando el script del exploit manualmente con python o de forma automática con metasploit)
![first3](/assets/images/first3.png)

## Explotación

Decido hacerlo con metasploit
![first4](/assets/images/first4.png)

Y veo que solo nos basta con configurar el HOST de la víctima, algo muy sencillo
![first6](/assets/images/first6.png)

Lo configuramos como toca, es decir, poniendo la ip de la máquina:
![first7](/assets/images/first7.png)

Le damos a run para ejecutar el script y tendriamos acceso ya como root!!
![first8](/assets/images/first8.png)
