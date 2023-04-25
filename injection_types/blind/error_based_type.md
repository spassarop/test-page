---
layout: default
title: 4.2 Basadas en error
permalink: /injection_types/blind/error_based_type
nav_order: 2
parent: 4. Inyecciones ciegas
has_children: true
---

# Inyecciones ciegas basadas en error

Suponga que la aplicación realiza la misma consulta SQL, pero no se comporta de manera diferente dependiendo de si la consulta devuelve algún dato.

En esta situación, a menudo es posible inducir a la aplicación a devolver respuestas condicionales activando errores SQL condicionalmente, dependiendo de una condición inyectada. Esto implica modificar la consulta para que cause un error en la base de datos si la condición es verdadera, pero no si la condición es falsa. Muy a menudo, un error no manejado por la base de datos causará alguna diferencia en la respuesta de la aplicación (, como un mensaje de error ), lo que nos permitirá inferir la verdad de la condición inyectada.

Suponga que envía 2 solicitudes: 
```  
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a 
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```  
Ambas usan una sentencia CASE para probar una condición y devolver una expresión diferente dependiendo de si la misma es verdadera. Con la primera entrada, el CASE evalúa a 'a' , que no causa ningún error, pero en la segunda entrada, se evalúa la ecuación 1/0, que causa un error (Zero división error). suponiendo que el error causa una diferencia en la respuesta HTTP obtenida, podemos utilizar las diferencias de los códigos de respuesta HTTP para conocer si la condición inyectada es verdadera y partir de allí para automatizar la inyección.

Usando esta técnica podemos extraer datos de la manera descrita anteriormente. 
```  
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```  

Puede encontrar diferentes formas de probar estos errores condicionales, en diferentes bases de datos en el [siguiente enlace](https://portswigger.net/web-security/sql-injection/cheat-sheet).