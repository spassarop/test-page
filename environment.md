---
layout: default
title: Ambiente de pruebas
permalink: /environment/
nav_order: 2
---

# Ambiente de pruebas

Para los laboratorios se utilizan las siguientes herramientas:
- Herramienta de pruebas de penetración de aplicaciones web (proxies de ataque):
  - [OWASP ZAP](https://www.zaproxy.org/download/). Para instalarlo e inspeccionar aplicaciones manualmente en un navegador ver en la [guía básica](https://www.zaproxy.org/getting-started/) las secciones “Install and Configure ZAP” y “Exploring an Application Manually” respectivamente.
  - [Burp Suite Community Edition](https://portswigger.net/burp) (versión gratuita). Para información de instalación y uso ver la [guía de PortSwigger](https://portswigger.net/burp/documentation/desktop/getting-started) que contiene imágenes y videos.
- Servicio DNS que pueda escuchar peticiones generadas por el servidor víctima. En este caso se usa Burp Collaborator (parte de Burp Suite Professional), por restricciones de la plataforma de ejercicios.
- [Python3.X](https://www.python.org/downloads/).
- [sqlmap](https://sqlmap.org/).



