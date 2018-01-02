### Write a simple form

`polls/templates/polls/detail.html`

    <h1>{{ question.question_text }}</h1>

    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

    <form action="{% url 'polls:vote' question.id %}" method="post">
    {% csrf_token %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
    {% endfor %}
    <input type="submit" value="Vote" />
    </form>

* forloop.counter indicates how many times the for tag has gone through its loop

------------------------------------------------------------------------------------
### Create a Django view that handles the submitted data

`polls/urls.py`

    path('<int:question_id>/vote/', views.vote, name='vote'),

------------------------------------------------------------------------------------
### Created a dummy implementation of the vote() function.

`polls/views.py`

    from django.shortcuts import get_object_or_404, render
    from django.http import HttpResponseRedirect, HttpResponse
    from django.urls import reverse

    from .models import Choice, Question
    # ...
    def vote(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        try:
            selected_choice = question.choice_set.get(pk=request.POST['choice'])
        except (KeyError, Choice.DoesNotExist):
            # Redisplay the question voting form.
            return render(request, 'polls/detail.html', {
                'question': question,
                'error_message': "You didn't select a choice.",
            })
        else:
            selected_choice.votes += 1
            selected_choice.save()
            # Always return an HttpResponseRedirect after successfully dealing
            # with POST data. This prevents data from being posted twice if a
            # user hits the Back button.
            return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))

------------------------------------------------------------------------------------
### Write vote() view redirects to the results page for the question.

`polls/views.py`

    from django.shortcuts import get_object_or_404, render

    def results(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/results.html', {'question': question})

------------------------------------------------------------------------------------
### Create a polls/results.html template

`polls/templates/polls/results.html`

    <h1>{{ question.question_text }}</h1>

    <ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
    {% endfor %}
    </ul>

    <a href="{% url 'polls:detail' question.id %}">Vote again?</a>

------------------------------------------------------------------------------------
### Process Flow

(browser) http://127.0.0.1:8000/polls/1/ -> (polls/views) views.detail -> 
(templates/polls) polls/detail.html -> 'Submit' -> /polls/1/vote/ -> (polls/views) views.vote
-> 'OK' -> (polls/views) views.results -> (templates/polls) polls/results.html
-> 'Error' -> (templates/polls) polls/detail.html

------------------------------------------------------------------------------------
### Avoiding race conditions

If two users of your website try to vote at exactly the same time, this might go wrong: The same value, let’s say 42, will be retrieved for votes. Then, for both users the new value of 43 is computed and saved, but 44 would be the expected value.

`Avoiding race conditions using F() : https://docs.djangoproject.com/en/2.0/ref/models/expressions/#avoiding-race-conditions-using-f`

------------------------------------------------------------------------------------
### Use generic views: Less code is better

1. Amend URLconf

`polls/urls.py`

    from django.urls import path

    from . import views

    app_name = 'polls'
    urlpatterns = [
        path('', views.IndexView.as_view(), name='index'),
        path('<int:pk>/', views.DetailView.as_view(), name='detail'),
        path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
        path('<int:question_id>/vote/', views.vote, name='vote'),
    ]

2. Amend views

`polls/views.py`

    from django.shortcuts import get_object_or_404, render
    from django.http import HttpResponseRedirect
    from django.urls import reverse
    from django.views import generic

    from .models import Choice, Question

    class IndexView(generic.ListView):
        template_name = 'polls/index.html'
        context_object_name = 'latest_question_list'

        def get_queryset(self):
            """Return the last five published questions."""
            return Question.objects.order_by('-pub_date')[:5]

    class DetailView(generic.DetailView):
        model = Question
        template_name = 'polls/detail.html'


    class ResultsView(generic.DetailView):
        model = Question
        template_name = 'polls/results.html'

    def vote(request, question_id):
        ... # same as above, no changes needed.