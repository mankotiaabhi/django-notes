# settings.py

settings.py is the heart of a Django project.

It contains all the configuration of your Django installation, controlling aspects like:

- Database settings
- Installed applications
- Middleware
- URL configuration
- Static file directories
- Much more

Understanding this file is very crucial for any Django developer, as it allows you to customize your project to meet specific requirements.

In this document, we will walk through each section of a typical settings.py.

## Import os and Path

```python
import os
from pathlib import Path
```

These lines import the os module and the Path class from the pathlib module. These are used to handle file paths in a way that's compatible with different operating systems.

## Base Directory

```python
BASE_DIR = Path(__file__).resolve().parent.parent
```

This line sets the BASE_DIR variable to the parent directory of the directory containing the settings.py file. 
This is typically the root directory of your Django project.
It's used as a reference point for other file paths in the settings.

## Secret Key

```python
SECRET_KEY = 'your-secret-key-here'
```

This secret key is used for cryptographic signing in Django. 
It should be kept secret and should be unique for each Django installation. 
In production, you should never hardcode this in your settings file. Instead, you can use environment variables:

```python
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
```

## Debug Mode

```python
DEBUG = True
```

Debug mode provides detailed error pages and should be set to False in production. You can use an environment variable to control this:

```python
DEBUG = os.environ.get('DJANGO_DEBUG', '') != 'False'
```

## Allowed Hosts

```python
ALLOWED_HOSTS = []
```

This is a list of host/domain names that your Django site can serve. This is a security measure to prevent HTTP Host header attacks. For development, you can use:

```python
ALLOWED_HOSTS = ['localhost', '127.0.0.1']
```

For production, you'd list your domain name like this:

```python
ALLOWED_HOSTS = ['www.yourdomain.com']
```

## Installed Apps

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

This tells Django which applications are active for this project. The default list includes Django's built-in applications. 
You'll add your own applications to the list as you create them:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # your custom app
    'another_app',  # another custom app
]
```

## Middleware

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

Middleware is a framework of hooks into Django's request/response processing.

It's a light, low-level "plugin" system for globally altering Django's input or output. You might add custom middleware here:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'myproject.middleware.CustomMiddleware',  # your custom middleware
]
```

## URL Configuration

```python
ROOT_URLCONF = 'myproject.urls'
```

In Django, `ROOT_URLCONF` points to the Python module that defines the URL routing for the project. A typical `urls.py` contains `path()` or `re_path()` entries and may use `include()` to delegate routing to app-specific modules.

Example:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

Key idea:
- `urlpatterns` maps request paths to views.
- `include()` allows modular URL routing.
- Order matters: the first matching pattern is used.

## Templates

The `TEMPLATES` setting configures Django’s template engine and search paths.

Example:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
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

Explanation:
- `BACKEND` selects the template engine.
- `DIRS` lists project-level template folders.
- `APP_DIRS` enables discovery of `templates/` inside apps.
- `context_processors` add common variables to every template context.

## WSGI Application

```python
WSGI_APPLICATION = 'myproject.wsgi.application'
```

`WSGI_APPLICATION` specifies the callable used by WSGI servers such as Gunicorn or uWSGI.

The corresponding `wsgi.py` usually looks like:

```python
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')
application = get_wsgi_application()
```

This makes Django accessible to a web server via the WSGI interface.

## What is WSGI?

WSGI, or Web Server Gateway Interface, is a specification that defines a standard interface between web servers and Python web applications or frameworks. It allows web servers to communicate with Python applications in a standardized way, enabling the deployment of Python web apps on various servers without tight coupling. WSGI acts as a bridge, handling the protocol for how requests are passed from the server to the application and responses back.

### History of WSGI

WSGI was first proposed in 2003 by Philip J. Eby as part of Python Enhancement Proposal (PEP) 333. At the time, Python web frameworks like Zope and Quixote had their own ways of interfacing with web servers, leading to fragmentation and incompatibility. The goal was to create a simple, universal interface inspired by CGI but more efficient for long-running applications. It was standardized for Python 2 in PEP 333 and updated for Python 3 in PEP 3333 in 2010. This standardization helped unify the Python web ecosystem, making it easier to deploy apps like Django and Flask across different servers.

### How WSGI Works: Real-Time Examples

WSGI defines two sides: the server side (which calls the application) and the application side (a callable that processes requests). A WSGI application is a Python callable (function or class) that takes two arguments: `environ` (a dictionary of CGI-style environment variables) and `start_response` (a callable to begin the response).

Here's a simple WSGI application example (from Wikipedia's WSGI page):

```python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return [b'Hello, World!']
```

