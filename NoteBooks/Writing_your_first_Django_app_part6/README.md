### Customize your app’s look and feel

s1. create a directory called static in your polls directory

s2. create a file called style.css that should be at polls/static/polls/style.css.

`polls/static/polls/style.css`

    li a {
        color: green;
    }

`polls/templates/polls/index.html`

    {% load static %}

    <link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />

------------------------------------------------------------------------------------
### Adding a background-image

`polls/static/polls/style.css`

    body {
        background: white url("images/background.png") no-repeat right bottom;
    }

------------------------------------------------------------------------------------
### Advance

1. [Managing static files](https://docs.djangoproject.com/en/2.0/howto/static-files/)

2. [The staticfiles reference](https://docs.djangoproject.com/en/2.0/ref/contrib/staticfiles/)

3. [Deploying static files](https://docs.djangoproject.com/en/2.0/howto/static-files/deployment/)
