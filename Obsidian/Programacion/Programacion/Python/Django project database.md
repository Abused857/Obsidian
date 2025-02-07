
Tener instaldo python y añadir a las varibles de entorno la ruta a python313 y python/Scripts en users app local common python

una vez hecho eso, creamos nuestra carpeta para el proyecto usando el comando

```
 mkdir y el nombre que le queramos dar.
```

ahora instalamos django:

```
python -m pip install django
```

Y ahjora ejecutamos el siguiente comando para ejecutar un nuevo proyecto de django que lo iniciara desde donde estgamos ern la carpeta que hemos creado:

```
django-admin startproject mysite djangotutorial
```

esto creará mysite denbtro de la carpeta djangotutorial

esto generará los siguientes ficheros en mysite:


Let’s look at what [`startproject`](https://docs.djangoproject.com/en/5.1/ref/django-admin/#django-admin-startproject) created:

djangotutorial/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py

These files are:

- `manage.py`: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about `manage.py` in [django-admin and manage.py](https://docs.djangoproject.com/en/5.1/ref/django-admin/).
    
- `mysite/`: A directory that is the actual Python package for your project. Its name is the Python package name you’ll need to use to import anything inside it (e.g. `mysite.urls`).
    
- `mysite/__init__.py`: An empty file that tells Python that this directory should be considered a Python package. If you’re a Python beginner, read [more about packages](https://docs.python.org/3/tutorial/modules.html#tut-packages "(in Python v3.13)") in the official Python docs.
    
- `mysite/settings.py`: Settings/configuration for this Django project. [Django settings](https://docs.djangoproject.com/en/5.1/topics/settings/) will tell you all about how settings work.
    
- `mysite/urls.py`: The URL declarations for this Django project; a “table of contents” of your Django-powered site. You can read more about URLs in [URL dispatcher](https://docs.djangoproject.com/en/5.1/topics/http/urls/).
    
- `mysite/asgi.py`: An entry-point for ASGI-compatible web servers to serve your project. See [How to deploy with ASGI](https://docs.djangoproject.com/en/5.1/howto/deployment/asgi/) for more details.
    
- `mysite/wsgi.py`: An entry-point for WSGI-compatible web servers to serve your project. See [How to deploy with WSGI](https://docs.djangoproject.com/en/5.1/howto/deployment/wsgi/) for more details.



correr el servidor:

```
py manage.py runserver
```


crear nuestra primera app ejecutar este comando a la altura del manage.py:

```
py manage.py startapp polls
```

That’ll create a directory `polls`, which is laid out like this:

polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py

This directory structure will house the poll application.

## Write your first view[¶](https://docs.djangoproject.com/en/5.1/intro/tutorial01/#write-your-first-view "Link to this heading")

Let’s write the first view. Open the file `polls/views.py` and put the following Python code in it:

```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

This is the most basic view possible in Django. To access it in a browser, we need to map it to a URL - and for this we need to define a URL configuration, or “URLconf” for short. These URL configurations are defined inside each Django app, and they are Python files named `urls.py`.

To define a URLconf for the `polls` app, create a file `polls/urls.py` with the following content:

```
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]

Your app directory should now look like:

polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py

The next step is to configure the global URLconf in the `mysite` project to include the URLconf defined in `polls.urls`. To do this, add an import for `django.urls.include` in `mysite/urls.py` and insert an [`include()`](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.include "django.urls.include") in the `urlpatterns` list, so you have:

`mysite/urls.py`[¶](https://docs.djangoproject.com/en/5.1/intro/tutorial01/#id3 "Link to this code")

from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]

The [`path()`](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.path "django.urls.path") function expects at least two arguments: `route` and `view`. The [`include()`](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.include "django.urls.include") function allows referencing other URLconfs. Whenever Django encounters [`include()`](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.include "django.urls.include"), it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing.

The idea behind [`include()`](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.include "django.urls.include") is to make it easy to plug-and-play URLs. Since polls are in their own URLconf (`polls/urls.py`), they can be placed under “/polls/”, or under “/fun_polls/”, or under “/content/polls/”, or any other path root, and the app will still work.

When to use [`include()`](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.include "django.urls.include")

You should always use `include()` when you include other URL patterns. The only exception is `admin.site.urls`, which is a pre-built URLconf provided by Django for the default admin site.

You have now wired an `index` view into the URLconf. Verify it’s working with the following command:

/ 

...\> py manage.py runserver

Go to [http://localhost:8000/polls/](http://localhost:8000/polls/) in your browser, and you should see the text “_Hello, world. You’re at the polls index._”, which you defined in the `index` view.

Page not found?

If you get an error page here, check that you’re going to [http://localhost:8000/polls/](http://localhost:8000/polls/) and not [http://localhost:8000/](http://localhost:8000/).

When you’re comfortable with the basic request and response flow, read [part 2 of this tutorial](https://docs.djangoproject.com/en/5.1/intro/tutorial02/) to start working with the database.


In our poll app, we’ll create two models: `Question` and `Choice`. A `Question` has a question and a publication date. A `Choice` has two fields: the text of the choice and a vote tally. Each `Choice` is associated with a `Question`.

These concepts are represented by Python classes. Edit the `polls/models.py` file so it looks like this:


```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Here, each model is represented by a class that subclasses [`django.db.models.Model`](https://docs.djangoproject.com/en/5.1/ref/models/instances/#django.db.models.Model "django.db.models.Model"). Each model has a number of class variables, each of which represents a database field in the model.

Each field is represented by an instance of a [`Field`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.Field "django.db.models.Field") class – e.g., [`CharField`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.CharField "django.db.models.CharField") for character fields and [`DateTimeField`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.DateTimeField "django.db.models.DateTimeField") for datetimes. This tells Django what type of data each field holds.

The name of each [`Field`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.Field "django.db.models.Field") instance (e.g. `question_text` or `pub_date`) is the field’s name, in machine-friendly format. You’ll use this value in your Python code, and your database will use it as the column name.

You can use an optional first positional argument to a [`Field`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.Field "django.db.models.Field") to designate a human-readable name. That’s used in a couple of introspective parts of Django, and it doubles as documentation. If this field isn’t provided, Django will use the machine-readable name. In this example, we’ve only defined a human-readable name for `Question.pub_date`. For all other fields in this model, the field’s machine-readable name will suffice as its human-readable name.

Some [`Field`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.Field "django.db.models.Field") classes have required arguments. [`CharField`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.CharField "django.db.models.CharField"), for example, requires that you give it a [`max_length`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.CharField.max_length "django.db.models.CharField.max_length"). That’s used not only in the database schema, but in validation, as we’ll soon see.

A [`Field`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.Field "django.db.models.Field") can also have various optional arguments; in this case, we’ve set the [`default`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.Field.default "django.db.models.Field.default") value of `votes` to 0.

Finally, note a relationship is defined, using [`ForeignKey`](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.ForeignKey "django.db.models.ForeignKey"). That tells Django each `Choice` is related to a single `Question`. Django supports all the common database relationships: many-to-one, many-to-many, and one-to-one.

## Activating models[¶](https://docs.djangoproject.com/en/5.1/intro/tutorial02/#activating-models "Link to this heading")

That small bit of model code gives Django a lot of information. With it, Django is able to:

- Create a database schema (`CREATE TABLE` statements) for this app.
    
- Create a Python database-access API for accessing `Question` and `Choice` objects.
    

But first we need to tell our project that the `polls` app is installed.

o include the app in our project, we need to add a reference to its configuration class in the [`INSTALLED_APPS`](https://docs.djangoproject.com/en/5.1/ref/settings/#std-setting-INSTALLED_APPS) setting. The `PollsConfig` class is in the `polls/apps.py` file, so its dotted path is `'polls.apps.PollsConfig'`. Edit the `mysite/settings.py` file and add that dotted path to the [`INSTALLED_APPS`](https://docs.djangoproject.com/en/5.1/ref/settings/#std-setting-INSTALLED_APPS) setting. It’ll look like this:

```
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```


Now Django knows to include the `polls` app. Let’s run another command:

```
python manage.py makemigrations polls
```

You should see something similar to the following:

Migrations for 'polls':
  polls/migrations/0001_initial.py
    + Create model Question
    + Create model Choice



By running `makemigrations`, you’re telling Django that you’ve made some changes to your models (in this case, you’ve made new ones) and that you’d like the changes to be stored as a _migration_.

Migrations are how Django stores changes to your models (and thus your database schema) - they’re files on disk. You can read the migration for your new model if you like; it’s the file `polls/migrations/0001_initial.py`. Don’t worry, you’re not expected to read them every time Django makes one, but they’re designed to be human-editable in case you want to manually tweak how Django changes things.

There’s a command that will run the migrations for you and manage your database schema automatically - that’s called [`migrate`](https://docs.djangoproject.com/en/5.1/ref/django-admin/#django-admin-migrate), and we’ll come to it in a moment - but first, let’s see what SQL that migration would run. The [`sqlmigrate`](https://docs.djangoproject.com/en/5.1/ref/django-admin/#django-admin-sqlmigrate) command takes migration names and returns their SQL:

```
py manage.py sqlmigrate polls 0001
```

The [`migrate`](https://docs.djangoproject.com/en/5.1/ref/django-admin/#django-admin-migrate) command takes all the migrations that haven’t been applied (Django tracks which ones are applied using a special table in your database called `django_migrations`) and runs them against your database - essentially, synchronizing the changes you made to your models with the schema in the database.

Migrations are very powerful and let you change your models over time, as you develop your project, without the need to delete your database or tables and make new ones - it specializes in upgrading your database live, without losing data. We’ll cover them in more depth in a later part of the tutorial, but for now, remember the three-step guide to making model changes:

- Change your models (in `models.py`).
    
- Run [`python manage.py makemigrations`](https://docs.djangoproject.com/en/5.1/ref/django-admin/#django-admin-makemigrations) to create migrations for those changes
    
- Run [`python manage.py migrate`](https://docs.djangoproject.com/en/5.1/ref/django-admin/#django-admin-migrate) to apply those changes to the database.
    

The reason that there are separate commands to make and apply migrations is because you’ll commit migrations to your version control system and ship them with your app; they not only make your development easier, they’re also usable by other developers and in production.

Read the [django-admin documentation](https://docs.djangoproject.com/en/5.1/ref/django-admin/) for full information on what the `manage.py` utility can do.

## Playing with the API[¶](https://docs.djangoproject.com/en/5.1/intro/tutorial02/#playing-with-the-api "Link to this heading")

Now, let’s hop into the interactive Python shell and play around with the free API Django gives you. To invoke the Python shell, use this command:

```
py manage.py shell
```


q = Questrion(question_text = asdfregsadefrg)

[[Python]]