This app returns a basic "Hello, World!" response. In a real deployment, a WSGI server like Gunicorn or uWSGI would call this function for each HTTP request, passing the `environ` dict with details like request method, URL, and headers.

For a more complex example, consider Django's WSGI integration. Django provides a `wsgi.py` file that exposes the app as a WSGI callable:

```python
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')
application = get_wsgi_application()
```

This allows servers to run Django apps. In production, tools like Gunicorn use WSGI to serve multiple requests concurrently, improving performance over CGI.

WSGI also supports middleware, which can wrap applications for tasks like logging or authentication. For instance, a middleware might modify the `environ` before passing it to the app.

In summary, WSGI revolutionized Python web deployment by providing a standard, allowing frameworks to focus on app logic while servers handle HTTP. It's widely used today in tools like Apache's mod_wsgi or Nginx with uWSGI, ensuring portability and efficiency.

## Database Configuration

`DATABASES` defines the database engines and connection settings.

Example for SQLite:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

Example for PostgreSQL:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('POSTGRES_DB'),
        'USER': os.environ.get('POSTGRES_USER'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': os.environ.get('POSTGRES_HOST', 'localhost'),
        'PORT': os.environ.get('POSTGRES_PORT', '5432'),
    }
}
```

Important fields:
- `ENGINE`
- `NAME`
- `USER`
- `PASSWORD`
- `HOST`
- `PORT`

## Password Validation

`AUTH_PASSWORD_VALIDATORS` configures password strength checks.

Example:

```python
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator', 'OPTIONS': {'min_length': 8}},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]
```

These validators help enforce strong user passwords and reduce security risk.

## Internationalization

Internationalization settings control language and timezone support.

Example:

```python
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_L10N = True
USE_TZ = True
```

Meaning:
- `LANGUAGE_CODE`: default language.
- `TIME_ZONE`: project time zone.
- `USE_I18N`: enable translation machinery.
- `USE_L10N`: format dates/numbers according to locale.
- `USE_TZ`: store datetimes in UTC and convert for users.

## Static Files

Static files are CSS, JavaScript, and images served by Django or a web server.

Example:

```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / 'static']
STATIC_ROOT = BASE_DIR / 'staticfiles'
```

For user-uploaded media:

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

Explanation:
- `STATIC_URL`: public URL prefix.
- `STATICFILES_DIRS`: folders for development.
- `STATIC_ROOT`: folder used by `collectstatic` in production.

## Default Auto Field

`DEFAULT_AUTO_FIELD` sets the type of primary key field for models created without an explicit `id`.

Example:

```python
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

This uses a 64-bit integer for automatic primary keys, avoiding overflow issues in large databases.

## What are the default Django apps inside it? Are there more?

Django ships with several built-in apps under `django.contrib`:
- `admin`
- `auth`
- `contenttypes`
- `sessions`
- `messages`
- `staticfiles`

There are more contrib apps available, such as:
- `django.contrib.sites`
- `django.contrib.humanize`
- `django.contrib.flatpages`
- `django.contrib.redirects`

Your project can also include custom apps and third-party packages.

## What is middleware? What are different kinds of middleware? Read up a little on each security issue.

Middleware are hooks that process requests and responses globally.

Example default middleware stack:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

Types of middleware:
- Security middleware: enforces HTTPS, headers, clickjacking protection.
- Session middleware: manages session cookies.
- Authentication middleware: attaches `request.user`.
- CSRF middleware: protects forms from cross-site request forgery.
- Message middleware: enables flash messages.
- Common middleware: URL normalization, content-length handling.

Middleware order is important because each one wraps the request and response.

## Read up on Django Security

Django includes features to make applications safer. Important topics:

- CSRF: Protects POST requests from unauthorized sites. Use `CsrfViewMiddleware` and include `{% csrf_token %}` in forms.
- XSS: Cross-site scripting attacks are prevented by Django’s automatic HTML escaping in templates. Use `{{ value }}` rather than `|safe` unless trusted.
- Clickjacking: Prevents pages from being framed by other sites. `XFrameOptionsMiddleware` sets the `X-Frame-Options` header.

Additional security settings:
- `SECURE_SSL_REDIRECT`
- `SESSION_COOKIE_SECURE`
- `CSRF_COOKIE_SECURE`
- `SECURE_HSTS_SECONDS`
- `SECURE_BROWSER_XSS_FILTER`
- `SECURE_CONTENT_TYPE_NOSNIFF`

Use environment variables for secrets and debug controls, and never deploy with `DEBUG = True`.

## Summary

A Django `settings.py` file organizes:
- Imports and base directory
- Secrets and debug mode
- Allowed hosts
- Installed apps
- Middleware
- URL routing
- Templates
- WSGI setup
- Database connections
- Password validation
- Localization
- Static/media handling
- Default model field behavior
- Security configuration

