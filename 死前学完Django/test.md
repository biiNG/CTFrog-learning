## Some philosophy
+ What is test?
  + Test is the module we create, and then as we make changes to our apps, we can check that our code still works as you originally intended, without having to perform time consuming manual testing.
+ Why we need test?
  + Time-saving
  + Test light up code from inside, when something is wrong, you can find where is wrong quickly.
  + Makes sure others won't change your design.

##　Writting test
+ There is a bug in our `polls` app
    ```py
    $ python manage.py shell
    >>> import datetime
    >>> from django.utils import timezone
    >>> from polls.models import Question
    >>> # create a Question instance with pub_date 30 days in the future
    >>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
    >>> # was it published recently?
    >>> future_question.was_published_recently()
    True
    ```
+ Create a test according to this bug
    in `polls/tests.py`
    ```py
    import datetime
    from django.test import TestCase
    from django.utils import timezone
    from .models import Question
    class QuestionModelTests(TestCase):
        def test_was_published_recently_with_future_question(self):
            """
            was_published_recently() returns False for questions whose pub_date
            is in the future.
            """
            time = timezone.now() + datetime.timedelta(days=30)
            future_question = Question(pub_date=time)
            self.assertIs(future_question.was_published_recently(), False)
    ```

+ Running tests
    ```shell
    $ python manage.py test polls
    ```

+ Analyse the process of it
    1. `manage.py test polls` looked for tests in the `polls` application
    2. it found a subclass of the `django.test.TestCase` class
    3. it created a special database for the purpose of testing
    4. it looked for test methods - ones whose names begin with `test`
    5. in `test_was_published_recently_with_future_question` it created a `Question` instance whose `pub_date` field is 30 days in the future
    6. using the `assertIs()` method, it discovered that its `was_published_recently()` returns `True`, though we wanted it to return `False`