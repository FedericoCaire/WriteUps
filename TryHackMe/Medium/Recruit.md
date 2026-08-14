### **Contexto del CTF**

**Recruit** es una máquina de dificultad *Medium* de la plataforma TryHackMe que simula el lanzamiento de un nuevo portal de reclutamiento corporativo. Esta plataforma web fue diseñada para que el personal de Recursos Humanos gestione las postulaciones de candidatos, mientras que los administradores supervisan y toman las decisiones finales de contratación. El objetivo es poder lograr iniciar sesión como Administrador.

### **Reconocimiento Inicial**

En un primer inicio se logró identificar un panel de login en el cual no hay opción de registro, lo llamativo de este panel fue un botón por debajo `Access API` el cual nos puede proveer alguna información.

<img width="1726" height="1028" alt="Image" src="https://github.com/user-attachments/assets/d2dc3981-a9f4-4db6-9899-d60a786f5081" />

<img width="2337" height="1367" alt="Image" src="https://github.com/user-attachments/assets/37b71acd-7ac0-4947-82df-d7afdbcd5f0b" />

`Access API` lleva a la ruta **/api.php** en donde se encuentran preguntas frecuentes de la API del sistema de reclutamiento, dentro de las respuestas comenta de un endpoint **/file.php** al cual se le puede dar el parámetro **cv** con una URL para que el sistema consulte el archivo en esa dirección. Este feature nos abre dos vías de ataques posible:

* SSRF (Server-Side Request Forgery): Permite forzar que el servidor envíe una request HTTP a un servidor controlado por el lado del atacante, en donde se puede obtener información como un token de sesión  
* Information Disclosure: Permite que el servidor muestre información de algún archivo al cual se requiere autorización, la cual el atacante no posee.

Primero se probo sin exito, un ataque SSRF:

Se puso el puerto 4444 en escucha y posteriormente se probó el endpoint poniendo como parámetro de cv la dirección y puerto de la máquina atacante.

<img width="582" height="96" alt="Image" src="https://github.com/user-attachments/assets/1c13fe2a-acc2-40ae-b36c-2505961dbbb4" />
<img width="1258" height="315" alt="Image" src="https://github.com/user-attachments/assets/aefe3711-b822-4907-b0ca-18702777b8d3" />

Se obtuvo un mensaje `Only local files are allowed`, por lo que se llego a la conclusión de la existencia de una deny list, por lo que se probo posteriormente agregar el prefijo `file://`

<img width="1278" height="314" alt="Image" src="https://github.com/user-attachments/assets/d56fa3b6-40a8-4bf1-9b51-5e5cb1da6458" />


donde se obtuvo una respuesta distinta `Access denied`, por lo que se determinó que no era posible realizar un SSRF por el momento.

### **Enumeración**

Se realizo una enumeración de directorios con la herramienta **gobuster** en busca de algún endpoint o directorio expuesta que sea de utilidad