Each section is essential to build a secure, maintainable Django project.

## Quick Reference Table

| Setting | Purpose | Example Value | Notes |
|---------|---------|---------------|-------|
| `BASE_DIR` | Root directory of the project | `Path(__file__).resolve().parent.parent` | Used as reference for other paths |
| `SECRET_KEY` | Cryptographic signing key | `os.environ.get('DJANGO_SECRET_KEY')` | Keep secret, use env vars in production |
| `DEBUG` | Enable detailed error pages | `os.environ.get('DJANGO_DEBUG', '') != 'False'` | Set to False in production |
| `ALLOWED_HOSTS` | Permitted host/domain names | `['localhost', '127.0.0.1']` | Security measure against host header attacks |
| `INSTALLED_APPS` | Active applications | `['django.contrib.admin', 'myapp']` | Include Django's built-in and custom apps |
| `MIDDLEWARE` | Request/response processors | `['django.middleware.security.SecurityMiddleware', ...]` | Order matters, processes requests globally |
| `ROOT_URLCONF` | Main URL configuration module | `'myproject.urls'` | Points to project's URL routing |
| `TEMPLATES` | Template engine configuration | `[{ 'BACKEND': 'django.template.backends.django.DjangoTemplates', ... }]` | Configures template directories and processors |
| `WSGI_APPLICATION` | WSGI callable for servers | `'myproject.wsgi.application'` | Interface for web servers like Gunicorn |
| `DATABASES` | Database connection settings | `{'default': {'ENGINE': 'django.db.backends.postgresql', ...}}` | Supports multiple databases |
| `AUTH_PASSWORD_VALIDATORS` | Password strength requirements | `[{'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator'}, ...]` | Enforces secure passwords |
| `LANGUAGE_CODE` | Default language | `'en-us'` | For internationalization |
| `TIME_ZONE` | Project time zone | `'UTC'` | UTC recommended, converted for users |
| `USE_I18N` | Enable translation | `True` | Internationalization support |
| `USE_L10N` | Locale-aware formatting | `True` | Dates/numbers per locale |
| `USE_TZ` | Timezone-aware datetimes | `True` | Store in UTC, convert for display |
| `STATIC_URL` | URL prefix for static files | `'/static/'` | CSS, JS, images |
| `STATICFILES_DIRS` | Development static directories | `[BASE_DIR / 'static']` | Additional static file locations |
| `STATIC_ROOT` | Production static files directory | `BASE_DIR / 'staticfiles'` | Used by collectstatic command |
| `MEDIA_URL` | URL prefix for user uploads | `'/media/'` | User-uploaded files |
| `MEDIA_ROOT` | Directory for user uploads | `BASE_DIR / 'media'` | File storage location |
| `DEFAULT_AUTO_FIELD` | Default primary key type | `'django.db.models.BigAutoField'` | For models without explicit id |

### Security Settings Quick Reference

| Setting | Purpose | Example | Notes |
|---------|---------|---------|-------|
| `SECURE_SSL_REDIRECT` | Force HTTPS | `True` | Redirect HTTP to HTTPS |
| `SESSION_COOKIE_SECURE` | Secure session cookies | `True` | Only send over HTTPS |
| `CSRF_COOKIE_SECURE` | Secure CSRF cookies | `True` | Only send over HTTPS |
| `SECURE_HSTS_SECONDS` | HTTP Strict Transport Security | `31536000` | Force HTTPS for specified time |
| `SECURE_BROWSER_XSS_FILTER` | Enable XSS filter | `True` | Browser XSS protection |
| `SECURE_CONTENT_TYPE_NOSNIFF` | Prevent MIME sniffing | `True` | Set X-Content-Type-Options header |

### Default INSTALLED_APPS

| App | Purpose |
|-----|---------|
| `django.contrib.admin` | Admin interface |
| `django.contrib.auth` | User authentication |
| `django.contrib.contenttypes` | Content type framework |
| `django.contrib.sessions` | Session management |
| `django.contrib.messages` | Flash messages |
| `django.contrib.staticfiles` | Static file handling |

### Default MIDDLEWARE Stack

| Middleware | Purpose |
|------------|---------|
| `SecurityMiddleware` | Security headers, HTTPS redirect |
| `SessionMiddleware` | Session cookie management |
| `CommonMiddleware` | URL normalization, content-length |
| `CsrfViewMiddleware` | CSRF protection |
| `AuthenticationMiddleware` | Attach user to request |
| `MessageMiddleware` | Flash message support |
| `XFrameOptionsMiddleware` | Clickjacking protection |

