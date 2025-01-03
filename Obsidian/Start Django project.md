
**Creamos nuestra carpeta para el proyecto de django y en la raíz creamos el entorno:

```
py -3 venv .venv
```

**Abrimos la consola y ejecutamos:

```
.venv\Scripts\activate 
```

**Selección de interpretes


control + shift + P

seleccionar intérprete -> python 3.x.x ('.venv: venv')  .\.venv....

**Update de pip en el entorno virtual:

```
python -m pip --upgrade pip
```

**Instalación de Django

```
python -m pip install django
```

**Correr una aplicación pequeña en django

Donde tenemos activado el venv:

```
django-admin startproject web_project .
```

el último punto hace que se instale en la raiz de donde está la terminal.

manage.py el administrador de utilidades de django los comandos de administrador de correr ejecutando python mangae.py <command> [options]

subcarpeta llamada web_project que contiene:

__init__.py: un archivo vacío que le dice a python que esta carpeta es un paquete.
asgi.py: un access point para servir tu proyecto en la web se suele dejar como está pusto que ya está para producción.

settings.py: contiene las settings para el proyecto Django el cual también modificas a lo largo del desarrollo.

urls.py: las rutas que iremos modificando también a lo largo del desarrollo.

wsgi.py: se deja por default normalment al igual que usgi.

**Creamos una pequeña ddbb vacía:

```
python manage.py migrate
```

**Verificar todo ok hasta aqui:

el entorno virtual venv esta activado usando el comando:

```
python manage.py runserver
```

para detenerlo control + C .

**Crear Django app

con el entorno virtual activado correr el comando de administrador startapp en la carpeta donde se encuentra manage.py.

```
python mangae.py startapp hello
```

esto creara una carpeta con el nombre hello.

**Crear una vista simple del home de la página en hello/views:

```
from django.http import HttpResponse

def home(request):
    return HttpResponse("Hello, Django!")

```


**Creamos la file para las url en hello/urls.py con el siguiente código:

```
from django.urls import path
from hello import views

urlpatterns = [
    path("", views.home, name="home"),
]

```

**Incluimos este fichero de rutas al que tiene nuestro urls.py que ya venía al crear el web_project:

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("", include("hello.urls")),
    path('admin/', admin.site.urls)
]
```


**Cear un perfil de debuggeo para la aplicación y que corra el server por defecto:

evitaremos tener que ejecutar python manage.py runserver con la siguiente config.

accedemos mediante el f5 al debugger y selecionamos cree un archivo launch.json:

```
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python Debugger: Django",
      "type": "debugpy",
      "request": "launch",
      "program": "${workspaceFolder}\\manage.py",
      "args": ["runserver"],
      "django": true,
      "justMyCode": true
    }
  ]
}

```

**Crear template en django

en la carpeta de web_project/settings.py localizar installed_apps y añadimos la entrada 'hello',

ahora en la carpeta de hello creamos la carpeta templates y dentro la carpeta hello, asi es la manera estandarizada de realizarlo en django y ahora ya dentro creamos el hello_there.html,
ahora en views.py importtamos from django.shortcuts import render
y hacemos que la funcion de hello_there lo utilice

```
def hello_there(request, name):
    print(request.build_absolute_uri()) #optional
    return render(
        request,
        'hello/hello_there.html',
        {
            'name': name,
            'date': datetime.now()
        }
    )

```


hacer que reciba un parametro modificamos la funcion para que se pueda enviar un date:

ejemplo de url enviada:

http://127.0.0.1:8000/hello/juan?date=2024-01-01

```
def hello_there(request, name):

    # Obtén la fecha del parámetro de consulta

    date_param = request.GET.get('date', None)

  

    if date_param:

        try:

            # Intenta convertir la fecha proporcionada en un objeto datetime

            date = datetime.strptime(date_param, "%Y-%m-%d")

        except ValueError:

            return HttpResponse("Formato de fecha inválido. Use AAAA-MM-DD.", status=400)

    else:

        # Usa la fecha actual si no se proporciona una

        date = datetime.now()

  

    return render(

        request,

        'hello/hello_there.html',

        {

            'name': name,

            'date': date,

        }

    )
```

servir ficheros estaticos css:

web_project/urls/py:

from django.contrib.staticfiles.urls import staticfiles_urlpatterns

y ahora en la carpeta hello creamos la carpeta static dentro la carpeta hello y dentro el site.css

mirar en settings.py que la línea de static este asi:;

STATIC_URL = '/static/'

**crear un template básico de página creando un snipper

puesto que muchas páginas van a compartir varios elementos como el navegador por ejemplo o el title esta bien crear snippers para ello. 

en template/hello creamos el archivo layout.html con el siguiente código:

```
<!DOCTYPE html>

