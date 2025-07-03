# Guía GEF para Explotación de Buffer Overflow

## 🎯 Comandos Esenciales para Buffer Overflow

### 1. **Análisis Inicial del Binario**

#### `checksec` - Verificar protecciones de seguridad

```bash
gef➤ checksec
```

**Qué muestra:**

- ASLR (Address Space Layout Randomization)
- NX/DEP (Data Execution Prevention)
- Stack Canaries
- PIE (Position Independent Executable)
- RELRO (Relocation Read-Only)

**Interpretación:**

- ✅ **Protección OFF** = Más fácil de explotar
- ❌ **Protección ON** = Necesitas técnicas avanzadas

---

### 2. **Encontrar el Offset del EIP/RIP**

#### `pattern create` - Generar patrón cíclico

```bash
gef➤ pattern create 200
```

**Uso:** Copia el patrón generado y úsalo como input para causar el crash.

#### `pattern search` - Encontrar offset

Después del crash:

```bash
gef➤ pattern search $rip    # Para 64-bit
gef➤ pattern search $eip    # Para 32-bit
```

**Resultado:** Te dice exactamente en qué posición se sobrescribe el registro de retorno.

---

### 3. **Análisis de Registros y Contexto**

#### `context` - Vista completa del estado

```bash
gef➤ context
```

**Muestra automáticamente:**

- Registros actuales
- Stack (pila)
- Código en ejecución
- Heap (si es relevante)

#### `registers` - Solo registros

```bash
gef➤ registers
```

---

### 4. **Exploración de Memoria**

#### `telescope` - Examinar memoria inteligentemente

```bash
gef➤ telescope $rsp          # Examinar stack
gef➤ telescope $rsp 20       # Examinar 20 entradas
gef➤ telescope 0x7ffe1234    # Examinar dirección específica
```

**Ventaja:** Muestra automáticamente hacia dónde apuntan los punteros.

#### `x` - Examinar memoria (comando estándar mejorado)

```bash
gef➤ x/20wx $rsp    # 20 words en hexadecimal
gef➤ x/10gx $rsp    # 10 quad-words (64-bit)
gef➤ x/s $rdi       # String en $rdi
```

---

### 5. **Control de Ejecución**

#### Breakpoints estratégicos

```bash
gef➤ break main                    # Breakpoint en main
gef➤ break *0x401234              # Breakpoint en dirección específica
gef➤ break *main+50               # Breakpoint en offset
```

#### Ejecución paso a paso

```bash
gef➤ run                          # Ejecutar programa
gef➤ continue                     # Continuar ejecución
gef➤ next                         # Siguiente línea (sin entrar en funciones)
gef➤ step                         # Siguiente instrucción (entra en funciones)
gef➤ finish                       # Terminar función actual
```

---

### 6. **Búsqueda de Gadgets ROP**

#### `rop` - Buscar gadgets ROP

```bash
gef➤ rop --search "pop rdi"       # Buscar gadget específico
gef➤ rop --search "ret"           # Buscar instrucciones ret
gef➤ rop --grep "pop.*ret"        # Buscar con regex
```

#### Buscar strings útiles

```bash
gef➤ search-pattern "/bin/sh"     # Buscar string "/bin/sh"
gef➤ search-pattern "system"      # Buscar función system
```

---

### 7. **Análisis de Funciones**

#### `info functions` - Listar funciones

```bash
gef➤ info functions              # Todas las funciones
gef➤ info functions system      # Buscar función específica
```

#### `disassemble` - Desensamblado mejorado

```bash
gef➤ disassemble main            # Desensamblar función main
gef➤ disassemble /r main         # Con bytes raw
```

---

### 8. **Trabajo con Librerías**

#### `vmmap` - Mapeo de memoria

```bash
gef➤ vmmap
```

**Muestra:** Todas las regiones de memoria cargadas, incluyendo librerías.

#### Encontrar direcciones base

```bash
gef➤ vmmap libc                  # Ver dónde está cargada libc
```

---

## 🔧 Flujo de Trabajo Típico para Buffer Overflow

### **Paso 1: Análisis inicial**

```bash
gef➤ checksec                    # Verificar protecciones
gef➤ info functions             # Ver funciones disponibles
gef➤ disassemble main           # Analizar función vulnerable
```

### **Paso 2: Encontrar vulnerabilidad**

```bash
gef➤ break *main+XX             # Breakpoint antes del input
gef➤ run
gef➤ pattern create 200         # Generar patrón
# Usar el patrón como input para causar crash
```

### **Paso 3: Calcular offset**

```bash
gef➤ pattern search $rip        # Encontrar offset exacto
```

### **Paso 4: Planificar exploit**

```bash
gef➤ vmmap                      # Ver layout de memoria
gef➤ search-pattern "/bin/sh"   # Buscar strings útiles
gef➤ rop --search "pop rdi"     # Buscar gadgets necesarios
```

### **Paso 5: Desarrollo del exploit**

```bash
gef➤ run < exploit.txt          # Probar exploit
gef➤ telescope $rsp             # Verificar contenido del stack
gef➤ context                    # Ver estado completo
```

---

## 🚀 Tips Avanzados

### **Scripting con GEF**

```bash
gef➤ pi print("Valor de RSP:", hex(gdb.parse_and_eval("$rsp")))
```

### **Logging y debugging**

```bash
gef➤ set logging on             # Guardar output en archivo
gef➤ set follow-fork-mode child # Seguir procesos hijo
```

### **Configuración útil**

```bash
gef➤ gef config context.nb_lines_code 10    # Más líneas de código
gef➤ gef config context.nb_lines_stack 20   # Más líneas de stack
```

---

## 📋 Comandos de Referencia Rápida

|Comando|Descripción|
|---|---|
|`checksec`|Verificar protecciones|
|`pattern create <size>`|Generar patrón cíclico|
|`pattern search <value>`|Encontrar offset|
|`telescope <addr>`|Examinar memoria|
|`vmmap`|Mapeo de memoria|
|`rop --search <pattern>`|Buscar gadgets ROP|
|`search-pattern <string>`|Buscar strings|
|`context`|Vista completa del estado|
|`disassemble <func>`|Desensamblado|
|`info functions`|Listar funciones|

---

**💡 Recuerda:** GEF muestra automáticamente el contexto después de cada comando de ejecución, lo que te ayuda a mantener una visión clara del estado del programa durante la explotación.