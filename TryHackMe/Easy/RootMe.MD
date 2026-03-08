### Escaneo de Puertos

Como primer paso se realizó un escaneo de todos los puertos de la
máquina víctima con la herramienta nmap

``` shell
nmap -sS -Pn -p- 172.17.0.2
```

-   -sS - Envía paquetes TCP Syn sin completar el handshake para mejorar
    la velocidad del escaneo
-   -Pn - Considera el host como activo

<img width="737" height="211" alt="Image" src="https://github.com/user-attachments/assets/56bc098f-b912-4a8b-895f-ad834efa4076" />
Se hallaron los siguientes puertos abiertos:

-   TCP 22 - SSH
-   TCP 80 - HTTP

### Enumeración

Luego de ver qué puertos se encuentran abiertos buscamos enumerar qué
servicios corren sobre ellos

<img width="804" height="314" alt="Image" src="https://github.com/user-attachments/assets/1cd91240-df40-46fa-b191-0021f92c6310" />
Hallamos que la máquina víctima corre sobre Ubuntu, exponiendo los
servicios

-   OpenSSH 8.2p1
-   Apache httpd 2.4.41

Investigando manualmente sobre los directorios básicos web comunes (css,
js, etc.) se encontró que hay dos archivos de estilos css que hacen
referencia a la página donde se utilizan.

<img width="439" height="346" alt="Image" src="https://github.com/user-attachments/assets/14f567a3-706d-4321-9c6c-0185fe7e2c77" />
El archivo que llama la atención es panel.css, por lo que se procedió a
buscar la ruta sobre la página, en donde se encontró el siguiente panel
de subida de archivos:

<img width="897" height="684" alt="Image" src="https://github.com/user-attachments/assets/c07c4bb2-8039-4001-917a-9a63ef1f5635" />
<img width="868" height="648" alt="Image" src="https://github.com/user-attachments/assets/5387f0ad-97b6-444f-80ea-ebe83ce0acd8" />
<img width="578" height="317" alt="Image" src="https://github.com/user-attachments/assets/0e78f64a-1bde-49f7-82ce-2698f7717cb7" />
Se realizó una prueba de subida de archivos en donde se puede ver que
los archivos se almacenan en un directorio de la ruta /uploads/

<img width="469" height="335" alt="Image" src="https://github.com/user-attachments/assets/4f9e0cb2-1bfb-4de4-9f3c-a392f4b0d2c5" />
Se realizó una enumeración de directorios del servidor web en donde
podemos observar que la página principal **/index.php**, lo que nos da
el conocimiento de que el servidor corre archivos php, lo cual nos puede
llegar a permitir ejecutar una reverse shell en la máquina víctima.

<img width="714" height="445" alt="Image" src="https://github.com/user-attachments/assets/4a97d8e4-20f3-4e41-bf41-b6949282e08e" />
Siguiendo esta información, buscamos una reverse shell escrita en php
del paquete webshells de kali linux

<img width="632" height="88" alt="Image" src="https://github.com/user-attachments/assets/dad6a203-cac7-477f-87fa-96c207b848fb" />
Luego de haber copiado el script **php-reverse-shell.php**, lo
modificamos para que envíe la reverse shell a nuestra dirección ip

<img width="801" height="533" alt="Image" src="https://github.com/user-attachments/assets/7e7f59ba-c5a2-47d5-80f3-09ea23d07866" />
### Explotación

Una vez con el archivo **php-reverse-shell.php** listo, se intentó subir
sin éxito al servidor web pero este valida los archivos con extensión
.php, por lo que habría que buscar alguna forma de bypassear esta medida

<img width="745" height="605" alt="Image" src="https://github.com/user-attachments/assets/8a1ec267-2502-4406-9858-c6508d692638" />
<img width="745" height="605" alt="Image" src="https://github.com/user-attachments/assets/12ac74a2-c7d6-422b-971a-8ba246ed62b9" />
Lo primero que se intentó crear un archivo **.htaccess** con el
siguiente contenido:

    AddType application/x-httpd-php .txt