<img width="2657" height="1321" alt="Image" src="https://github.com/user-attachments/assets/047d5553-086e-4b70-bed5-da39ae95d894" />
~~~
/api.php              (Status: 200\) \[Size: 4151\]  
/assets               (Status: 301\) \[Size: 315\] \[--\> http://10.67.175.234/assets/\]  
/config.php           (Status: 200\) \[Size: 0\]  
/file.php             (Status: 200\) \[Size: 20\]  
/header.php           (Status: 200\) \[Size: 457\]  
/index.php            (Status: 200\) \[Size: 1417\]  
/javascript           (Status: 301\) \[Size: 319\] \[--\> http://10.67.175.234/javascript/\]  
/logout.php           (Status: 302\) \[Size: 0\] \[--\> index.php\]  
/mail                 (Status: 301\) \[Size: 313\] \[--\> http://10.67.175.234/mail/\]  
/phpmyadmin           (Status: 301\) \[Size: 319\] \[--\> http://10.67.175.234/phpmyadmin/\]
~~~

En esta búsqueda se encontraron dos ruta interesante, la primera `mail` la cual redirecciona a un directorio del mismo nombre, donde había un único archivo llamado `mail.log`, y la segunda es un archivo `config.php` el cual se encuentra vacio pero devolvió un estado HTTP 200 OK.

<img width="898" height="669" alt="Image" src="https://github.com/user-attachments/assets/3de9e289-da7f-44f2-bf6b-8dada8b91af9" />

<img width="2051" height="1391" alt="Image" src="https://github.com/user-attachments/assets/9b6e1d5e-2eb9-415e-a670-84160f61611c" />

En el archivo `mail.log` se encontró un mail enviado por HR Operations hacia el departamento de IT-Support comentando que el deploy del portal fue exitoso, lo llamativo de este mail es que comenta que las credenciales del usuario **hr** se encuentran almacenadas en el archivo `config.php`, como ya vimos previamente accediendo directamente desde su ruta devuelve un archivo vacío (Size: 0), por lo que es un target ideal para explotar una vulnerabilidad de information disclosure ya identificada con el endpoint de la api `file.php`

<img width="1232" height="1303" alt="Image" src="https://github.com/user-attachments/assets/a199c8a3-8555-4b08-b761-dda2104629c0" />

El resultado del ataque fue exitoso, con lo que se obtuvieron las credenciales **hr:hrpassword123**. Con estas credenciales se ganó acceso inicial al portal web, obteniendo así la primer flag.

<img width="2459" height="1600" alt="Image" src="https://github.com/user-attachments/assets/93d394c6-a993-42bb-bbee-18356519474f" />

### **Escalada de privilegios**

El objetivo del CTF es lograr acceso con rol de administrador, por lo que una vez ganado el acceso inicial, dentro del panel se identificó un buscador por nombres, el cual es potencialmente vulnerable a una inyección sql.

<img width="2459" height="352" alt="Image" src="https://github.com/user-attachments/assets/3cc5ec26-13d1-4b26-9823-b508398fb38e" />


Lo primero que se intento fue realizar una inyección clasica, en busca de ver como se comportaba el sistema.

<img width="2459" height="482" alt="Image" src="https://github.com/user-attachments/assets/1bebf234-84e9-46b3-85b9-785f1d7d212f" />


Este devolvió un error de sintaxis sql, el cual por primera instancia confirma que el buscador es un punto de inyección, y además provee información sobre el motor de base de datos, el cual en este caso es MySql server, lo cual no es de utilidad para utilizar una consulta inyectada acorde.

Como primera instancia se buscó obtener información de las tablas existentes en la base de datos, por lo que se utilizó el siguiente input:

`' UNION SELECT 1,GROUP\_CONCAT(table\_name),3,4 FROM information\_schema.tables WHERE table\_schema=database() \-- \-`

<img width="2272" height="671" alt="Image" src="https://github.com/user-attachments/assets/1c4a067c-2dca-4d9b-af2b-c51c28ae371e" />


Como respuesta se obtuvo que existen dos tablas en las base de datos (candidates y users), por lo que el siguiente paso fue ver los atributos de la tabla `users` en busca de información de logueo que nos permita escalar nuestros privilegios, por lo que se utilizó el siguiente input:

`' UNION SELECT 1,GROUP\_CONCAT(column\_name),3,4 FROM information\_schema.columns WHERE table\_schema=database() AND table\_name='users' \-- \-`

<img width="2272" height="671" alt="Image" src="https://github.com/user-attachments/assets/ac9c73ec-af84-4aa9-a21e-ee705df35462" />

En la respuesta se obtuvo que la base de datos almacena usuario y contraseña en texto plano, por lo que se utilizó el siguiente input para imprimirlas:

`' UNION SELECT 1,GROUP\_CONCAT(username,':',password),3,4 FROM users \-- \-`

<img width="2272" height="671" alt="Image" src="https://github.com/user-attachments/assets/219195ea-f3e5-4ede-a082-b65e03a8a276" />

Con esto se obtuvieron las credenciales **admin:admin@001admin** del usuario administrador del sistema. Con esto se ingresó en la plataforma y se obtuvo la flag final, finalizando asi el desafío.

<img width="1655" height="778" alt="Image" src="https://github.com/user-attachments/assets/b64313ef-d796-4fe4-b0c4-8a25fd3f35ae" />

<img width="2462" height="1441" alt="Image" src="https://github.com/user-attachments/assets/8d4de716-8f92-42c2-bd2b-f1bb73c4b18f" />


