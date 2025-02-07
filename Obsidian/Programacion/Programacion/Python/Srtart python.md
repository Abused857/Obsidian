**Installar pyton en windows

descargamos python de su página principal 

-> https://www.python.org/downloads/

**Creamos el entorno virtual en la carpeta 
para ello presionamos control + shift + P y empezamos a escribir 'Python: Create Environment'

seleccionamos Venv y seleccionamos el interprete que queramos.

**Creamos el hello.py

para ello en la raiz del proyecto creamos nuevo file y lo llamamos hello.py

Escribimos una variable con un string cualquiera y luego usamos print(variable)

```
msg = "Roll a dice!"

print(msg)
```

ahora podemos ejecutarlo desde arriba a la derecha dando al play.

también podemos seleccionar las líneas que queramos dar click derecho y seleccionar ejecutar en la terminal.

**Debuggear en python

seleccionamos la extensión de python debugger e instalamos en visual code.

Para inicializar el debugger se presiona f5 seleccionamos python file.

**Importar y usar una librería

Windows

```
py -m pip install numpy
```

Importar numpy y poner un número random

```
import numpy as np

print(np.random.randit(1,9))
```


**Activar el enviroment de python crear el requirements

Desactivar la restricción de ejecutar scripts en windows powershell abrelo como administrador y pon:

```
Set-ExecutionPolicy Unrestricted
```

en la terminal desde la raíz 

```
.\.venv\Scripts\activate
```

ver como se creo el venv si con el . o sin él.

y ahora actualizamos el requirements

```
pip freeze > requirements.txt
```

Si ahora queremos instalar estas dependencias en otro entorno simplemente ejecutamos.

```
pip install -r requirements.txt
```

[[Python]]