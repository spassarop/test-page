---
layout: default
title: 4.3 Basadas en tiempo
permalink: /injection_types/blind/time_based_type
nav_order: 3
parent: 4. Inyecciones ciegas
has_children: true
---

# Inyecciones ciegas basadas en tiempo

Esta variante utiliza retrasos en la consulta SQL para determinar si la inyección ha sido exitosa. Por ejemplo, se puede enviar una consulta maliciosa que contenga un retraso de una cantidad determinada de tiempo para determinar si la inyección ha sido exitosa.

Continuando con nuestro ejemplo anterior suponga que la aplicación ahora detecta errores de la base de datos y los maneja de una mejor manera, por lo que realizar una consulta no causa ninguna diferencia en la respuesta de la aplicación, En esta situación a menudo es posible explotar un inyección al provocar retrasos de forma condicional en la respuesta del servidor, veamos un ejemplo: 

Las técnicas para activar un retraso de tiempo son muy específicas para el tipo de base de datos que se utiliza. En **Microsoft SQL Server**, la entrada como la siguiente se puede usar para probar una condición y desencadenar un retraso dependiendo de si la expresión es verdadera:

`'; IF (1=2) WAITFOR DELAY '0:0:10'-- '; IF (1=1) WAITFOR DELAY '0:0:10'--`

En este caso la primera entrada no provocara un retaso, al ser falsa la condición, y la segunda condición provocara un retraso de 10 segundos, Partiendo de esta consulta podemos recuperar datos de la forma ya descrita, probando de forma sistematicamente un caracter a la vez:

`'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--`

Para otros tipos de base de datos:
**Oracle :**`dbms_pipe.receive_message(('a'),10)`
**PostgreSQL :** `SELECT pg_sleep(10)`
**MySQL :** `SELECT SLEEP(10)`