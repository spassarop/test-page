---
layout: default
title: 1. Introducción
permalink: /intro/
nav_order: 3
---

# Introducción

## ¿Qué es una inyección SQL (SQLi)?

La inyección SQL (*SQL injection*, **SQLi**) es una vulnerabilidad que permite a un atacante interferir con las sentencias que una aplicación ejecuta al interactuar con su base de datos, asumiendo que éstas son generadas con el lenguaje de consulta SQL. 

Las fallas de SQLi se introducen cuando los desarrolladores crean consultas a la base de datos construidas dinámicamente con concatenaciones de *strings*, o transformaciones similares, que incluyen datos provenientes de un usuario de la aplicación.

## Impacto

Generalmente, una SQLi permite al atacante **visualizar datos** que normalmente no podrán ser consultados, tales como datos de otros usuarios o internos de la aplicación en si misma. Usualmente también es posible **modificar o eliminar** estos registros.

Transformando esto en impacto, un atacante puede causar cambios persistentes en el **contenido o comportamiento** de la aplicación. También acceder a **información confidencial** para llevar a cabo otros ataques más complejos o hacerla pública con fines comerciales o daños de reputación/imagen. Dependiendo de la explotabilidad, sería posible escalar un ataque de SQLi para **comprometer el servidor** de base de datos e infraestructura adyacente, o ejecutar ataques de **denegación de servicio**.
