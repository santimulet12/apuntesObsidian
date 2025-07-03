El binario que tenemos es **command_exec**.
![[Pasted image 20250626141825.png]]
![[Pasted image 20250626142017.png]]
Vemos que al colocar muchos caracteres se produce una **Violación de segmento**, y al ver la key = 41414141, deducimos que el RIP se ha rellenado por la letra "A" , su valor Hexadecimal es "41".
Lo siguiente que haremos es crear un pattern con un módulo de Metasploit llamado **pattern_create.rb**, de 400 bytes, lo haremos de la siguiente forma:
```
/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/pattern_create.rb -l 400
```
El script devuelve lo siguiente:
```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A
```
![[Pasted image 20250626142800.png]]
Podemos ver que la **key** (RIP) ahora tiene un valor diferente, **63413563**. Teniendo en cuenta este valor del RIP, buscaremos el byte exacto donde el programa crashea, buscaremos el offset. Para ello haremos uso del módulo de Metasploit llamado **pattern_offset.rb**, este módulo recibirá  la longitud del pattern que usamos anteriormente (**400**) y el RIP que obtuvimos (**63413563**). Lo ejecutaremos de la siguiente forma:
```
/opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/pattern_offset.rb -l 400 -q 63413563
```
![[Pasted image 20250626143534.png]]
Ahora que ya sabemos que el programa explota en el caracter **76**, probaremos modificar el RIP, los siguientes 4 chars. Para ello crearemos un pattern de 76 caracteres y agregaremos 4 chars para modificar el RIP, lo haremos de la siguiente forma:
```
 /opt/metasploit-framework/embedded/bin/ruby /opt/metasploit-framework/embedded/framework/tools/exploit/pattern_create.rb -l 76
```
pattern de 76:
```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4A
```
Probaremos modificar el RIP, con "BBBB" al final de la siguiente forma:
```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4ABBBB
```
![[Pasted image 20250626144102.png]]
Vemos que la **key**=42424242, "42" es el valor de "B" en Hexadecimal.
Ahora que tenemos el **offset (76)**  y el **RIP (63413563)**. Crearemos un script en python que modifique la key, para que valga **dead**:
![[Pasted image 20250626151422.png]]
Este código, envía el patern de 76 bytes y modifica el RIP (es como el EIP pero para bins 64bits) para que valga **dead** pero en formato Little Endian, de la siguiente forma:
```
dead > ad de > \xad\xde
```
Al ejecutar el script, vemos lo siguiente:
![[Pasted image 20250626151657.png]]
El comando **whoami** se ha ejecutado satisfactoriamente, por lo tanto, podemos reemplazar el comando **whoami** por **chmod u+s /bin/bash** para brindarle privilegios SUID a la **bash** y asi poder escalar privilegios.
Nos llevamos el código a la maquina victima
![[Pasted image 20250626152214.png]]
Hacemos los cambios mencionados anteriormente
![[Pasted image 20250626152339.png]]
Ejecutamos el script:
![[Pasted image 20250626152518.png]]
Vemos que la **bash** ahora tiene privilegios **SUID**, ahora ejecutaremos el siguiente comando para escalar privilegios como root:
```
/bin/bash -p
```
![[Pasted image 20250626152641.png]]
