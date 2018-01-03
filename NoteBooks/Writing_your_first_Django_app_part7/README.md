### Customize the admin form

##### This particular change makes the “Publication date” come before the “Question” field:

`polls/admin.py`

    from django.contrib import admin

    from .models import Question

    class QuestionAdmin(admin.ModelAdmin):
        fields = ['pub_date', 'question_text']

    admin.site.register(Question, QuestionAdmin)

##### As if it has dozens of fields, you might want to split the form up into fieldsets:

`polls/admin.py`

    from django.contrib import admin

    from .models import Question

    class QuestionAdmin(admin.ModelAdmin):
        fieldsets = [
            (None,               {'fields': ['question_text']}),
            ('Date information', {'fields': ['pub_date']}),
        ]

    admin.site.register(Question, QuestionAdmin)


------------------------------------------------------------------------------------
### Adding related objects

##### first way is to register Choice with the admin just as we did with Question.

`polls/admin.py`

    from django.contrib import admin

    from .models import Choice, Question
    # ...
    admin.site.register(Choice)

#####  better second way that add a bunch of Choices directly when you create the Question object.

`polls/admin.py`

    from django.contrib import admin

    from .models import Choice, Question

    class ChoiceInline(admin.StackedInline):
        model = Choice
        # provide enough fields for 3 choices (records existed exclude)
        extra = 3

    class QuestionAdmin(admin.ModelAdmin):
        fieldsets = [
            (None,               {'fields': ['question_text']}),
            ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}), # show/hide fields
        ]
        inlines = [ChoiceInline]

    admin.site.register(Question, QuestionAdmin)

##### displayed in a more compact, table-based format:

`polls/admin.py`

    class ChoiceInline(admin.TabularInline):
        #...


------------------------------------------------------------------------------------
### Customize the admin change list

##### A tuple of field names to display order of columns 

`polls/admin.py`

    class QuestionAdmin(admin.ModelAdmin):
        # ...
        list_display = ('question_text', 'pub_date', 'was_published_recently')

##### note that the column header 

`polls/models.py`

    class Question(models.Model):
        # ...
        def was_published_recently(self):
            now = timezone.now()
            return now - datetime.timedelta(days=1) <= self.pub_date <= now
        # order by pub_date
        was_published_recently.admin_order_field = 'pub_date'
        # default value
        was_published_recently.boolean = True
        # column display name
        was_published_recently.short_description = 'Published recently?'

[list_display](https://docs.djangoproject.com/en/2.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display)

##### Create list page filter & search (at QuestionAdmin)

`polls/admin.py`

    list_filter = ['pub_date']

    search_fields = ['question_text']

------------------------------------------------------------------------------------
### Customize the admin look and feel

##### Customizing your project’s templates

`mysite/settings.py`

    TEMPLATES = [
        {
            ...
            'DIRS': [os.path.join(BASE_DIR, 'templates')],
            ...
            
        },
    ]

_DIRS is a list of filesystem directories to check when loading Django templates;_

__Where are the Django source files?__ : 

    > python -c "import django; print(django.__path__)"

##### How to override templates

s1. create mysite/templates/admin/base_site.html 

s2. copy content from 'django/contrib/admin/templates/base_site.html'

s3. replace by below:

    {% block branding %}
    <h1 id="site-name"><a href="{% url 'admin:index' %}">Polls Administration</a></h1>
    {% endblock %}

[template loading documentation](https://docs.djangoproject.com/en/2.0/topics/templates/#template-loading)

