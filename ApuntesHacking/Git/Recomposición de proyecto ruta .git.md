
Al realizar un fuzzing de directorios web encontramos un directorio "`.git`"

![[Pasted image 20250802112155.png]]

### GitDumper
****
Vamos a recomponer los archivos del proyecto de la página haciendo uso de la herramienta `GitDumper`.

```bash
wget https://raw.githubusercontent.com/internetwache/GitTools/refs/heads/master/Dumper/gitdumper.sh
chmod +x gitdumper.sh
```

Vamos a recomponer el proyecto de la siguiente forma.

```
./gitdumper.sh http://<IP>/.git ./project
```

![[Pasted image 20250802120301.png]]

![[Pasted image 20250802120604.png]]
### GitExtractor
****
Ahora, con la herramienta `GitExtractor` extraeremos los archivos del proyecto web de la máquina.

```bash
wget https://raw.githubusercontent.com/internetwache/GitTools/refs/heads/master/Extractor/extractor.sh
chmod +x extractor.sh
```

```
./extractor.sh project ./fullproject
```

![[Pasted image 20250802121631.png]]

![[Pasted image 20250802121704.png]]

