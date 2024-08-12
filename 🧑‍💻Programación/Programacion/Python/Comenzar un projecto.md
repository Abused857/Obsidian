
### .markdown

# Crear la Estructura del Proyecto Python: 

`primer_proyecto`  

Este documento describe los pasos para configurar un proyecto básico en Python llamado `primer_proyecto`.  
## 1. Crear el Directorio del Proyecto  

**Abrir una terminal** y navegar al directorio deseado:    

```
cd ruta/del/directorio
```

2. **Crear el directorio del proyecto**:
    
    `mkdir primer_proyecto cd primer_proyecto`
    

## 2. Inicializar el Entorno Virtual

1. **Crear un entorno virtual** para gestionar dependencias:
    
    `python -m venv venv`
    
2. **Activar el entorno virtual**:
    
    - En **Windows**:
        
        
        `venv\Scripts\activate`
        
    - En **macOS/Linux**:
        
        `source venv/bin/activate`
        

## 3. Crear Estructura de Directorios

Crear la siguiente estructura de directorios y archivos:

```
primer_proyecto/
├── venv/                    # Entorno virtual
├── primer_proyecto/         # Paquete principal
│   ├── __init__.py          # Inicializador del paquete
│   └── main.py              # Archivo principal del código
├── tests/                   # Directorio para pruebas
│   ├── __init__.py          # Inicializador del paquete de pruebas
│   └── test_main.py         # Archivo para pruebas de main.py
├── .gitignore                # Ignorar archivos en Git
├── requirements.txt         # Dependencias del proyecto
├── README.md                # Documentación del proyecto
└── setup.py                 # Configuración del paquete

```

## 4. Crear Archivos Iniciales

1. **`__init__.py`** en `primer_proyecto/` y `tests/`:
    
    - Estos archivos pueden estar vacíos, pero son necesarios para que Python trate estos directorios como paquetes.
2. **`main.py`** en `primer_proyecto/`:
3. 
```
    def main():
    print("Hola, mundo!")

if __name__ == "__main__":
    main()

```
    
3. **`test_main.py`** en `tests/`:
    
```
   import unittest
from primer_proyecto.main import main

class TestMain(unittest.TestCase):
    def test_main(self):
        with self.assertLogs(level='INFO'):
            main()

if __name__ == "__main__":
    unittest.main()

```
    
4. **`.gitignore`**:
    
    `venv/ *.pyc __pycache__/`
    
5. **`requirements.txt`**:
    
    - Deja este archivo vacío por ahora. Añade las dependencias conforme las necesites.
6. **`README.md`**:
    
    
    # Primer Proyecto  Este es el primer proyecto en Python.`
    
7. **`setup.py`**:
    
```
    from setuptools import setup, find_packages

setup(
    name='primer_proyecto',
    version='0.1',
    packages=find_packages(),
    install_requires=[],
    entry_points={
        'console_scripts': [
            'primer_proyecto=primer_proyecto.main:main',
        ],
    },
)

```

## 5. Instalar Dependencias

Si necesitas instalar dependencias adicionales:

1. Instala el paquete:
    
    `pip install <paquete>`
    
2. **Actualizar `requirements.txt`**:
    
    `pip freeze > requirements.txt`
    

## 6. Inicializar Git (Opcional)

1. **Inicializa un repositorio Git**:
    
    `git init`
    
2. **Añade todos los archivos y haz un commit**:
    
    `git add . git commit -m "Primer commit"`


[[Programacion/Python/Python]]