---
layout: post
title: "BreakMySSH"
author: "s0ksy"
---
Estaremos resolviendo una máquina de Dockerlabs de dificultad muy fácil, donde nos encontraremos ante un servicio SSH sin nada más y tendremos que conseguir el user y la passwd a través de fuerza bruta

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
- -p- : Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
- --open: Indicamos que solo queremos que nos muestre los puertos abiertos
- --min-rate 5000: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
- -sS: Para que nos haga un escaneo de tipo SYN, también para agilizar el escaneo y que sea sigiloso
- -n: Para que no se nos aplique resolución DNS (agilizar escaneo también)
- -Pn: Para que no nos lance PING, también para hacer el escaneo más rápido

Podemos observar que solo tenemos el puerto 22 (ssh) abierto:
![break 1](/assets/images/break1.png)

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto). No vemos nada importante, solo que la version del `ssh` es la `OpenSSH 7.7` que nos podría ser de utilidad.
![break 2](/assets/images/break2.png)

Me voy a descargar un diccionario de github, ya que estuve probando con varios diccionarios nativos de Linux y demás y no me encontraba con ninguno y tuve que preguntar que es lo que me recomendaban. Me recomendaron este diccionario y es el que he usado.
![break 3](/assets/images/break3.png)

Ahora le hacemos un ataque de fuerza bruta con "Hydra" indicando la wordlist a usar para enumerar usuarios, la wordlist a usar para enumerar contraseñas y el servicio de donde es dicho usuario. Nos encuentra ya `root` y la contraseña `estrella`!!
![break 4](/assets/images/break4.png)

Nos conectamos por el servicio de `ssh` introduciendo las credenciales obtenidas anteriormente.... y YA SOMOS ROOT!!
![break 5](/assets/images/break5.png)
