[Django project](https://www.djangoproject.com/) \
Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel. It’s free and open source.

[Models and the admin site](https://docs.djangoproject.com/en/4.2/intro/tutorial02/)

* Database setup \
Django uses SQLite by default, it's useful for small projects. "When starting your first real project, however, you may want to use a more scalable database like PostgreSQL, to avoid database-switching headaches down the road."

- project/settings.py \
**INSTALLED_APPS**: setting at the top of the file. That holds the names of all Django applications that are activated in this Django instance. \

By default, **INSTALLED_APPS** contains the following apps, all of which come with Django: \
**django.contrib.admin**: The admin site. You’ll use it shortly.
**django.contrib.auth**: An authentication system.
**django.contrib.contenttypes**: A framework for content types.
**django.contrib.sessions**: A session framework.
**django.contrib.messages**: A messaging framework.
**django.contrib.staticfiles**: A framework for managing static files. 

* Creating models \
"A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing."

```
# models.py

from django.db import models

"""
A Question has a question and a publication date. A Choice has two fields: the text of the choice and a vote tally. Each Choice is associated with a Question.
"""

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

```
"Django apps are “pluggable”: You can use an app in multiple projects, and you can distribute apps, because they don’t have to be tied to a given Django installation."

```
$ python manage.py makemigrations polls

"""
By running makemigrations, you’re telling Django that you’ve made some changes to your models (in this case, you’ve made new ones) and that you’d like the changes to be stored as a migration.
"""
```
Migrations let you change your models over time, as you develop your project, without the need to delete your database or tables and make new ones - it specializes in upgrading your database live, without losing data.

three-step guide to making model changes:
* Change your models (in models.py).
* Run python manage.py makemigrations to create migrations for those changes
* Run python manage.py migrate to apply those changes to the database.

* Playing with the API \
```
$ python manage.py shell
"""
Use this instead of simply typing “python”, because manage.py sets the DJANGO_SETTINGS_MODULE environment variable, which gives Django the Python import path to your mysite/settings.py file.
"""
```
* Introducing the Django Admin 
Generating admin sites for your staff or clients to add, change, and delete content is tedious work that doesn’t require much creativity. For that reason, Django entirely automates creation of admin interfaces for models.

The admin isn’t intended to be used by site visitors. It’s for site managers.

- Creating an admin user 
```
$ python manage.py createsuperuser
Username: admin
Email address: admin@example.com
Password: **********
Password (again): *********
Superuser created successfully.
$ python manage.py runserver
```
- Make the app modifiable in the admin
```
# admin.py
from django.contrib import admin
from .models import Question
admin.site.register(Question)
```
[Views and templates](https://docs.djangoproject.com/en/4.2/intro/tutorial03/)

A view is a “type” of web page in your Django application that generally serves a specific function and has a specific template. 

In Django, web pages and other content are delivered by views. Each view is represented by a Python function (or method, in the case of class-based views). Django will choose a view by examining the URL that’s requested (to be precise, the part of the URL after the domain name). A URL pattern is the general form of a URL - for example: /newsarchive/<year>/<month>/. To get from a URL to a view, Django uses what are known as ‘URLconfs’. A URLconf maps URL patterns to views.

* Write views that actually do something \
Each view is responsible for doing one of two things: returning an HttpResponse object containing the content for the requested page, or raising an exception such as Http404. The rest is up to you.

```
# view.py
from django.http import HttpResponse
from .models import Question
def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
# Leave the rest of the views (detail, results, vote) unchanged
```
Your project’s TEMPLATES setting describes how Django will load and render templates. The default settings file configures a DjangoTemplates backend whose APP_DIRS option is set to True. By convention DjangoTemplates looks for a “templates” subdirectory in each of the INSTALLED_APPS.

Within the templates directory you have just created, create another directory called polls, and within that create a file called index.html. In other words, your template should be at polls/templates/polls/index.html. Because of how the app_directories template loader works as described above, you can refer to this template within Django as polls/index.html.

```
# views.py
from django.http import HttpResponse
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))

"""
That code loads the template called polls/index.html and passes it a context. The context is a dictionary mapping template variable names to Python objects.
"""
```

```
# views.py
from django.shortcuts import render
from .models import Question
def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)

"""
It’s a very common idiom to load a template, fill a context and return an HttpResponse object with the result of the rendered template. Django provides a shortcut. Here’s the full index() view

Note that once we’ve done this in all these views, we no longer need to import loader and HttpResponse (you’ll want to keep HttpResponse if you still have the stub methods for detail, results, and vote).

The render() function takes the request object as its first argument, a template name as its second argument and a dictionary as its optional third argument. It returns an HttpResponse object of the given template rendered with the given context.
"""
```
