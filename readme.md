# Google Login & Signup with Django

This tutorial will teach you all you need to know to get started with authenticating your users with google. This tutorial does assume you have your project setup already

## Getting Started

### 1. Opening django project

- I am going to assume you have a django project created already
- Open your project in your code editor

### 2. Create a new project on google
- Head to google cloud consle below:
    ```https://console.cloud.google.com/```

- Click on the project tab and select 'New Project'

- Give your project a name and click create

- Wait for project to get created

### 3. Setup OAUTH Consent Screen

This is where we setup what a user sees when they click 'Signin with google'. We need to set this up before we can get OAuth credentials

- Select project after it has been created

- Expand navigation menu on the left and select:
    - APIs & Services > OAuth Consent Screen

- Under user type, select external & click Create

- Under the form, 
    - Fill in app name & select user support email
    - Upload a logo (optional)
    - optionally add app domains 
    - We will leave authorized domains for now
    - Fill in a developer contact info
    - click save and continue

- Next we need to add scopes for the information we want to get access to from the google account:
    - Click Add or remove scopes
    - Select the first 2 options (These will give us access to information lile a user's email, name, profile photo, ....)
    - Click update
    - Click save and continue

- Adding test users:
    - We can skip this step because we will successfully be able to login without adding an email as a test user 

    - Click save and continue

- Summary
    - Make sure all the info are correct
    - Back to dashboard

### 4. Create Credentials
We are going to make use of these credentials in our django project 

- Click credentials in the side nav

- Click:
    ```Create credentials > OAuth client Id```

- Appication type should be web application

- Give it a name

- Next, we ned to fill authorised redirect url:
    - Add the following URIs:
        ```http://127.0.0.1:8000```
        ```http://127.0.0.1:8000/accounts/google/login/callback/```

        make sure you add exactly those urls

    - Click create
    - Download created credentials

### 5. Install allauth

- Run the following command to install allauth
    ```bash
    pip install "django-allauth[socialaccount]"
    ```

### 6. Configuration in settings.py
- Add a ```SITE_ID``` variable in settings.py:
    ```python
    SITE_ID = 1
    ```
- Add the following to your ```INSTALLED_APPS```:
    ```python
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
    ```

- Next is to specify authentication backends
    ```python
    AUTHENTICATION_BACKENDS = [
        'django.contrib.auth.backends.ModelBackend', # standard django backend
        'allauth.account.auth_backends.AuthenticationBackend', # all auth backend
    ]
    ```

- Next, specify the login and logout redirect urls:
     ```python
    LOGIN_REDIRECT_URL = '/'
    LOGOUT_REDIRECT_URL = '/'
    SOCIALACCOUNT_LOGIN_ON_GET = True
    ```

- Add the following to the top of the ```MIDDLEWARE``` list:
    ```
    'allauth.account.middleware.AccountMiddleware',
    ```

### 7. Including allauth urls in urls.py:

- Head to your urls.py file in your project folder
- Add the following:
    ```python
    path('accounts/', include('allauth.urls')),
    ```

### 8. Update template button with login link
- Add the ```socialaccount``` template tag to make use of google login url:
    ```html
    {% load socialaccount %}
    <br><button onclick="window.location.href = '{% provider_login_url 'google' %}' " type="button" class="login-with-google-btn" >
        <b>Sign in with Google</b>
    </button>
    ```

### 9. Setup social account in admin

- Create a superuser to access django admin dashboard:
    ```bash
    python manage.py createsuperuser
    ```

- Click Sites & click the first entry to edit

- Change domain & display names to:
    ```url
    http://127.0.0.1:8000
    ```

- Click social applications and add a new social application:
    - Select google as provider
    - give it a name 'google'
    - Get client id and secret from downloaded json file
    - Select what site this provider is for 
    - Click save

### 10. Runserver and test to see if user's can signin

- In a different browser, visit base url and attempt to sign in using google account
- After successful sign in, select social accounts in django admin dashboard to see connected googlee account
- Click users to see the newely created user

### 11. Create a signal to listen to user signup

- Create a new file called ```signals.py``` in your app folder
- Add the following imports:
    ```python
    from django.dispatch import receiver
    from allauth.account.signals import user_signed_up
    ```
- Create a receiver signal that listens and runs when a new user signs up with google:
    ```python
    @receiver(user_signed_up)
    def handle_user_signed_up(request, sociallogin, user, **kwargs):

        # grab the user's data
        new_user_data = sociallogin.account.extra_data

        print(new_user_data)

        # perform tasks/processing on data

    ```

- Import signal into ```app.py``` file:
    ```python
    from django.apps import AppConfig

    class CoreConfig(AppConfig):
        default_auto_field = 'django.db.models.BigAutoField'
        name = 'Core'

        # create ready method
        def ready(self):
            # import signals
            import Core.signals
    ```
