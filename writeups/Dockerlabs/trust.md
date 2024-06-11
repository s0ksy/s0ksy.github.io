---
layout: post
title: "s0ksy | Trust"
author: "s0ksy"
---
Estaremos resolviendo una máquina de Dockerlabs de dificultad muy fácil, donde veremos como podemos explotar la contraseña de un ssh haciendo fuerza bruta con la herramienta llamada "Hydra"
 
## Reconocimiento

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
* -p- : Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
* --open: Indicamos que solo queremos que nos muestre los puertos abiertos
* --min-rate 5000: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
* -sS: Para que nos haga un escaneo de tipo SYN, también para agilizar el escaneo y que sea sigiloso
* -n: Para que no se nos aplique resolución DNS (agilizar escaneo también)
* -Pn: Para que no nos lance PING, también para hacer el escaneo más rápido

Podemos observar que tenemos el puerto 22 (ssh) y 80 (http) abiertos:
![recon1](/assets/images/recontrust1.png)

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto).
![recon2](/assets/images/recontrust2.png)

Entramos en la página web y encontramos la página default de Apache, nada interesante
![recon9](/assets/images/recontrust9.png)

Procedemos a hacer fuzzing con gobuster para ver que directorios de interés podemos encontrar de esta página web, indicando con el parámetro -x que queremos que nos los busque con las extensiones indicadas (php, html, txt) y encontramos /secret.php que nos llama la atención:
![recon3](/assets/images/recontrust3.png)

Procedemos a meterlo en la página web y obtenemos un "Hola Mario, Esta web no se puede hackear". Interesante, mario podría ser un usuario para entrar por ssh (puerto 22)
![recon4](/assets/images/recontrust4.png)

## Explotación
Por lo tanto, decido hacerle un ataque de fuerza bruta con "Hydra" indicando el usuario, la wordlist a usar y el servicio de donde es dicho usuario. OBTENEMOS LA CONTRASEÑA DEL SSH!!: chocolate.
![recon5](/assets/images/recontrust5.png)

Entramos por ssh y de primeras hacemos un ls para ver que archivos, directorios... podemos ver a priory, pero nada.
![recon6](/assets/images/recontrust6.png)
## Escalada de privilegios

Vamos a intentar conseguir usuario de SUPERADMIN y hacemos un sudo -l para ver los comandos que podemos usar siendo usuario "mario" y vemos /usr/bin/vim, un bin interesante que lo podemos usar para escalar privilegios

![recon7](/assets/images/recontrust7.png)

Nos vamos a la página de gtfobins https://gtfobins.github.io/gtfobins/vim/#suid y buscamos el bin: vim, para ver que comando podemos ejecutar para esa escalada de privilegios y encontramos lo siguiente:
![recon8](/assets/images/recontrust8.png)

Ponemos el comando por la terminal... y lo logramos ya estamos como usuario ROOT!!
![recon10](/assets/images/recontrust10.png)

