### Design your model

`mysite/news/models.py`

    from django.db import models

    class Reporter(models.Model):
        full_name = models.CharField(max_length=70)

        def __str__(self):
            return self.full_name

    class Article(models.Model):
        pub_date = models.DateField()
        headline = models.CharField(max_length=200)
        content = models.TextField()
        reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)

        def __str__(self):
            return self.headline

------------------------------------------------------------------------------------
### Install it (create the database tables automatically)

__>>> python manage.py migrate__

`*migrations: https://docs.djangoproject.com/en/2.0/topics/migrations/`

------------------------------------------------------------------------------------
### Enjoy the free API

__>>> python manage.py shell__

##### Import the models we created from our "news" app
__>>> from news.models import Reporter, Article__

##### No reporters are in the system yet.
__>>> Reporter.objects.all()__
<QuerySet []>

##### Create a new Reporter.
__>>> r = Reporter(full_name='John Smith')__

##### Save the object into the database. You have to call save() explicitly.
__>>> r.save()__

##### Now it has an ID.
__>>> r.id__
1

##### Now the new reporter is in the database.
__>>> Reporter.objects.all()__
<QuerySet [<Reporter: John Smith>]>

##### Fields are represented as attributes on the Python object.
__>>> r.full_name__
'John Smith'

##### Django provides a rich database lookup API.
__>>> Reporter.objects.get(id=1)__
<Reporter: John Smith>

__>>> Reporter.objects.get(full_name__startswith='John')__
<Reporter: John Smith>

__>>> Reporter.objects.get(full_name__contains='mith')__
<Reporter: John Smith>

__>>> Reporter.objects.get(id=2)__
Traceback (most recent call last):
    ...
DoesNotExist: Reporter matching query does not exist.

##### Create an article.
__>>> from datetime import date__

__>>> a = Article(pub_date=date.today(), headline='Django is cool',__
__...     content='Yeah.', reporter=r)__

__>>> a.save()__

##### Now the article is in the database.
__>>> Article.objects.all()__
<QuerySet [<Article: Django is cool>]>

##### Article objects get API access to related Reporter objects.
__>>> r = a.reporter__

__>>> r.full_name__
'John Smith'

##### And vice versa: Reporter objects get API access to Article objects.
__>>> r.article_set.all()__
<QuerySet [<Article: Django is cool>]>

##### The API follows relationships as far as you need, performing efficient
##### JOINs for you behind the scenes.
##### This finds all articles by a reporter whose name starts with "John".

__>>> Article.objects.filter(reporter__full_name__startswith='John')__

<QuerySet [<Article: Django is cool>]>

##### Change an object by altering its attributes and calling save().
__>>> r.full_name = 'Billy Goat'__

__>>> r.save()__

##### Delete an object with delete().
__>>> r.delete()__

------------------------------------------------------------------------------------
### registering your model in the admin site

`mysite/news/admin.py`

    from django.contrib import admin
    from . import models

    admin.site.register(models.Article)

------------------------------------------------------------------------------------
### Design your URLs

`mysite/news/urls.py`

    from django.urls import path
    from . import views

    urlpatterns = [
        path('articles/<int:year>/', views.year_archive),
        path('articles/<int:year>/<int:month>/', views.month_archive),
        path('articles/<int:year>/<int:month>/<int:pk>/', views.article_detail),
    ]

ex: equested the URL `/articles/2005/05/39323/` -> 
  call function `news.views.article_detail(request, year=2005, month=5, pk=39323)`

------------------------------------------------------------------------------------
### Write your views

`mysite/news/views.py`

    from django.shortcuts import render
    from .models import Article

    def year_archive(request, year):
        a_list = Article.objects.filter(pub_date__year=year)
        context = {'year': year, 'article_list': a_list}
        return render(request, 'news/year_archive.html', context)

------------------------------------------------------------------------------------
### Design your templates

`mysite/news/templates/news/year_archive.html`

    {% extends "base.html" %}

    {% block title %}Articles for {{ year }}{% endblock %}

    {% block content %}
    <h1>Articles for {{ year }}</h1>

    {% for article in article_list %}
        <p>{{ article.headline }}</p>
        <p>By {{ article.reporter.full_name }}</p>
        <p>Published {{ article.pub_date|date:"F j, Y" }}</p>
    {% endfor %}
    {% endblock %}

`*data filter format: https://docs.djangoproject.com/en/2.0/howto/custom-template-tags/#howto-writing-custom-template-filters`

------------------------------------------------------------------------------------
### Design Templat for common-use

`mysite/templates/base.html`

    {% load static %}
    <html>
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
        <img src="{% static "images/sitelogo.png" %}" alt="Logo" />
        {% block content %}{% endblock %}
    </body>
    </html>

`*static: https://docs.djangoproject.com/en/2.0/howto/static-files/`

------------------------------------------------------------------------------------

### Appendix

##### Migrations

  migrate : applying and unapplying migrations. (applying those to database)
  
  makemigrations : creating new migrations based on the changes you have made to your models. (commits)
  
  sqlmigrate : displays the SQL statements for a migration.
  
  showmigrations : lists a project’s migrations and their status.  