Lo cual indica al servidor apache considerar los archivos .txt como php
y ejecutarlos. Con la finalidad de enviar nuestra misma reverse shell
pero con la extensión .txt

<img width="805" height="120" alt="Image" src="https://github.com/user-attachments/assets/ad839161-6f49-4af6-b9b2-197d1126b6cc" />
<img width="745" height="605" alt="Image" src="https://github.com/user-attachments/assets/2dd6948f-ec95-4ae2-90b4-4480b8364dfd" />
<img width="804" height="684" alt="Image" src="https://github.com/user-attachments/assets/fb1b668a-d9e2-439e-a92c-7177303dd4ef" />
El servidor no tuvo en consideración el archivo **.htaccess**, por lo
que se optó por cambiar de estrategia. Se envió el mismo archivo pero
con la extensión .phtml la cual se ejecuta de misma manera que un .php

<img width="804" height="46" alt="Image" src="https://github.com/user-attachments/assets/ef960a02-d722-4351-a607-5dab0cc11541" />
<img width="879" height="676" alt="Image" src="https://github.com/user-attachments/assets/5a3afa4f-4165-4a2e-a297-47c704bf1924" />
La subida del archivo fue exitosa, al momento de presionar **"Veja!"**,
botón que nos direcciona hacia la ruta del archivo, se ejecutó la
reverse shell enviada, recibiendo así la conexión en la máquina atacante
que estaba escuchando el puerto 1234, ganando así acceso al servidor.

<img width="877" height="222" alt="Image" src="https://github.com/user-attachments/assets/0067f7e3-f70c-4c90-a1cf-54c785abe1d7" />
### Post Explotación

Se obtuvo acceso al servidor con el usuario **www-data**, por lo que
debemos escalar nuestros privilegios. El primer intento fue enumerar los
comandos que podemos ejecutar como sudoer con nuestro usuario actual,
con el comando:

``` bash
sudo -l
```

<img width="878" height="246" alt="Image" src="https://github.com/user-attachments/assets/207489ce-e01b-40fb-b09b-e20e82029301" />
Sin éxito, lo siguiente a probar fue buscar los archivos con el bit SUID
(Set User ID) activado, lo cual nos permite ejecutar ese archivo con los
permisos del usuario que creó el archivo, que normalmente es el usuario
**root**. Por lo que realizamos la búsqueda con el comando:

``` bash
find / -perm -4000 -type f 2>/dev/null
```

<img width="680" height="397" alt="Image" src="https://github.com/user-attachments/assets/1cdd9763-cc99-4cb0-a470-076d9130c143" />
Exitosamente se encontró el binario de python 2.7, del cual podemos
abusarnos para escalar nuestros privilegios, por lo que se preparó el
siguiente comando para lanzar un bash desde un microscript en python

``` bash
python -c 'import os;os.system("/bin/bash")'
```

<img width="732" height="111" alt="Image" src="https://github.com/user-attachments/assets/e60b3a98-cee0-4511-8615-d9db78ed1ff3" />
La bash se ejecutó exitosamente pero bash dropeó automáticamente
nuestros privilegios ya que detectó que el UID real es diferente del UID
efectivo. Por lo que se modificó el script agregando la flag -p
(preserve privileges) para que esto no ocurra

``` bash
python -c 'import os; os.execv("/bin/bash", ["bash","-p"])'
```

<img width="882" height="118" alt="Image" src="https://github.com/user-attachments/assets/f7044d72-dc15-46ab-aa30-1962c45b3131" />
Y ahora sí se pudo obtener una shell con el usuario root.

### Captura de Flags

<img width="266" height="99" alt="Image" src="https://github.com/user-attachments/assets/a83e42af-fc30-4c2b-a11f-31946323e871" />
<img width="260" height="94" alt="Image" src="https://github.com/user-attachments/assets/995b7ab7-8b98-4e4c-9337-bdd4dc773370" />
