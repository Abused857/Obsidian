
# 🐧 Comandos Linux Útiles

## 📁 Gestión de Archivos y Directorios

---
### `df -h`

🔍 **Palabras clave**: espacio disco, particiones, uso disco, tamaño  
🛠️ **Qué hace**: Muestra el uso del disco por sistema de archivos en formato legible (GB/MB).  
📌 **Ejemplo usado**:

```bash
df -h
```

📌 **Explicación columnas**:

- `Filesystem`: nombre del dispositivo o punto de montaje
    
- `Size`: tamaño total
    
- `Used`: espacio usado
    
- `Avail`: espacio disponible
    
- `Use%`: porcentaje en uso
    
- `Mounted on`: punto de montaje
### `ls -la`

🔍 **Palabras clave**: listar, archivos ocultos, permisos  
🛠️ **Qué hace**: Lista el contenido de un directorio en formato largo, incluyendo archivos ocultos (los que comienzan con `.`) y permisos.  
📌 **Ejemplo usado**:
```bash
ls -la
```

### `du -sh *`

🔍 **Palabras clave**: tamaño, directorio, disco  
🛠️ **Qué hace**: Muestra el tamaño total de cada subdirectorio/archivo en la carpeta actual, en formato legible (KB, MB, GB).  
📌 **Ejemplo usado**:

```bash
du -sh *
```

### `du -sh 2504*`

🔍 **Palabras clave**: tamaño, wildcard, filtro  
🛠️ **Qué hace**: Muestra el tamaño de todos los directorios que comienzan con `2504`. Útil para filtrar por nombre.  
📌 **Ejemplo usado**:


```bash
du -sh 2504*
```


### `rm -rf <directorio>`

🔍 **Palabras clave**: eliminar, borrar carpeta, forzar  
🛠️ **Qué hace**: Elimina directorios y archivos recursivamente sin pedir confirmación.  
📌 **Precaución**: **Este comando no pregunta. Asegúrate de estar en la ruta correcta.**  
📌 **Ejemplo usado** (masivo):

```bash
rm -rf 250400005 250400006 250400007 ... 250400224
```

### `sudo rm -f <archivo>`

🔍 **Palabras clave**: borrar archivo, forzar, permisos  
🛠️ **Qué hace**: Elimina un archivo con privilegios elevados (`sudo`), sin confirmación.  
📌 **Nota**: Si el objetivo es un directorio, este comando fallará. Usa `-r` si es necesario.  
📌 **Ejemplo**:

```bash
sudo rm -f 250500001.pdf
```

### `cd <ruta>`

🔍 **Palabras clave**: cambiar directorio, navegar  
🛠️ **Qué hace**: Cambia al directorio especificado.  
📌 **Ejemplo usado**:

```bash
cd /var/www/html/storage/RecordDocuments
```


### `tail -f`

🔍 **Palabras clave**: logs, monitoreo en vivo, ver últimos registros  
🛠️ **Qué hace**: Muestra las últimas líneas de un archivo y sigue mostrando en tiempo real lo que se agrega. Ideal para observar archivos de log mientras un proceso está en ejecución.  
📌 **Ejemplo típico**:


```bash
tail -f logs/error.log
```
📌 **Explicación**:

- `tail`: muestra las últimas líneas de un archivo (por defecto, las últimas 10).
    
- `-f`: **"follow"** — mantiene la vista abierta y actualiza cuando se agregan nuevas líneas.
    

📌 **Uso habitual**:

- Monitorear errores de Apache, PHP o cualquier otro servicio en tiempo real.
    
- Diagnóstico de procesos o cron jobs.
    

📌 **Variaciones útiles**:


```bash
tail -n 50 logs/error.log     # Muestra las últimas 50 líneas
tail -f /var/log/syslog       # Sigue el log del sistema

```

📌 **Salida esperada**:  
Verás las últimas líneas del archivo e irán apareciendo nuevas automáticamente al final, como si fuera un "streaming" de texto.

## 🧠 Monitorización del sistema

---

### `htop`

🔍 **Palabras clave**: procesos, RAM, CPU, monitor, system resources  
🛠️ **Qué hace**: Monitor interactivo en terminal que muestra procesos del sistema, uso de CPU, memoria, y más.  
📌 **Usos comunes**:

- Ver qué procesos están consumiendo recursos
    
- Matar procesos desde la interfaz
    

📌 **Instalación (si no está):**
```bash
sudo apt install htop
```


```bash
htop
```



### `chmod -R 777 tmp`

🔍 **Palabras clave**: permisos, escritura, recursivo  
🛠️ **Qué hace**: Da permisos de lectura, escritura y ejecución a todos para el directorio `tmp` y su contenido.  
📌 **Ejemplo**:

```bash
chmod -R 777 tmp
```

⚠️ **Cuidado** con permisos 777 en entornos de producción. Solo para debugging/local.

### `sed -i 's/\r$//' <archivo>`

🔍 **Palabras clave**: quitar retornos, windows, formato dosunix  
🛠️ **Qué hace**: Quita los retornos de línea de Windows (`\r`) de un archivo.  
📌 **Ejemplo usado**:

```bash
`sed -i 's/\r$//' bin/cake`
```
📌 Útil si copiaste un script desde Windows o con formato incorrecto.



### `ps aux | grep <texto>`

🔍 **Palabras clave**: buscar procesos, listado, grep  
🛠️ **Qué hace**: Lista todos los procesos del sistema y filtra los que coinciden con el texto.  
📌 **Ejemplos usados**:

```bash
ps aux | grep expedientes 
```

```bash
ps aux | grep 'cake.php Queues.jobs daemon'
```

```bash
ps aux | grep php
```

### `kill -9 <PID>`

🔍 **Palabras clave**: matar proceso, detener  
🛠️ **Qué hace**: Fuerza la terminación inmediata del proceso con PID específico.  
📌 **Ejemplo**:


```bash
kill -9 12345
```

⚠️ Asegúrate de que el PID sea correcto y el proceso no sea crítico del sistema.


### `sudo pkill -f 'cake.php Queues.jobs daemon'`

🔍 **Palabras clave**: detener daemon, background process, cakephp  
🛠️ **Qué hace**: Mata procesos cuyo comando completo (línea completa) coincide con el patrón dado.  
📌 **Ejemplo**:

```bash
sudo pkill -f 'cake.php Queues.jobs daemon'
```
