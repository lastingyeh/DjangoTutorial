###  Tell Django is installed and which version by running at this env.

*> python -m django --version*

------------------------------------------------------------------------------------
### Creating a project

*> django-admin startproject mysite*

------------------------------------------------------------------------------------
### Let’s look at what startproject created:

[manage.py](https://docs.djangoproject.com/en/2.0/ref/django-admin/)

[mysite/settings.py](https://docs.djangoproject.com/en/2.0/topics/settings/)

[mysite/urls.py](https://docs.djangoproject.com/en/2.0/topics/http/urls/)

[mysite/wsgi.py](https://docs.djangoproject.com/en/2.0/howto/deployment/wsgi/)

------------------------------------------------------------------------------------
### The development server

*> python manage.py runserver*

    Note : Ignore the warning about unapplied database migrations for now; we’ll deal with the database shortly.

------------------------------------------------------------------------------------
### visit http://127.0.0.1:8000/ with your Web browser

*Note : Changing the port

*> python manage.py runserver 8080*

*Note : to listen on all available public IPs 

*> python manage.py runserver 0:8000*

    `*0 is a shortcut for 0.0.0.0 : https://docs.djangoproject.com/en/2.0/ref/django-admin/#django-admin-runserver`

------------------------------------------------------------------------------------
### Creating the Polls app

*> python manage.py startapp polls*

------------------------------------------------------------------------------------
### Write your first view

`polls/views.py`

    from django.http import HttpResponse

    def index(request):
        return HttpResponse("Hello, world. You're at the polls index.")

`polls/urls.py` (create urls.py)

    from django.urls import path

    from . import views

    urlpatterns = [
        path('', views.index, name='index'),
    ]

`mysite/urls.py`

    from django.urls import include, path
    from django.contrib import admin

    urlpatterns = [
        path('polls/', include('polls.urls')),
        path('admin/', admin.site.urls),
    ]

[path refs](https://docs.djangoproject.com/en/2.0/ref/urls/#django.urls.path)