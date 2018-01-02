##### Writing more views

`polls/views.py`

    def detail(request, question_id):
        return HttpResponse("You're looking at question %s." % question_id)

    def results(request, question_id):
        response = "You're looking at the results of question %s."
        return HttpResponse(response % question_id)

    def vote(request, question_id):
        return HttpResponse("You're voting on question %s." % question_id)
    Wire these new views into the polls.urls module by adding the following path() calls:

`polls/urls.py`

    from django.urls import path

    from . import views

    urlpatterns = [
        # ex: /polls/
        path('', views.index, name='index'),
        # ex: /polls/5/
        path('<int:question_id>/', views.detail, name='detail'),
        # ex: /polls/5/results/
        path('<int:question_id>/results/', views.results, name='results'),
        # ex: /polls/5/vote/
        path('<int:question_id>/vote/', views.vote, name='vote'),
    ]

------------------------------------------------------------------------------------
##### Write views that actually do something

`polls/views.py`

1. modify new index() view, which displays the latest 5 poll questions in the system, separated by commas, according to publication date:

    from django.http import HttpResponse
    from .models import Question

    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        output = ', '.join([q.question_text for q in latest_question_list])
        return HttpResponse(output)

2. Create template

2.1 The default settings file configures a DjangoTemplates backend whose APP_DIRS option is set to True. 
By convention DjangoTemplates looks for a “templates” subdirectory in each of the INSTALLED_APPS.

`polls/templates/polls/index.html`

    {% if latest_question_list %}
        <ul>
        {% for question in latest_question_list %}
            <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
        {% endfor %}
        </ul>
    {% else %}
        <p>No polls are available.</p>
    {% endif %}

2.2 update our index view in polls/views.py to use the template:

`polls/views.py`

    from django.http import HttpResponse
    from django.template import loader
    from .models import Question

    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        template = loader.get_template('polls/index.html')
        context = {
            'latest_question_list': latest_question_list,
        }
        return HttpResponse(template.render(context, request))

2.3 update our index view in polls/views.py to use the template:

`polls/views.py`

    from django.http import HttpResponse
    from django.template import loader

    from .models import Question

    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        template = loader.get_template('polls/index.html')
        context = {
            'latest_question_list': latest_question_list,
        }
        return HttpResponse(template.render(context, request))

2.4 full index() view rewritten as use shortcut: render()

`polls/views.py`

    from django.shortcuts import render
    from .models import Question

    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        context = {'latest_question_list': latest_question_list}
        return render(request, 'polls/index.html', context)

2.5 Raising a 404 error

`polls/views.py`

    from django.http import Http404
    from django.shortcuts import render
    from .models import Question
    # ...
    def detail(request, question_id):
        try:
            question = Question.objects.get(pk=question_id)
        except Question.DoesNotExist:
            raise Http404("Question does not exist")
        return render(request, 'polls/detail.html', {'question': question})

`polls/templates/polls/detail.html`

    {{ question }}

2.6 A shortcut: get_object_or_404()

`polls/views.py`

    from django.shortcuts import get_object_or_404, render

    from .models import Question
    # ...
    def detail(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/detail.html', {'question': question})

* There’s also a get_list_or_404() function – except using filter() instead of get(). It raises Http404 if the list is empty.

------------------------------------------------------------------------------------
##### Use the template system

`polls/templates/polls/detail.html`

    <h1>{{ question.question_text }}</h1>
    <ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }}</li>
    {% endfor %}
    </ul>

------------------------------------------------------------------------------------
##### Removing hardcoded URLs in templates

`polls/templates/polls/index.html`

    ...
    <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

* URL definition as specified in the polls.urls

the 'name' value as called by the {% url %} template tag

    path('<int:question_id>/', views.detail, name='detail'),

* If you want to change the URL of the polls detail view to something else, perhaps to something like polls/specifics/12/ instead of doing it in the template (or templates) you would change it in polls/urls.py:

added the word 'specifics'

    path('specifics/<int:question_id>/', views.detail, name='detail'),

------------------------------------------------------------------------------------
##### Namespacing URL names

1. add namespace

`polls/urls.py`

    app_name = 'polls'

2. change template from:

`polls/templates/polls/index.html`

    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    