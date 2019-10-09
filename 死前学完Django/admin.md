# Admin
+ Django creates a unified interface for site administrators to edit content.
## Creating an admin user
+ Run the command: `python manage.py createsuperuser`
+ You can see this page at after logging into it  
  ![admin page](admin_page.png)
  You should see a few types of editable content: `groups` and `users`. They are provided by `django.contrib.auth`, the authentication framework shipped by Django.
+ we need to tell the admin that `Question` objects have an admin interface. To do this, open the polls/admin.py file, and edit it to look like this:
    ```py
    from django.contrib import admin
    from .models import Question
    admin.site.register(Question)
    ```
