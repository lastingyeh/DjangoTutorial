### Database setup

`mysite/settings.py`

[DB setting](https://docs.djangoproject.com/en/2.0/ref/databases/#third-party-notes)

[reference documentation](https://docs.djangoproject.com/en/2.0/ref/settings/#std:setting-DATABASES)

------------------------------------------------------------------------------------
### TIME_ZONE

`mysite/settings.py`

[set TIME_ZONE](https://docs.djangoproject.com/en/2.0/ref/settings/#std:setting-TIME_ZONE)

    Note : at Windows environment. TIME_ZONE must be set to match the system time zone.

------------------------------------------------------------------------------------
### INSTALLED_APPS 

    django.contrib.admin – The admin site. You’ll use it shortly.
    
    django.contrib.auth – An authentication system.
    
    django.contrib.contenttypes – A framework for content types.
    
    django.contrib.sessions – A session framework.
    
    django.contrib.messages – A messaging framework.
    
    django.contrib.staticfiles – A framework for managing static files.

------------------------------------------------------------------------------------
### create the tables in the database before we can use them.

    creates any necessary database tables according to the database settings in your mysite/settings.py file 
    and the database migrations shipped with the app

*> python manage.py migrate*

------------------------------------------------------------------------------------
### Creating models

`polls/models.py`

    from django.db import models

    class Question(models.Model):
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')


    class Choice(models.Model):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)

------------------------------------------------------------------------------------
### Activating models

1. `mysite/settings.py`

    INSTALLED_APPS = [
        'polls.apps.PollsConfig',
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    ]

2. *> python manage.py makemigrations polls*

        if it made some changes to your models (in this case, you’ve made new ones) 
        and that you’d like the changes to be stored as a migration.

3. *> python manage.py sqlmigrate polls 0001*

        The sqlmigrate command doesn’t actually run the migration on your database - it just prints it to the screen so that you can see what SQL Django thinks is required. It’s useful for checking what Django is going to do or if you have database administrators who require SQL scripts for changes.

4. *> python manage.py check*

        checks for any problems in your project without making migrations or touching the database.

5. *> python manage.py migrate*

        create those model tables in your database

------------------------------------------------------------------------------------
### Three-step guide to making model changes

    1. Change your models (in models.py)
    
    2. Run python manage.py makemigrations to create migrations for those changes
    
    3. Run python manage.py migrate to apply those changes to the database.

------------------------------------------------------------------------------------
### Playing with the API

*> python manage.py shell*

*>>> import django*

*>>> django.setup()*

    *If this raises an AttributeError, you’re probably using a version of Django that doesn’t match this tutorial version. 
    You must run python from the same directory manage.py is in, or ensure that directory is on the Python path, so that import mysite works.

Base API Test at shell

*>>> from polls.models import Question, Choice*

*>>> Question.objects.all()*

<QuerySet []>

*>>> from django.utils import timezone*

*>>> q = Question(question_text="What's new?", pub_date=timezone.now())*

*>>> q.save()*

*>>> q.id*

1

*>>> q.question_text*

"What's new?"

*>>> q.pub_date*

datetime.datetime(2017, 12, 29, 2, 51, 48, 641367, tzinfo=<UTC>)

*>>> q.question_text = "What's up?"*

*>>> q.save()*

*>>> Question.objects.all()*

<QuerySet [<Question: Question object (1)>]>

*>>> exit()*

Modify Models and test at shell

    `polls/models.py`

    import datetime

    from django.db import models
    from django.utils import timezone

    class Question(models.Model):
        # ...
        def __str__(self):
            return self.question_text

        # self-defined method
        def was_published_recently(self):
            return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    class Choice(models.Model):
        # ...
        def __str__(self):
            return self.choice_text

*>>> from polls.models import Question, Choice*

*>>> Question.objects.all()*

<QuerySet [<Question: What's up?>]>

*>>> Question.objects.filter(id=1)*

<QuerySet [<Question: What's up?>]>

*>>> Question.objects.filter(question_text__startswith='What')*

<QuerySet [<Question: What's up?>]>

*>>> from django.utils import timezone*

*>>> current_year = timezone.now().year*

*>>> Question.objects.get(pub_date__year=current_year)*

<Question: What's up?>

*>>> Question.objects.get(id=2)*

...
polls.models.DoesNotExist: Question matching query does not exist.

*>>> Question.objects.get(pk=1)*

<Question: What's up?>

*>>> q = Question.objects.get(pk=1)*

*>>> q.was_published_recently()*
True

*>>> q.choice_set.all()*

<QuerySet []>

*>>> q.choice_set.create(choice_text='Not much', votes=0)*

<Choice: Not much>

*>>> q.choice_set.create(choice_text='The sky', votes=0)*

<Choice: The sky>

*>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)*

*>>> c.question*

<Question: What's up?>

*>>> q.choice_set.all()*

<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

*>>> q.choice_set.count()*

3

*>>> Choice.objects.filter(question__pub_date__year=current_year)*

<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

*>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')*

*>>> c.delete()*

(1, {'polls.Choice': 1})

------------------------------------------------------------------------------------
### Introducing the Django Admin

    1. Creating an admin user
    
        *> python manage.py createsuperuser*
    
    2. Start the development server
    
        *> python manage.py runserver*
    
        *open a Web browser : http://127.0.0.1:8000/admin/*
    
    3. Make the poll app modifiable in the admin

`polls/admin.py`

    from django.contrib import admin

    from .models import Question

    admin.site.register(Question)

    4. Explore the free admin functionality

[refs](https://docs.djangoproject.com/en/2.0/intro/tutorial02/#explore-the-free-admin-functionality)