<html>

<head>

    <meta charset="utf-8"/>

    <title>{% block title %}{% endblock %}</title>

    {% load static %}

    <link rel="stylesheet" type="text/css" href="{% static 'hello/site.css' %}"/>

</head>

  

<body>

<div class="navbar">

    <a href="{% url 'home' %}" class="navbar-brand">Home</a>

    <a href="{% url 'about' %}" class="navbar-item">About</a>

    <a href="{% url 'contact' %}" class="navbar-item">Contact</a>

</div>

  

<div class="body-content">

    {% block content %}

    {% endblock %}

  

    <hr/>

    <footer>

        <p>&copy; 2018</p>

    </footer>

</div>

</body>

</html>
```


**Crear el snipper

en vcode en archivo vamos a preferencias y elegimos fragmento de código:

html e introducimos el siguiente código:

```
{

    "Django Tutorial: template extending layout.html": {

    "prefix": "djextlayout",

    "body": [

        "{% extends \"hello/layout.html\" %}",

        "{% block title %}",

        "$0",

        "{% endblock %}",

        "{% block content %}",

        "{% endblock %}"

    ],

  

    "description": "Boilerplate template that extends layout.html"

},

  

}
```

ahora creamos nuestro about home contact para ello las urls se tienen que ver así:

```
from django.urls import path

from hello import views

  

urlpatterns= [

    path("", views.home, name = "home"),

    path("hello/<name>", views.hello_there, name = "hello_there"),

    path("home/", views.home, name="home"),

    path("about/", views.about, name="about"),

    path("contact/", views.contact, name="contact"),

]
```

ahora creamos cada archivo .html usando el snipper y configuramos cada uno siguiendo este ejemplo:

```
{% extends "hello/layout.html" %}

{% block title %}

About

{% endblock %}

{% block content %}

<p>Prueba del bloque de about</p>

{% endblock %}
```

y en las views renderizamos cada página:

```
def home(request):

    return render(request, "hello/home.html")

def contact(request):

    return render(request, "hello/contact.html")

def about(request):

    return render (request, "hello/about.html")
```

y nuestro css en site.css se verá tal que así:

```
.message {

    font-weight: 600;

    color: blue;

}

  

.navbar {

    background-color: lightslategray;

    font-size: 1em;

    font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;

    color: white;

    padding: 8px 5px 8px 5px;

}

  

.navbar a {

    text-decoration: none;

    color: inherit;

}

  

.navbar-brand {

    font-size: 1.2em;

    font-weight: 600;

}

  

.navbar-item {

    font-variant: small-caps;

    margin-left: 30px;

}

  

.body-content {

    padding: 5px;

    font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;

}
```

**Models y base de datos en django 

solo modificar models migrations lo maneja djandgo por detrás aplicar cambios en el modelo y migrarlo:

```
python manage.py makemigrations
python manage.py migrate

```

en models.py añadir :

### [Define models](https://code.visualstudio.com/docs/python/tutorial-django#_define-models)

A Django model is again a Python class derived from `django.db.model.Models`, which you place in the app's `models.py` file. In the database, each model is automatically given a unique ID field named `id`. All other fields are defined as properties of the class using types from `django.db.models` such as `CharField` (limited text), `TextField` (unlimited text), `EmailField`, `URLField`, `IntegerField`, `DecimalField`, `BooleanField`. `DateTimeField`, `ForeignKey`, and `ManyToMany`, among others. (See the [Model field reference](https://docs.djangoproject.com/en/3.1/ref/models/fields/) in the Django documentation for details.)

Each field takes some attributes, like `max_length`. The `blank=True` attribute means the field is optional; `null=true` means that a value is optional. There is also a `choices` attribute that limits values to values in an array of data value/display value tuples.

For example, add the following class in `models.py` to define a data model that represents dated entries in a simple message log:


```
from django.db import models
from django.utils import timezone

class LogMessage(models.Model):
    message = models.CharField(max_length=300)
    log_date = models.DateTimeField("date logged")

    def __str__(self):
        """Returns a string representation of a message."""
        date = timezone.localtime(self.log_date)
        return f"'{self.message}' logged on {date.strftime('%A, %d %B, %Y at %X')}"

```

**Crear un superuser

```
python manage.py createsuperuser --username=<username> --email=<email>
```

