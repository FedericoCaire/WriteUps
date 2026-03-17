### Escaneo de puertos

~~~
sudo nmap -sS -Pn 10.64.180.74
~~~
- -sS - Envía Paquetes TCP Syn sin completar el handshake para mejorar la velocidad del escaneo
- -Pn - Considera el host como activo

<img width="1303" height="439" alt="Image" src="https://github.com/user-attachments/assets/5c5242a5-3c07-4bc6-a028-431849362a31" />

Se hallaron los siguientes puertos abiertos:
- TCP 21 - FTP
- TCP 22 - SSH
- TCP 80 - HTTP

### Enumeración

Luego de ver que puertos se encuentran abiertos buscamos enumerar qué servicios corren sobre ellos

<img width="1783" height="1228" alt="Image" src="https://github.com/user-attachments/assets/2a977a32-d28d-4f55-abd0-917db751c2d1" />

Lo primero que se visualizo es que el sitio web principal se encuentra en desarrollo, por lo que no había mucho por explorar

<img width="1603" height="659" alt="Image" src="https://github.com/user-attachments/assets/65d38c78-28e8-4c02-92f3-6bc2f82029c1" />

Hallamos que la maquina victima corre sobre Ubuntu, exponiendo los servicios:
- vsftpd 3.0.3
- OpenSSH 7.2p2
- Apache httpd 2.4.18

Luego se investigó las versiones de los servicios corriendo en busca de alguna vulnerabilidad pública conocida pero no se encontró ningún exploit útil

<img width="1603" height="659" alt="Image" src="https://github.com/user-attachments/assets/53eae65f-d54d-49ff-873b-64fa0d539e41" />

Se realizó una conexión exitosa al servidor FTP con el usuario *anonymous*, dentro de este servidor se encontró un archivo *.txt*, una imagen *.jpg* y un directorio llamado ftp/

<img width="1615" height="180" alt="Image" src="https://github.com/user-attachments/assets/8454b88e-ec5f-4e9b-9f3e-560d4d86ec7c" />

<img width="777" height="540" alt="Image" src="https://github.com/user-attachments/assets/07f1af3f-7964-428e-965a-1044c184c598" />

En el contenido del archivo *.txt* se encontró un mensaje entre desarrolladores, hablando sobre el meme adjunto en la imagen *.jpg*. Pero no se encontró ninguna información útil para ganar acceso a la máquina

<img width="1555" height="907" alt="Image" src="https://github.com/user-attachments/assets/d94ad611-7ee2-4a4d-9842-0ca23beacab3" />

<img width="451" height="377" alt="Image" src="https://github.com/user-attachments/assets/c6e50988-bc93-4163-8cba-cfc6c62a3404" />

Se realizó una enumeración de directorios en la página web en donde se encontró el directorio /files/ el cual corresponde al directorio raíz del servidor FTP

<img width="1601" height="508" alt="Image" src="https://github.com/user-attachments/assets/671fb380-bf5d-41b5-858e-f120886ad3f8" />

Se probó exitosamente subir un archivo en el directorio */ftp/* del servidor FTP el cual puede ser visto desde la web en la ruta */files/ftp/*. Esta falta de configuración puede ser útil para ganar acceso inicial a la máquina víctima.

### Explotación

Sabiendo que el servidor web es Apache es muy probable que corra archivos .php, por lo que se busco una reverse shell escrita en php del paquete webshells de kali linux

<img width="1614" height="96" alt="Image" src="https://github.com/user-attachments/assets/00f56802-f56b-4b75-aaeb-615241bcb006" />

<img width="1621" height="1140" alt="Image" src="https://github.com/user-attachments/assets/5400fc2a-f861-45bf-a590-cae4f3d37de9" />

Se modificó el script para que la reverse shell sea enviada hacia nuestra ip y posteriormente se puso el puerto indicado en escucha dentro de la máquina atacante en espera de la ejecución del script

<img width="1151" height="126" alt="Image" src="https://github.com/user-attachments/assets/60033f89-8950-4b08-b93c-5225c44e93a8" />

<img width="798" height="292" alt="Image" src="https://github.com/user-attachments/assets/5bd7f027-ff3d-41db-91ab-a19c6ac9442d" />

<img width="469" height="333" alt="Image" src="https://github.com/user-attachments/assets/a06e9aa6-9bb8-4066-8310-9dbe0323dac6" />

Se subió el script de reverse shell en php dentro del servidor ftp exitosamente.

<img width="1611" height="446" alt="Image" src="https://github.com/user-attachments/assets/5a5b6b5d-a115-4657-9a4c-7672454d62eb" />

<img width="187" height="143" alt="Image" src="https://github.com/user-attachments/assets/f6c6b5c5-9c0c-48c8-824e-4f34858c71cf" />

Luego de abrir el archivo dentro del navegador se ejecutó el script exitosamente, obteniendo así acceso a la máquina.

### Post Explotación

Lo primero que realizó fue obtener la primera flag en el archivo *recipe.txt*

<img width="1604" height="274" alt="Image" src="https://github.com/user-attachments/assets/5db636e4-2b58-49c4-8ca5-7ebec94f1d17" />

Luego se buscó sin éxito desde el directorio raíz del sistema la segunda flag *user.txt*

<img width="783" height="96" alt="Image" src="https://github.com/user-attachments/assets/9f26e667-edc9-4fb6-860a-fcb93568127d" />

Se listo el directorio */home* en busca de directorios de otros usuarios en donde se encontró el home del usuario lennie, pero este se encontraba bien protegido, por lo cual no se pudo ingresar al mismo

