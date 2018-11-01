# Django_How_To_Use_Bootstrap_form
## Create Project
```
dev ➜  mkdir bootstrap_login_form && cd bootstrap_login_form
(bootstrap_login_form) ➜  virtualenv -p python3 .
bootstrap_login_form source bin/activate
(bootstrap_login_form) ➜  bootstrap_login_form  pip install django==2.0.7
(bootstrap_login_form) ➜  bootstrap_login_form pip install django-crispy-forms
(bootstrap_login_form) ➜  bootstrap_login_form mkdir src && cd src
(bootstrap_login_form) ➜  src django-admin startproject myproject .
```
## Create app (people) && templates
```
(bootstrap_login_form) ➜  src django-admin startapp people

(bootstrap_login_form) ➜  src touch templates/base.html
(bootstrap_login_form) ➜  src mkdir templates/people
(bootstrap_login_form) ➜  src touch templates/people/person_form.html
(bootstrap_login_form) ➜  src touch templates/people/person_list.html
(bootstrap_login_form) ➜  src touch templates/people/person_update_form.html

```
## Installation
```
pip install django-crispy-forms

```
### settings.py
```
INSTALLED_APPS = [
    ...

    'crispy_forms',
]

CRISPY_TEMPLATE_PACK = 'bootstrap4'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,'templates')],#new for templates
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
## Setup bootstrap in the base.html
```
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <title>Django People</title>
  </head>
  <body>
    <div class="container">
      <div class="row justify-content-center">
        <div class="col-8">
          <h1 class="mt-2">Django People</h1>
          <hr class="mt-0 mb-4">
          {% block content %}
          {% endblock %}
        </div>
      </div>
    </div>
  </body>
</html>
```
## models.py
```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=130)
    email = models.EmailField(blank=True)
    job_title = models.CharField(max_length=30, blank=True)
    bio = models.TextField(blank=True)
```
## view.py
```
Let’s say we wanted to create a view to add new Person objects. 
In that case we could use the built-in CreateView:

```
```
from django.views.generic import CreateView
from .models import Person

class PersonCreateView(CreateView):
    model = Person
    fields = ('name', 'email', 'job_title', 'bio')

```
## people/person_form.html
```
{% extends 'base.html' %}

{% load crispy_forms_tags %}

{% block content %}
  <form method="post" novalidate>
    {% csrf_token %}
    {{ form|crispy }}
    <button type="submit" class="btn btn-success">Save person</button>
  </form>
{% endblock %}

```
## Add another features in the fields
```
There are some cases where you may want more freedom to render your fields. You can do so by rendering the fields manually and using the as_crispy_field template filter:

```
```
{% extends 'base.html' %}

{% load crispy_forms_tags %}

**people/person_form.html**

{% block content %}
  <form method="post" novalidate>
    {% csrf_token %}
    <div class="row">
      <div class="col-6">
        {{ form.name|as_crispy_field }}
      </div>
      <div class="col-6">
        {{ form.email|as_crispy_field }}
      </div>
    </div>
    {{ form.job_title|as_crispy_field }}
    {{ form.bio|as_crispy_field }}
    <button type="submit" class="btn btn-success">Save person</button>
  </form>
{% endblock %}
```
## Form Helpers
```
The django-crispy-forms app have a special class named FormHelper to make your life easier and to give you complete control over how you want to render your forms.

Here is an example of an update view:
```
```
forms.py

from django import forms
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit
from people.models import Person

class PersonForm(forms.ModelForm):
    class Meta:
        model = Person
        fields = ('name', 'email', 'job_title', 'bio')

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.form_method = 'post'
        self.helper.add_input(Submit('submit', 'Save person'))
```
```
The job is done inside the __init__() method. The rest is just a regular Django model form. Here I’m defining that this form should handle the request using the POST method and the form should have an submit button with label “Save person”.

```
```
Now our view, just regular Django code:
```
## views.py
```
from django.views.generic import UpdateView
from people.models import Person
from people.forms import PersonForm

class PersonUpdateView(UpdateView):
    model = Person
    form_class = PersonForm
    template_name = 'people/person_update_form.html'
```
## people/person_update_form.html
```
{% extends 'base.html' %}

{% load crispy_forms_tags %}

{% block content %}
  {% crispy form %}
{% endblock %}
```
## Edit myproject/Urls.py
```
from django.urls import path

from people.views import PersonListView, PersonCreateView, PersonUpdateView


urlpatterns = [
    path('', PersonListView.as_view(), name='person_list'),
    path('add/', PersonCreateView.as_view(), name='person_add'),
    path('<int:pk>/edit/', PersonUpdateView.as_view(), name='person_edit'),
]
```
## people/person_list.html
```
{% extends 'base.html' %}

{% block content %}
  <p>
    <a href="{% url 'person_add' %}" class="btn btn-primary">Add person</a>
  </p>
  <table class="table table-bordered">
    <thead>
      <tr>
        <th>Name</th>
        <th>Email</th>
        <th>Job Title</th>
      </tr>
    </thead>
    <tbody>
      {% for person in people %}
        <tr>
          <td><a href="{% url 'person_edit' person.pk %}">{{ person.name }}</a></td>
          <td>{{ person.email }}</td>
          <td>{{ person.job_title }}</td>
        </tr>
      {% empty %}
        <tr class="table-active">
          <td colspan="3">No data</td>
        </tr>
      {% endfor %}
    </tbody>
  </table>
{% endblock %}
```
## Edit again views.py
```
from django.views.generic import ListView, CreateView, UpdateView
from django.urls import reverse_lazy

from people.models import Person
from people.forms import PersonForm


class PersonListView(ListView):
    model = Person
    context_object_name = 'people'


class PersonCreateView(CreateView):
    model = Person
    fields = ('name', 'email', 'job_title', 'bio')
    success_url = reverse_lazy('person_list')


class PersonUpdateView(UpdateView):
    model = Person
    form_class = PersonForm
    template_name = 'people/person_update_form.html'
    success_url = reverse_lazy('person_list')
```
I learn From:
https://simpleisbetterthancomplex.com/tutorial/2018/08/13/how-to-use-bootstrap-4-forms-with-django.html
