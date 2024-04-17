---
date: 2024-04-17 00:25:54
layout: post
title: "Que hace un desarrollador contra los ciberdelincuentes?"
subtitle: Reflexiones sobre la industria, nuestro rol y su evolucion de cara al futuro
description: discusion sobre el futuro de la industria
image: https://sectricity.com/wp-content/uploads/2023/05/Hacker-Cyber-Security-Internet-Sectricity.jpg
optimized_image:
category: Discusion
tags: Charla Just-Chatting
author: ojoconelbyte
paginate: false
---
**Que puede hacer un desarrollador a la hora de desarrollar una aplicación sin fallas de seguridad?** 
Como se puede desde el rol del que crea el programa, pensar un paso adelante para no caer en trampas [no tan "conocidas" para el que viene del desarrollo pero si para los atacantes] y comprometer la integridad digital de los datos de nuestros clientes? 
No tengo ni de lejos la respuesta a preguntas tan amplias en el campo de la tecnología de la información. Pero habiendo tenido varios años de experiencia profesional como desarrollador de WebApps (Fullstack mas que nada backend) y luego aprendiendo a explotar vulnerabilidades (owasp top 10) en modalidad de entrenamientos CaptureTheFlag en plataformas como Hack The Box, para poder lograr acceso al usuario del dominio de una maquina externa, y escalar privilegios hacia un usuario administrador, pienso lo siguiente:

- 1. Falta de chequeo exhaustivos a la hora de sanitizar formularios y cualquier cosa en el sistema donde el usuario que utiliza la web pueda realizar un input. 
Que podemos mejorar? que tipo de variables utilizamos, que permitimos y que no, y más que nada, definir con exactitud para que queremos ese dato y para que se va a usar, sabiendo eso definimos el tipo de dato necesario, por ejemplo un enum, si queremos listar 3 opciones para elegir, sin la necesidad de que sea solo un string o cadena de texto, en donde el usuario puede llegar a ejecutar código malicioso si no se chequea su input a la hora de ejecución. 

- 2. Falta de estar al día con las CVEs y Modus Operandi de atacantes actuales. 
Hay que estar al día con los IOCs (indicator of compromise) y Vulnerabilidades y Exposiciones Comunes (CVEs) para saber que todo el software Open-Source tiene versiones vulnerables a distintas exploits segun el vector que elija el atacante. 
Por ejemplo un ejemplo de primera sería que si buscamos en searchsploit ssh user enumeration, podemos ver que: 
OpenSSH 2.3 < 7.7 - Username Enumeration
OpenSSH 2.3 < 7.7 - Username Enumeration
OpenSSH 7.2p2 - Username Enumeration
OpenSSHd 7.2p2 - Username Enumeration
todas esas versiones de ssh son vulnerables a filtrar datos de nombres y credenciales de usuarios. 

- 3. Uso de Static Analysis, y automatización de chequeos por más superficiales que sean. 
Si bien es verdad que el Blue Team o la parte de seguridad defensiva, tiene que olvidarse de parchear un agujero y ya puede ser victima de un ciberataque, pero a la vez, una actitud proactiva puede detener o relentizar muchísimos ataques. 
Usar linters, analisis de codigo estatico, y crear scripts propios para analizar exploits, si hay problemas de variables y tipados en nuestro repo, no guardar credenciales en texto claro o algoritmos de hasheados debiles no preparados para ser la unica linea de defensa hacia la desencriptacion de un ciberdelincuente, etc. Cuanto más podamos utilizar la tecnología inteligentemente, proactivamente y de manera creativa, más podemos alejar los problemas a futuro. 

Estos tres son consejos basicos y para nada exhaustivos, luego hay medidas de seguridad de prevencion en torno al blue team, donde tenemos que saber analizar nuestra pc de trabajo, un poco de analisis forense de memoria y recursos es necesario, tener antivirus como Windows Defender y scanners como malwareBytes, si tenemos que descargar / modificar / analizar archivos de origen sospechoso o con posible codigo malicioso, aprovecharse de los entornos de virtualizacion como VirtualBox o VMWare, usar contraseñas robustas y tener un password manager, autenticacion de dos factores en nuestras cuentas, etc. 

Realmente hay que ser proactivo, mas como desarrollador en donde potencialmente una linea sola de codigo puede arruinar toda una empresa si es la responsable por filtrar una base de datos por ejemplo. Sanitizar inputs del usuario es lo principal al lado de saber si las versiones y software que utilizamos tiene ya exploits publicos (CVEs) a la vez de saber si nos han comprometido (IOCs).

Cada vez supongo que habra mas ataques, a la vez mas defensas, pero sobre todo nuevos vectores de ataque, con la evolucion constante de la tecnologia, y un rol cada vez mas importante que ocupa en el dia a dia de TODO el planeta.

Espero encontrarlos en proximas publicaciones y que les haya gustado esta! 
Gracias por leer!

Juan - xjxcxn3lbyte
