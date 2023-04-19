---
layout: default
title: 2. Detección
permalink: /detection/
nav_order: 4
---

# Detección

## ¿Cómo detectar una inyección?

A pesar de poder ejecutar herramientas automatizadas en toda la aplicación, este enfoque suele tomar mucho tiempo dependiendo de la superficie de ataque (cantidad de *endpoints*, parámetros, cookies, otros cabezales HTTP). Un enfoque manual y más preciso es útil para detectar alguna señal que dé lugar a más pruebas manuales o automatizarlas centradas en potenciales puntos de inyección.

Estas son algunas técnicas de detección manual, que implican enviar a la aplicación:
- El caracter de **comilla simple** (`'`) en busca de errores o cambios de comportamiento.
- **Condiciones lógicas** como `OR 1=1` y `OR 1=2` buscando diferencias en las respuestas.
- Contenido con sintaxis SQL que resulte en **demoras** controladas (ver el punto de inyecciones ciegas).
- Contenido diseñado para disparar **interacciones de red** fuera de banda (*out-of-band*) ejecutadas dentro de la consulta SQL y monitorear con un servicio si se recibe tal pedido de interacción. Implica tener infraestructura preparada (ver el punto de inyecciones ciegas).

## Puntos de inyección

El escenario clásico de SQLi es en la cláusula `WHERE` de una sentencia `SELECT`, aunque pueden ocurrir en cualquier otra parte y con los otros tipos de sentencias. Las ubicaciones más comunes son:
- `UPDATE`: en los valores actualizados o la cláusula `WHERE`.
-  `INSERT`: en los valores insertados.
-  `SELECT`:
   -  en los nombres de tabla o columnas.
   -  en la cláusula `ORDER BY`, en las columnas a ordenar o la dirección (`ASC`/`DESC`).