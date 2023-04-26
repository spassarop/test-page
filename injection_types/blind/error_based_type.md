---
layout: default
title: 4.2 Basadas en error
permalink: /injection_types/blind/error_based_type
nav_order: 2
parent: 4. Inyecciones ciegas
has_children: true
---

# Inyecciones ciegas basadas en error

La detección se complica. La aplicación no se comporta de manera diferente agregando condiciones extra que se esperaría que puedan terminar en una consulta SQL. En esta situación, es posible **inducir a la aplicación a devolver respuestas condicionales activando errores SQL condicionalmente**, dependiendo de una condición inyectada. Esto implica modificar la consulta para que **cause un error en la base de datos** si la condición es verdadera, pero no si la condición es falsa (o al revés). Un error del DBMS no manejado por la aplicación causará alguna diferencia en la respuesta (como un mensaje de error), lo que permite inferir la veracidad de la condición inyectada.

Con el mismo ejemplo de consulta que en [4.1 Con respuestas condicionales](/injection_types/blind/conditional_type), se envían estos dos *payloads* de inyección: 

```  
...Tj4' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a 
...Tj4' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```  

Ambos usan una sentencia `CASE` para **probar una condición y devolver una expresión diferente** dependiendo de si la misma es verdadera. Con la primera entrada, el `CASE` evalúa a `'a'`, que no causa ningún error, pero en la segunda entrada, se evalúa la ecuación `1/0`, que causa un error (*Zero división error*). **Suponiendo que el error causa una diferencia** en la respuesta HTTP obtenida, podemos utilizar las diferencias de los códigos de respuesta HTTP para identificar si la condición inyectada es verdadera **y a partir de allí automatizar la inyección**.

Usando esta técnica se pueden extraer datos de la manera descrita anteriormente:

```  
...Tj4' AND (SELECT CASE WHEN 
    (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') 
    THEN 1/0 ELSE 'a' END FROM Users) = 'a
```  

Información básica sobre variantes de sintaxis para generar errores condicionales en distintos DBMS:

| Oracle | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual` |
| Microsoft | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END` |
| PostgreSQL | `1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)` |
| MySQL | `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` |

La sintaxis no siempre va a ser tan directa de aplicar y la interpretación de algunos paréntesis o condiciones de error pueden cambiar para un mismo DBMS en diferentes versiones.