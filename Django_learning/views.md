## Definations
+ A view is a "type" of Web page in your Django application that generally serves a specific function and has a specific template. 
+ In Django, web pages and other content are all delivered by views. Django will choose a view by examining the URL that’s requested (to be precise, the part of the URL after the domain name).
+ A URL pattern is simply the general form of a URL - for example: `/newsarchive/<year>/<month>/`. To get from a URL to a view, Django uses what are known as `URLconfs`. A URLconf maps URL patterns to views.

## Writting more views
```py
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)
def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)
def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
+ These views are slightly different from `index`, because they take an
argument.

## How `urls.py` works?
+ Wire these new views into the polls.urls module
    ```py
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
    ```
+ Analyse the whole process
    1. Somebody requests a page from your website – say, `"/polls/34/"`
    2. Django loads the `mysite.urls` Python module because it’s pointed to by the `ROOT_URLCONF` setting. 
    3. Django finds the variable named `urlpatterns` and traverses the patterns in order. After finding the match at 'polls/', **it strips off the matching text `("polls/")` and sends the remaining text – "34/" – to the "polls.urls" URLconf for further processing.** 
    4. There it matches `<int:question_id>/`, resulting in a call to the `detail()` view like so: `detail(request=<HttpRequest object>, question_id=34)`. The question_id=34 part comes from `<int:question_id>`. Using angle brackets "captures" part of the URL and sends it as a keyword argument to the view function. The `:question_id>` part of the string defines the name that will be used to identify the matched pattern, and the `<int:` part is a converter that determines what patterns should match this part of the URL path.

## Writting more useful views
>Each view is responsible for doing one of two things: returning an HttpResponse object containing the content for the requested page, or raising an exception such as Http404. The rest is up to you. You can let the `views.py` do whatever you want. All Django wants is that HttpResponse. Or an exception.  

+ For example: 
    ```py
    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
    ```
    _This function make use of Django’s own database API._

### Using templates to create HttpResponse

+ The page’s design is hard-coded in the view. Changing the way the page looks can be difficult, you’ll have to edit this Python code. Using `template` that the view can use can be a better approach. **Django will look for a `templates` subdirectory in each of the `INSTALLED_APPS` and use the first template it matched.**
    >About Namespace
    Now we might be able to get away with putting our templates directly in `polls/templates` (rather than creating another polls subdirectory), but it would actually be a bad idea. Django will choose the first template it finds whose name matches, and if you had a template with the same name in a different application, Django would be unable to distinguish between them. We need to be able to point Django at the right one, and the easiest way to ensure this is by namespacing them. That is, by putting those templates inside another directory named for the application itself.
+ the `template`
    ```html
    {% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
    <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
    {% else %}
    <p>No polls are available.</p>
    {% endif %}
    ```
+ rewrite the `index()` using template
    ```py
    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        template = loader.get_template('polls/index.html')
        context = {
        'latest_question_list': latest_question_list,
        }
        return HttpResponse(template.render(context, request))
    ```
    + A shortcut function: `render()`
        ```py
        from django.shortcuts import render
        from .models import Question
        def index(request):
            latest_question_list = Question.objects.order_by('-pub_date')[:5]
            context = {'latest_question_list': latest_question_list}
            return render(request, 'polls/index.html', context)
        ```
        `render`() function takes three arguments
        1. the request object
        2. a template name
        3. a dictionary   
     
        It returns an `HttpResponse` object of the given template rendered with the given context.

### Remove hardcoded urls in our template
+ in the polls/index.html template, the link was partially hardcoded like this:
    ```html
    <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    ```
    this is not quite favorable---it becomes challenging to change URLs on projects with a lot of templates.

+ Remenber we have defined the `name` argument in the `path()` functions in the `polls.urls`module, you can remove a reliance on specific URL paths defined in your url configurations by using the `{% url %}` template tag:
    ```html
    <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
    ```
+ Suggest you later want to change the URL of the polls detail view to something else, perhaps to something like `polls/specifics/12/`, **instead of doing it in the template (or templates) you would change it in polls/urls.py:**
    ```py
    # added the word 'specifics'
    path('specifics/<int:question_id>/', views.detail, name='detail'),
    ```

## Raising 404 error
```py
from django.http import Http404
from django.shortcuts import render
from .models import Question
def detail(request, question_id):
    try:
    question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
    raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```
+ in `polls/templates/polls/detail.html`
    `{{ question }}`

+ shoutcut function `get_object_or_404()`
    ```py
    def detail(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/detail.html', {'question': question})
    ```

+ There’s also a `get_list_or_404()` function, which works just as `get_object_or_404()` – except using `filter()` instead of `get()`. It raises `Http404` if the list is empty.

>Philosophy
Why do we use a helper function get_object_or_404() instead of automatically catching the ObjectDoesNotExist exceptions at a higher level, or having the model API raise Http404 instead of ObjectDoesNotExist?
Because that would couple the model layer to the view layer. One of the foremost design goals of Django is to maintain **loose coupling**. Some controlled coupling is introduced in the django.shortcuts module.

## Django template namespaces
+ In real Django projects, there might be five, ten, twenty apps or more. How does Django differentiate the URL names between them? For example, the polls app has a detail view, and so might an app on the same project that is for a blog. How does one make it so that Django knows which app view to create for a url when using the {% url %} template tag? **The answer is to add namespaces to your URLconf.**

+ To create a template namespace, add `app_name` to urls.py
    ``` py
    from django.urls import path
    from . import views

    app_name='polls'm
    urlpatterns = [
        path('', views.index, name='index'),
        path('<int:question_id>/', views.detail, name='detail'),
        path('<int:question_id>/results/', views.results, name='results'),
        path('<int:question_id>/vote/', views.vote, name='vote'),
    ]
    ```
    then you can use 
    ```py
    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    ``` 
    ` to specify the certain `urlpattern`

## Writting a form
+ We are using `templates` to regulate a form
    in `polls/templates/polls/detail.html`
    ```html
    <h1>{{ question.question_text }}</h1>

    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    <form action="{% url 'polls:vote' question.id %}" method="post">
    {% csrf_token %}
    {% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{
    ˓→choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
    <input type="submit" value="Vote">
    </form>
    ```
    + Whenever you create a form that alters data server-side, use `method="post"`.
    + We need to worry about _Cross Site Request Forgeries_. In short, all POST forms that are targeted at internal URLs should use the `{% csrf_token %}` template tag.

+ Rewrite the `vote()` function to create a proper view
    ```py
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
    ```
    + **`request.POST` is a dictionary-like object that lets you access submitted data by key name.** In this case, `request.POST['choice']` returns the ID of the selected choice, as a string. **`request.POST` values are always strings**. It raise `KeyError` if the key does not exist.
    <br>
    + After incrementing the choice count, the code returns an `HttpResponseRedirect` rather than a normal HttpResponse. **`HttpResponseRedirect` takes a single argument: the URL to which the user will be redirected** (see the following point for how we construct the URL in this case). 
    <br>
    + The `reverse()` function in the `HttpResponseRedirect` constructor in this example. This function helps avoid having to hardcode a URL in the view function. **It is given the name of the view that we want to pass control to and the variable portion of the URL pattern that points to that view.** In this case, this `reverse()` call will return a string like `'/polls/3/results/'`
    <br>
+ We should also change the `result()` view to
    ```py
    def results(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/results.html', {'question': question})
    ```
    and change its template, `results.html` to
    ```HTML
    <h1>{{ question.question_text }}</h1>
    <ul>
    {% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
    {% endfor %}
    </ul>
    <a href="{% url 'polls:detail' question.id %}">Vote again?</a>
    ```
>**Note**: The code for our vote() view does have a small problem. It first gets the selected_choice object from the database, then computes the new value of votes, and then saves it back to the database. If two users of your website try to vote at exactly the same time, this might go wrong: The same value, let’s say 42, will be retrieved for votes. Then, for both users the new value of 43 is computed and saved, but 44 would be the expected value.
**This is called a race condition.** If you are interested, you can read Avoiding race conditions using F() to learn how you can solve this issue.

## Use generic views to limit the amount of code

+ Change `polls/urls.py` to
    ```py
    from django.urls import path
    from . import views
    app_name='polls'
    urlpatterns = [
        path('',views.IndexView.as_view(),name='index'),
        path('<int:pk>/',views.DetailView.as_view(),name='detail'),
        path('<int:pk>/results/',views.ResultsView.as_view(),name='results'),
        path('<int:question_id>/vote/',views.vote,name='vote'),
    ]
    ```
+ Change `polls/views.py` to
    ```py
    class IndexView(generic.ListView):
        template_name='polls/index.html'
        context_object_name='latest_question_list'

        def get_queryset(self):
            """return the last 5 posted questions"""
            return Question.objects.order_by('-pub_date')[:5]

    class DetailView(generic.DetailView):
        model=Question
        template_name='polls/detail.html'

    class ResultsView(generic.DetailView):
        model=Question
        template_name='polls/results.html'
    ```
### Tips
+ We're using two generic views here: `ListView` and `DetailView`. `ListView` refer to "display a list of objects" and `DetailView` refer to "display a detail page for a particular type of object."
+ Each generic view needs to know what model it will be acting upon. This is provided using the model attribute.
+ Notice the attribute is `pk` rather than `question_id`.
+ The `template_name` attribute is used to tell Django to use a specific template name instead of the autogenerated default template name, usually is `<app name>/<model name>_detail.html`
+ Similarly, the `ListView` generic view uses a default template called `<app name>/<model name>_list.html`; we use template_name to tell `ListView` to use our existing `"polls/index.html"` template.
+ For `DetailView` the question variable is provided automatically – since we’re using a Django model (Question), **Django is able to determine an appropriate name for the context variable.** 
+ For `ListView`, the automatically generated context variable is `question_list`. To override this we provide the `context_object_name` attribute, specifying that we want to use `latest_question_list` instead. 