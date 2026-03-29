### Escaneo de Puertos

Como primer paso se realizó un escaneo de todos los puertos de la maquina victima con la herramienta nmap

- -sS - Envía Paquetes TCP Syn sin completar el handshake para mejorar la velocidad del escaneo
- -Pn - Considera el host como activo

<img width="727" height="273" alt="Image" src="https://github.com/user-attachments/assets/6309e293-6584-4f0a-b500-f6b11adbb9c0" />

Se hallaron los siguientes puertos abiertos:
- TCP 21 - FTP
- TCP 22 - SSH
- TCP 80 - HTTP

### Enumeración

<img width="936" height="728" alt="Image" src="https://github.com/user-attachments/assets/c7ed83a0-a272-44d8-a73f-2327cc2572c0" />

<img width="1679" height="594" alt="Image" src="https://github.com/user-attachments/assets/05b207d0-ba4a-4c14-997f-21d6b9e75084" />

Revisando la página web se encontró una fan page de Brooklyn Nine-Nine sin ningún tipo de elemento o comentario interesante en su código fuente

<img width="416" height="249" alt="Image" src="https://github.com/user-attachments/assets/5729c4c2-0641-4dc2-94ba-aa15e819a026" />

Se probó ingresar al servicio ftp de la maquina victima con el usuario Anonymous, logrando acceso exitosamente al mismo

<img width="720" height="155" alt="Image" src="https://github.com/user-attachments/assets/bc4e092c-f0f2-4b80-b6ab-708a5164b51f" />

<img width="799" height="142" alt="Image" src="https://github.com/user-attachments/assets/67dd74d9-f79e-4c41-9676-16dfafa22e70" />

Dentro del servicio FTP se encontró una nota en el directorio raíz, la nota era firmada por parte de Amy hacia Jake pidiéndole que cambie su contraseña ya que esta era muy débil.

### Explotación

Teniendo en cuenta que la contraseña del usuario Jake es débil, se podría aprovechar que el servicio SSH está activo para probar contraseñas por fuerza bruta hacia este usuario

<img width="802" height="342" alt="Image" src="https://github.com/user-attachments/assets/6ef3af7b-ee21-4d01-b612-8fb5ac1ced92" />

Se aplicó fuerza bruta al usuario Jake utilizando la herramienta Hydra, exitosamente se encontró la contraseña 987654321

<img width="797" height="279" alt="Image" src="https://github.com/user-attachments/assets/b9ecffad-5be6-45be-928f-87343e39b02e" />

Se obtuvo acceso inicial a la maquina victima via SSH con las credenciales jake:987654321

### Post Explotación

Una vez dentro de la maquina victima hay que analizar como escalar nuestros privilegios

<img width="611" height="183" alt="Image" src="https://github.com/user-attachments/assets/6e0877ef-af87-4152-8a0a-3cf53371dac0" />

<img width="613" height="162" alt="Image" src="https://github.com/user-attachments/assets/f969f985-7b41-4b88-aeff-a4569a458bd3" />

Se encontraron otros dos usuarios en el directorio home, holt y amy, llamativamente se pudo listar el directorio home de amy el cual contaba con su propia clave ssh, pero no se logró obtenerlas de forma simple con el usuario de jake

<img width="495" height="158" alt="Image" src="https://github.com/user-attachments/assets/f102aa06-b209-4be3-9578-6c9f0c0f7f8b" />

Analizando el directorio home de holt se obtuvo la primer flag user.txt, pero nada más interesante para lograr escalada de privilegios o un movimiento lateral

<img width="802" height="183" alt="Image" src="https://github.com/user-attachments/assets/a62554e3-9950-43f2-90ed-2940d3d8e5e9" />

Se listo los comandos posibles de usar con sudo, en donde se encontró que se permite ejecutar **/usr/bin/less** lo cual nos va a permitir obtener acceso root 

~~~ shell
sudo less /etc/hosts
~~~

<img width="338" height="57" alt="Image" src="https://github.com/user-attachments/assets/9cd82b13-940e-4fb6-9de7-a7f9633be413" />

<img width="547" height="225" alt="Image" src="https://github.com/user-attachments/assets/29ea8314-86e9-4c31-b061-25211a78207d" />

El comando less permite visualizar un archivo y por debajo de la terminal permite la ejecución de comandos si se ingresa un signo de exclamación antes, por lo que escribiendo !/bin/bash se obtiene una shell con el usuario root.
Una vez teniendo una terminal con el usuario root se obtuvo la última flag, finalizando así el CTF.







































