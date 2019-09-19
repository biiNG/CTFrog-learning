# Model

## Definations

### Models

*Database layout, with additional metadata.*

### Migration

Migrations are entirely derived from your models file, and are essentially just a history that Django can roll through to update your database schema to match your current models.

### Implementations

These concepts are represented by simple Python classes.

```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

#### *NOTICE*

1. Each model is represented by a class that subclasses `django.db.models.Model`.
2. Each model has a number of class variables, each of which represents a database field in the model.
3. Each "field" is represented by an instance of a `Field` class – e.g., `CharField` for character fields and `DateTimeField` for datetimes.
4. The name of each `Field` instance (e.g. `question_text` or `pub_date`) is the field’s name, they're their "machine-friendly" name.  
You can also use an optional first positional argument to a `Field` to designate a human-readable name, just like

    ```python
    pub_date = models.DateTimeField('date published')
    ```  

    The "human readable name" is used in a couple of introspective parts of Django, and it doubles as documentation.
5. Some Field classes have required arguments. CharField, for example, requires that you give it a max_length. That’s used not only in the database schema, but in validation.

### Activating Models

1. Configure the `settings.py` to tell our app has been installed already.
    1. Edit `mysite/settings.py` and add that dotted path to the `INSTALLED_APPS` setting.

    ```python
    INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    ]
    ```

2. Now Django knows to include the polls app. Make migration for it.
In cmd: `python manage.py makemigrations polls`
By running makemigrations, you’re telling Django that you’ve made some changes to your models (in this case, you’ve made new ones) and that you’d like the changes to be stored as a migration.

3. The `sqlmigrate` command takes migration names and returns their SQL:

    ```
    python manage.py sqlmigrate polls 0001
    ```

    ***The sqlmigrate command doesn’t actually run the migration on your database - it just prints it to the screen so that you can see what SQL Django thinks is required. It’s useful for checking what Django is going to do or if you have database administrators who require SQL scripts for changes.***

4. Input the `migrate` command, which will run the migrations for you and manage your database schema automatically. After completing the former steps, using `migrate` to create those model tabels in your database. 
`python manage.py migrate`

### Interactive with APIs---*Shell*

`python manage.py shell`
We’re using this instead of simply typing `python`, because `manage.py` sets the `DJANGO_SETTINGS_MODULE` environment variable, which gives Django the Python import path to your `mysite/settings.py` file.

+ Using the model to create a `Question`: 

    ```python
    from django.utils import timezone
    q = Question(question_text="What's new?", pub_date=timezone.now())
    ```
+ Save a `Question`: `q.save()`

+ Checking all the `Questions`: `Question.objects.all()`

+ Get control of a certain `Question`: `Question.objects.get(id=1)` or `Question.objects.get(pk=1)`

+ Add `Choice` to it:

    ```python     
    q = Question.objects.get(pk=1)
    q.choice_set.all()
    q.choice_set.create(choice_text='Not much', votes=0)
    ```

+ Check a `Choice`'s `Question`: `c.question`