<img width="1134" height="477" alt="Image" src="https://github.com/user-attachments/assets/8a8a8bd1-94c2-4f7c-82e8-cc9f1a5fff32" />

Teniendo conocimiento de la existencia del usuario lennie, se intentó realizar fuerza bruta sin éxito vía ssh con la herramienta hydra.

<img width="1601" height="935" alt="Image" src="https://github.com/user-attachments/assets/2111aa48-0f48-4f30-a45f-49454e376c35" />

Indagando por el directorio raíz se encontró un directorio no usual llamado incidents/ el cual contenía un archivo de captura de paquetes llamado *suspicious.pcapng*

<img width="311" height="268" alt="Image" src="https://github.com/user-attachments/assets/a02dce07-c018-4af0-805f-e0faedd11080" />

Se extrajo el archivo por el directorio */files/ftp* del servidor Apache para posteriormente ser analizado desde la máquina atacante

<img width="376" height="82" alt="Image" src="https://github.com/user-attachments/assets/095b780f-37a6-477c-b1fb-16d9b5ceb9ec" />

<img width="428" height="387" alt="Image" src="https://github.com/user-attachments/assets/d148b5a4-663b-40cb-b1f8-296447f0f003" />

Dentro de la captura de paquetes se pudo visualizar un atacante dentro de la maquina victima con el usuario *www-data* el cual busca realizar sin éxito un movimiento lateral con otro usuario del sistema llamado vagrant utilizando la contraseña c4ntg3t3n0ughsp1c3

<img width="1896" height="532" alt="Image" src="https://github.com/user-attachments/assets/5b997e59-f173-4ca4-a8ea-4393bb3095ab" />

Conociendo el usuario *lennie* el cual no fue probado por el atacante de la captura de paquetes visto, probamos realizar el movimiento lateral con el usuario lennie utilizando la contraseña que probaba el otro atacante.

<img width="810" height="359" alt="Image" src="https://github.com/user-attachments/assets/fddbb6e6-2890-4a5f-90b8-d68e1fcce6b8" />

Se logró acceder exitosamente al usuario lennie. Posteriormente se obtuvo la flag user.txt

<img width="315" height="92" alt="Image" src="https://github.com/user-attachments/assets/e86de0a5-4938-4e8a-8b30-e7797a0a9a9a" />

<img width="381" height="68" alt="Image" src="https://github.com/user-attachments/assets/e9d95e6a-5bfd-4e7c-a198-23de3c7719c0" />

Una vez logueados como el usuario lennie se empezó a buscar la forma de escalar los privilegios. Lo primero que llamo la atencion fue, dentro del directorio home del usuario, una carpeta llamada *scripts*

<img width="635" height="175" alt="Image" src="https://github.com/user-attachments/assets/a116fca8-8bc8-4e02-b364-5223175ace00" />

<img width="514" height="134" alt="Image" src="https://github.com/user-attachments/assets/d6dc3186-5326-43b7-b40c-1f4d4180bfa6" />

<img width="527" height="90" alt="Image" src="https://github.com/user-attachments/assets/e2353853-7486-4917-ad79-484062da3981" />

Dentro de la carpeta *scripts* se encontró un script planner.sh, el cual tiene la particularidad que fue creado por el usuario root, lo que puede llegar a ser un cronjob del mismo, un cronjob es una tarea programada la cual se ejecuta cada cierto tiempo previamente programado, y en este caso puede ser que el usuario root lo ejecute cada cierto tiempo, por lo cual nos podemos aprovechar de eso para obtener una shell con su usuario.

El script de bash ejecuta solo 2 comandos, el cual parece más interesante es /etc/print.sh, que en caso de poder ser editado podríamos hacer que el usuario root ejecute lo que queramos

<img width="583" height="93" alt="Image" src="https://github.com/user-attachments/assets/559fa518-e5d7-4494-9a0d-a9d7f12053c6" />

Viendo las propiedades del archivo */etc/print.sh* se encontro que es propiedad del usuario lennie, quien posee permisos de edición del mismo

<img width="798" height="56" alt="Image" src="https://github.com/user-attachments/assets/64145ba4-9e8f-49a7-972a-10160fbe15de" />

Se sobreescribio el script /etc/print.sh con una reverse shell escrita en bash dirigida a la máquina atacante, la cual puso el puerto 4444 en escucha.

<img width="596" height="249" alt="Image" src="https://github.com/user-attachments/assets/565732ca-0f47-45ca-987d-18b32d3f22db" />

En menos de 1 minuto se ejecutó el cronjob enviando una shell con el usuario root hacia la máquina atacante, permitiendo así obtener la última flag y finalizar el CTF.














