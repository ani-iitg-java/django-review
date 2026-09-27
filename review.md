# Django Backend Review — Technical Paper

## 1. Django `settings.py`

### 1.1 What is `settings.py`?

`settings.py` is the central configuration module of a Django project.

It defines how Django should behave, including:

* Security configuration
* Database configuration
* Installed applications
* Middleware
* Templates
* Static files
* Internationalization
* Time zone
* Authentication
* WSGI/ASGI application configuration
* Logging
* Password validation
* Session configuration

A typical Django project contains:

```text
project/
├── manage.py
└── project/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

Django loads the settings module when the application starts.

The environment variable:

```text
DJANGO_SETTINGS_MODULE
```

normally tells Django which settings module to load.

For example:

```text
DJANGO_SETTINGS_MODULE=ipl.settings
```

means Django will import:

```python
ipl/settings.py
```

---

# 2. `SECRET_KEY`

## 2.1 What is `SECRET_KEY`?

`SECRET_KEY` is a secret cryptographic value used by Django for security-related operations.

Example:

```python
SECRET_KEY = 'django-insecure-...'
```

It should be:

* Random
* Long
* Unpredictable
* Kept secret
* Different between environments when appropriate

It should NOT be committed publicly to GitHub.

A production application commonly reads it from an environment variable:

```python
import os

SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]
```

or through a secret-management system.

---

## 2.2 Why does Django need a secret key?

Django uses the secret key as part of several cryptographic mechanisms.

It is associated with functionality such as:

* Cryptographic signing
* Password-reset related tokens
* Session-related security mechanisms
* CSRF-related mechanisms
* Other security-sensitive signing operations

The important interview answer is:

> `SECRET_KEY` is a cryptographically strong secret used by Django for signing and other security mechanisms. If an attacker obtains it, security-sensitive signed data may become vulnerable.

Therefore:

```python
SECRET_KEY = "my-secret-key"
```

is unsafe in production.

---

## 2.3 What happens if `SECRET_KEY` is leaked?

A leaked secret key is a serious security issue.

An attacker who knows it may be able to compromise mechanisms that depend on Django's cryptographic signing.

Therefore:

```text
Never commit production SECRET_KEY to Git.
```

Instead:

```text
Environment variables
        ↓
Django settings
        ↓
SECRET_KEY
```

---

# 3. `INSTALLED_APPS`

`INSTALLED_APPS` tells Django which applications are active in the project.

Example:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'entity',
]
```

The first six are Django's commonly generated built-in applications.

---

## 3.1 Default Django applications

### 1. `django.contrib.admin`

Provides Django's administration interface.

It allows developers to manage database objects through a browser.

For example:

```text
/admin/
```

can be used to manage models registered with the admin site.

---

### 2. `django.contrib.auth`

Provides authentication and authorization functionality.

It includes concepts such as:

* Users
* Passwords
* Groups
* Permissions
* Login
* Logout
* Authentication

Example:

```python
from django.contrib.auth.models import User
```

---

### 3. `django.contrib.contenttypes`

Provides the ContentTypes framework.

It allows Django to keep track of model types dynamically.

It is also used by functionality such as:

* Generic relationships
* Permissions

---

### 4. `django.contrib.sessions`

Provides session support.

A session allows Django to associate data with a particular user across multiple requests.

Example:

```text
Request 1
    ↓
Login
    ↓
Session created
    ↓
Request 2
    ↓
Django identifies user's session
```

---

### 5. `django.contrib.messages`

Provides temporary messages between requests.

Example:

```python
from django.contrib import messages

messages.success(request, "Data saved successfully.")
```

The next page can display:

```text
Data saved successfully.
```

---

### 6. `django.contrib.staticfiles`

Helps manage static assets such as:

```text
CSS
JavaScript
Images
```

Example:

```python
STATIC_URL = 'static/'
```

---

## 3.2 Are there more built-in Django applications?

Yes.

The six applications above are the standard ones included in a newly generated project, but Django provides many additional reusable applications and frameworks.

Examples include:

```text
django.contrib.sites
django.contrib.sitemaps
django.contrib.postgres
django.contrib.humanize
django.contrib.gis
```

Applications can also come from:

```text
Third-party packages
        +
Your own applications
```

For example:

```python
INSTALLED_APPS = [
    ...
    'entity',
    'rest_framework',
]
```

The important concept is:

> `INSTALLED_APPS` defines the collection of Django applications that participate in the project.

---

# 4. Middleware

## 4.1 What is middleware?

Middleware is a layer between the web server/request and the Django view.

It can process:

```text
Request
   ↓
Middleware
   ↓
Middleware
   ↓
View
   ↓
Middleware
   ↓
Middleware
   ↓
Response
```

Middleware can inspect or modify:

* HTTP requests
* HTTP responses
* Sessions
* Authentication
* Security headers
* CSRF tokens
* Exceptions

---

## 4.2 Why do we need middleware?

Suppose every request needs authentication information.

Instead of writing:

```python
def view(request):
    authenticate_user()
    ...
```

inside every view, Django can use:

```python
AuthenticationMiddleware
```

Then the request gets:

```python
request.user
```

This is a cross-cutting concern.

Middleware is useful for functionality that applies across many or all requests.

---

# 5. Django Middleware Execution

Suppose we have:

```python
MIDDLEWARE = [
    "MiddlewareA",
    "MiddlewareB",
]
```

Conceptually, a request travels down:

```text
Request
   ↓
Middleware A
   ↓
Middleware B
   ↓
View
   ↓
Middleware B
   ↓
Middleware A
   ↓
Response
```

This is sometimes described as an onion-like structure.

Middleware can therefore execute logic:

```text
before view
after view
```

and can sometimes stop the request before the view is executed.

---

# 6. Important Django Middleware

A standard Django project contains middleware similar to:

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

Now understand each one.

---

# 7. `SecurityMiddleware`

```python
django.middleware.security.SecurityMiddleware
```

Provides several security-related protections.

Depending on configuration, it can handle things such as:

```text
HTTPS redirects
HSTS
Content-Type sniffing protection
Referrer Policy
COOP
```

For example:

```python
SECURE_SSL_REDIRECT = True
```

can redirect HTTP requests to HTTPS.

HSTS can be configured with:

```python
SECURE_HSTS_SECONDS
```

The basic security principle is:

```text
HTTP
 ↓
HTTPS
 ↓
Encrypted communication
```

---

# 8. `SessionMiddleware`

```python
django.contrib.sessions.middleware.SessionMiddleware
```

Enables session support.

It makes:

```python
request.session
```

available.

Example:

```python
request.session["username"] = "Aniket"
```

Later:

```python
username = request.session["username"]
```

Sessions allow information to persist across requests.

---

# 9. `CommonMiddleware`

```python
django.middleware.common.CommonMiddleware
```

Provides common HTTP-related functionality.

Examples include URL normalization involving:

```text
APPEND_SLASH
PREPEND_WWW
```

It also handles some request/response normalization and user-agent restrictions.

Example:

```python
APPEND_SLASH = True
```

If the application has:

```text
/users/
```

and the client requests:

```text
/users
```

Django can redirect appropriately.

---

# 10. `CsrfViewMiddleware`

```python
django.middleware.csrf.CsrfViewMiddleware
```

Provides CSRF protection.

This is one of the most important security middleware components.

---

# 11. CSRF

## 11.1 What is CSRF?

CSRF means:

> Cross-Site Request Forgery

The attacker tricks a user's browser into making an unwanted request to a website where the user is already authenticated.

Suppose the victim is logged into:

```text
bank.example.com
```

The victim has a valid authentication cookie.

The attacker creates a malicious page:

```html
<form action="https://bank.example.com/transfer" method="POST">
    ...
</form>
```

If the browser automatically sends the victim's authentication credentials, the server might believe:

```text
The legitimate user requested the transfer.
```

The problem is:

```text
Browser automatically sends credentials
        +
Attacker tricks browser into making request
        =
CSRF
```

---

# 12. How Django prevents CSRF

Django uses a CSRF token.

A form typically contains:

```html
<form method="POST">
    {% csrf_token %}
    ...
</form>
```

Django generates a token associated with the user's CSRF secret.

When the request is submitted:

```text
Browser
   ↓
POST request
   ↓
CSRF token
   ↓
CsrfViewMiddleware
   ↓
Validate token
   ↓
View
```

If validation fails:

```text
403 Forbidden
```

is normally returned.

---

## 12.1 Why can't an attacker simply provide the CSRF token?

The important security property is that another origin should not be able to read the legitimate site's CSRF secret/token.

Therefore the attacker may know:

```text
POST /transfer
```

but should not know the valid CSRF token.

---

## 12.2 Which HTTP methods are normally checked?

CSRF protection focuses on methods that can modify state, such as:

```text
POST
PUT
PATCH
DELETE
```

Safe methods such as:

```text
GET
HEAD
OPTIONS
TRACE
```

are treated differently.

A GET request should not perform a state-changing operation.

Bad design:

```text
GET /delete-user/5
```

Better:

```text
DELETE /users/5
```

or a POST-based deletion endpoint depending on the application architecture.

---

# 13. XSS

## 13.1 What is XSS?

XSS means:

> Cross-Site Scripting

It occurs when an attacker manages to inject malicious JavaScript into content that is later executed by another user's browser.

Example malicious input:

```html
<script>
    alert("Hacked");
</script>
```

Suppose an application stores this value:

```text
username = "<script>...</script>"
```

and later directly inserts it into HTML.

The browser may interpret it as JavaScript.

---

# 14. Types of XSS

The major conceptual categories are:

### Stored XSS

Malicious script is stored on the server.

```text
Attacker
   ↓
Database
   ↓
Victim requests page
   ↓
Malicious script returned
   ↓
Browser executes it
```

### Reflected XSS

The malicious payload is immediately reflected in the response.

Example:

```text
/search?q=<script>...</script>
```

### DOM-based XSS

The vulnerability occurs primarily in client-side JavaScript when unsafe data is inserted into the DOM.

---

# 15. Django's XSS protection

Django templates automatically escape many dangerous characters.

For example:

```django
{{ username }}
```

is normally escaped.

An input such as:

```html
<script>alert(1)</script>
```

is rendered as text rather than executable HTML.

However, Django cannot automatically protect every situation.

Developers must be careful with:

```django
{% autoescape off %}
```

and:

```django
{{ value|safe }}
```

The `safe` filter tells Django that content should not be escaped.

Therefore:

```text
safe + untrusted input
=
potential XSS vulnerability
```

---

# 16. Clickjacking

## 16.1 What is clickjacking?

Clickjacking occurs when an attacker embeds another website inside a frame and tricks the user into clicking something different from what they believe they are clicking.

Conceptually:

```text
Attacker Website
┌─────────────────────────────┐
│ Fake button                 │
│                             │
│   Hidden/transparent iframe │
│   ┌─────────────────────┐   │
│   │ Victim Website       │   │
│   │ Delete / Transfer    │   │
│   └─────────────────────┘   │
└─────────────────────────────┘
```

The victim thinks they are clicking:

```text
"Win Prize"
```

but the click may actually activate:

```text
"Delete Account"
```

---

# 17. Django clickjacking protection

Django provides:

```python
XFrameOptionsMiddleware
```

which can add:

```text
X-Frame-Options
```

to responses.

For example:

```text
X-Frame-Options: DENY
```

means the page should not be rendered inside a frame.

This protects against many clickjacking attacks.

---

# 18. Other important middleware

## `AuthenticationMiddleware`

Adds:

```python
request.user
```

to the request.

It relies on session middleware.

---

## `MessageMiddleware`

Provides the messages framework.

Example:

```python
messages.success(request, "Saved!")
```

---

## `GZipMiddleware`

Compresses responses using gzip when appropriate.

This reduces response size:

```text
Large response
     ↓
GZip
     ↓
Smaller response
     ↓
Client
```

---

## `LocaleMiddleware`

Provides localization support.

It can determine language preferences and activate the appropriate language.

---

## `ConditionalGetMiddleware`

Supports conditional HTTP requests involving things such as:

```text
ETag
Last-Modified
```

This can allow clients to avoid downloading unchanged content.

---

## Cache middleware

Django also provides:

```python
UpdateCacheMiddleware
FetchFromCacheMiddleware
```

These support site-wide caching.

---

# 19. Middleware Ordering

Middleware ordering matters.

For example:

```text
SessionMiddleware
        ↓
AuthenticationMiddleware
```

Authentication middleware needs session information.

Therefore:

```python
SessionMiddleware
```

must occur before:

```python
AuthenticationMiddleware
```

Similarly, CSRF and authentication-related middleware have ordering considerations.

Interview answer:

> Middleware is not simply a list of independent components. Middleware can depend on request/response state produced by middleware earlier in the chain, so ordering matters.

---

# 20. WSGI

## 20.1 What does WSGI mean?

WSGI means:

> Web Server Gateway Interface

It is a Python standard/interface that defines communication between:

```text
Web Server
     ↓
WSGI
     ↓
Python Web Application
```

Django provides a WSGI application object.

Typical:

```python
# wsgi.py

import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault(
    'DJANGO_SETTINGS_MODULE',
    'project.settings'
)

application = get_wsgi_application()
```

---

# 21. Why do we need WSGI?

A web server such as:

```text
Nginx
Apache
Gunicorn
```

needs a standardized way to communicate with a Python web application.

WSGI provides that interface.

Conceptually:

```text
Client
  ↓
Nginx
  ↓
Gunicorn
  ↓
WSGI
  ↓
Django
  ↓
URL
  ↓
View
  ↓
Response
```

The important point is:

> WSGI is an interface/standard, not the Django framework itself.

---

# 22. WSGI vs ASGI

WSGI is primarily synchronous.

ASGI was introduced as an asynchronous-capable successor/interface.

Conceptually:

```text
WSGI
 ↓
Traditional synchronous Python web applications
```

while:

```text
ASGI
 ↓
Synchronous + asynchronous applications
 ↓
WebSockets
 ↓
Long-lived connections
```

Modern Django supports both WSGI and ASGI.

For an interview:

> WSGI is the traditional Python web application interface for synchronous request/response applications, while ASGI supports asynchronous communication and protocols such as WebSockets.

---

# 23. Django Models

A model represents application data and its database structure.

Example:

```python
from django.db import models

class Student(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()
    email = models.EmailField()
```

Django can map this model to a database table.

Conceptually:

```text
Python Model
     ↓
Django ORM
     ↓
Database Table
```

---

# 24. `on_delete=models.CASCADE`

Suppose:

```python
class Department(models.Model):
    name = models.CharField(max_length=100)


class Employee(models.Model):
    name = models.CharField(max_length=100)

    department = models.ForeignKey(
        Department,
        on_delete=models.CASCADE
    )
```

Relationship:

```text
Department
    |
    | 1
    |
    | *
Employee
```

If:

```text
Department A
```

is deleted, Django's CASCADE behavior also deletes related:

```text
Employee
Employee
Employee
```

---

# 25. Why is CASCADE dangerous?

Suppose:

```text
Company
  ↓
Department
  ↓
Employee
```

If relationships use CASCADE, deleting the company could cause a chain of deletions.

Therefore you should choose `on_delete` according to business requirements.

---

# 26. Other `on_delete` options

### `CASCADE`

Delete related objects.

```python
on_delete=models.CASCADE
```

### `PROTECT`

Prevent deletion.

```python
on_delete=models.PROTECT
```

Raises a protection-related exception if dependent objects exist.

### `RESTRICT`

Prevents deletion under restricted relationship conditions.

```python
on_delete=models.RESTRICT
```

### `SET_NULL`

Set the foreign key to `NULL`.

Requires:

```python
null=True
```

Example:

```python
user = models.ForeignKey(
    User,
    null=True,
    on_delete=models.SET_NULL
)
```

### `SET_DEFAULT`

Set the foreign key to its default value.

### `SET(...)`

Set the field to a specified value or callable.

### `DO_NOTHING`

Django takes no deletion action.

This can lead to database integrity errors if the database enforces referential integrity.

---

# 27. Django Model Fields

A model field represents a database-backed attribute.

Example:

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2
    )
    quantity = models.IntegerField()
    active = models.BooleanField(default=True)
```

Common fields include:

```text
CharField
TextField
IntegerField
PositiveIntegerField
FloatField
DecimalField
BooleanField
DateField
DateTimeField
EmailField
URLField
UUIDField
FileField
ImageField
JSONField
ForeignKey
OneToOneField
ManyToManyField
```

---

# 28. `CharField` vs `TextField`

### `CharField`

Usually used for bounded-length strings.

```python
name = models.CharField(max_length=100)
```

### `TextField`

Used for larger amounts of text.

```python
description = models.TextField()
```

---

# 29. `IntegerField`

Used for integer values.

```python
age = models.IntegerField()
```

---

# 30. `DecimalField`

Useful for exact decimal values.

```python
price = models.DecimalField(
    max_digits=10,
    decimal_places=2
)
```

For financial values, `DecimalField` is generally preferable to floating-point representation.

---

# 31. `DateField` vs `DateTimeField`

```python
birth_date = models.DateField()
```

stores a date.

```python
created_at = models.DateTimeField()
```

stores date and time.

Common configuration:

```python
created_at = models.DateTimeField(auto_now_add=True)
updated_at = models.DateTimeField(auto_now=True)
```

`auto_now_add` is typically used when an object is initially created.

`auto_now` updates the field when the model is saved.

---

# 32. `null=True` vs `blank=True`

This is a common interview question.

### `null=True`

Primarily relates to database storage.

```python
age = models.IntegerField(null=True)
```

means the database column may contain NULL.

### `blank=True`

Primarily relates to validation/forms.

```python
name = models.CharField(
    max_length=100,
    blank=True
)
```

means the field can be blank during validation.

Therefore:

```text
null → database
blank → validation
```

They are not synonyms.

---

# 33. Model Validators

Validators check whether a value satisfies a condition.

Example:

```python
from django.core.validators import MinValueValidator

age = models.IntegerField(
    validators=[
        MinValueValidator(18)
    ]
)
```

Now:

```text
age = 15
```

fails validation.

---

# 34. Common Django Validators

Django provides validators such as:

```text
MinValueValidator
MaxValueValidator
MinLengthValidator
MaxLengthValidator
RegexValidator
EmailValidator
URLValidator
DecimalValidator
validate_email
validate_ipv4_address
validate_ipv6_address
validate_comma_separated_integer_list
```

Example:

```python
from django.core.validators import MaxValueValidator

score = models.IntegerField(
    validators=[
        MaxValueValidator(100)
    ]
)
```

---

# 35. Validator vs Database Constraint

This distinction is important.

A validator generally participates in Django-side validation.

Example:

```python
MinValueValidator(18)
```

is not equivalent to a database constraint.

For database-level guarantees, Django also provides constraints such as:

```python
UniqueConstraint
CheckConstraint
```

Therefore:

```text
Validation
    ≠
Database constraint
```

For critical data integrity, database constraints are important.

---

# 36. Python Module vs Python Class

This is a basic Python concept frequently asked in Django interviews.

## Module

A Python module is generally a `.py` file containing Python definitions.

Example:

```text
models.py
```

is a module.

It can contain:

```python
class Student:
    pass

def calculate():
    pass

PI = 3.14
```

---

## Class

A class is a blueprint for creating objects.

Example:

```python
class Student:
    pass
```

`Student` is a class.

We can create an object:

```python
student = Student()
```

---

## Difference

```text
Python module
    ↓
Usually a .py file
    ↓
Contains classes/functions/variables
```

Whereas:

```text
Python class
    ↓
Blueprint for objects
```

In Django:

```text
models.py
```

is a module.

Inside it:

```python
class Match(models.Model):
    ...
```

`Match` is a class.

---

# 37. Django ORM

ORM means:

> Object Relational Mapping

Django ORM allows Python code to interact with relational databases without manually writing SQL for most common operations.

Instead of:

```sql
SELECT * FROM matches;
```

you can write:

```python
Matches.objects.all()
```

Conceptually:

```text
Python code
     ↓
Django ORM
     ↓
SQL
     ↓
Database
```

---

# 38. Django Shell

The Django shell is extremely useful for learning and debugging ORM queries.

Run:

```bash
python manage.py shell
```

Then import a model:

```python
from entity.models import Matches
```

Now:

```python
Matches.objects.all()
```

returns a QuerySet.

---

# 39. Basic ORM Queries

## Get all records

```python
Matches.objects.all()
```

---

## Filter

```python
Matches.objects.filter(
    season=2015
)
```

Equivalent conceptually to:

```sql
SELECT *
FROM matches
WHERE season = 2015;
```

---

## Get one object

```python
Matches.objects.get(id=1)
```

`get()` expects exactly one matching object.

If no object exists:

```text
DoesNotExist
```

If multiple objects match:

```text
MultipleObjectsReturned
```

---

## Exclude

```python
Matches.objects.exclude(
    season=2015
)
```

---

## Ordering

```python
Matches.objects.order_by("season")
```

Descending:

```python
Matches.objects.order_by("-season")
```

---

# 40. QuerySet

A QuerySet represents a collection of database queries/results.

Example:

```python
queryset = Matches.objects.filter(season=2015)
```

Important property:

> QuerySets are generally lazy.

This means constructing:

```python
Matches.objects.filter(season=2015)
```

does not necessarily immediately execute the database query.

The query is evaluated when the results are needed.

Examples of evaluation include:

```python
list(queryset)
```

```python
for match in queryset:
    ...
```

```python
len(queryset)
```

and other operations.

---

# 41. How to see SQL generated by ORM

In Django shell:

```python
queryset = Matches.objects.filter(season=2015)
```

Then:

```python
print(queryset.query)
```

This displays the SQL generated by the QuerySet.

Example conceptually:

```sql
SELECT ...
FROM ...
WHERE season = 2015;
```

This is extremely useful for understanding what Django ORM is actually doing.

---

# 42. `connection.queries`

You can also inspect executed queries during a shell session:

```python
from django.db import connection

connection.queries
```

This can show executed SQL queries and their timing.

However, query logging/debugging configuration matters, and this should not be treated as a production query-monitoring strategy.

---

# 43. ORM Lookups

Django provides field lookups.

Example:

```python
Matches.objects.filter(
    season__gte=2015
)
```

means:

```text
season >= 2015
```

Other examples:

```python
season__gt=2015
season__lt=2015
season__lte=2015
name__exact="Mumbai"
name__iexact="mumbai"
name__contains="mum"
name__icontains="mum"
```

---

# 44. Aggregations

Aggregation calculates a value across multiple database rows.

Examples:

```text
COUNT
SUM
AVG
MIN
MAX
```

Django provides:

```python
from django.db.models import Count, Sum, Avg, Min, Max
```

---

# 45. Example of Aggregation

Suppose:

```text
Sales
-----
100
200
300
```

We want total sales.

```python
from django.db.models import Sum

Sales.objects.aggregate(
    total=Sum("amount")
)
```

Result conceptually:

```python
{
    "total": 600
}
```

Aggregation produces a summary value.

---

# 46. `aggregate()` vs `annotate()`

This is one of the most important Django ORM interview questions.

### `aggregate()`

Produces a summary for the entire QuerySet.

Example:

```python
Player.objects.aggregate(
    total=Count("id")
)
```

Result:

```python
{
    "total": 100
}
```

Think:

```text
Many rows
   ↓
One summary result
```

---

# 47. `annotate()`

`annotate()` adds calculated information to each result/group.

Example:

```python
from django.db.models import Count

Team.objects.annotate(
    player_count=Count("players")
)
```

Conceptually:

```text
Team A → player_count = 11
Team B → player_count = 14
Team C → player_count = 12
```

Each Team object gets an additional calculated attribute:

```python
team.player_count
```

Think:

```text
Rows/groups
    ↓
Each gets calculated information
```

---

# 48. Aggregation vs Annotation

A useful mental model:

```text
aggregate()
    ↓
Whole QuerySet
    ↓
Summary

annotate()
    ↓
Each object/group
    ↓
Additional calculated field
```

Example:

```python
Player.objects.aggregate(
    avg_age=Avg("age")
)
```

returns:

```text
One average
```

Whereas:

```python
Team.objects.annotate(
    avg_age=Avg("players__age")
)
```

can give:

```text
Team A → average age
Team B → average age
Team C → average age
```

---

# 49. SQL Equivalent of `annotate()`

Suppose:

```python
Team.objects.annotate(
    player_count=Count("players")
)
```

The generated SQL is conceptually similar to:

```sql
SELECT
    team.id,
    team.name,
    COUNT(player.id)
FROM team
LEFT JOIN player
    ON player.team_id = team.id
GROUP BY team.id, team.name;
```

This is why understanding SQL helps when learning Django ORM.

---

# 50. `values()` with Aggregation

You may want to group by a field.

Example:

```python
Matches.objects.values(
    "season"
).annotate(
    total=Count("id")
)
```

Conceptually:

```text
2015 → 59
2016 → 60
2017 → 59
```

Equivalent SQL concept:

```sql
SELECT season, COUNT(id)
FROM matches
GROUP BY season;
```

---

# 51. Why are Aggregations Important?

Aggregations are useful for:

* Reports
* Dashboards
* Analytics
* Statistics
* Charts
* Business intelligence
* Ranking
* Counting records
* Financial calculations

For example, your IPL project can use:

```python
Matches.objects.values(
    "season"
).annotate(
    total_matches=Count("id")
)
```

to generate data for a chart.

---

# 52. Migrations

## What is a migration?

A migration is Django's mechanism for representing database schema changes as Python files.

Suppose the model initially contains:

```python
class Student(models.Model):
    name = models.CharField(max_length=100)
```

Later we add:

```python
age = models.IntegerField()
```

The model changed.

Django needs to change the database table too.

Migration system:

```text
models.py changes
       ↓
makemigrations
       ↓
Migration file
       ↓
migrate
       ↓
Database schema changes
```

---

# 53. `makemigrations`

Run:

```bash
python manage.py makemigrations
```

Django detects model changes and generates migration files.

Example:

```text
0001_initial.py
0002_student_age.py
```

A migration is a normal Python file containing migration operations.

---

# 54. `migrate`

Run:

```bash
python manage.py migrate
```

This applies migrations to the database.

Conceptually:

```text
Migration file
      ↓
Django migration engine
      ↓
SQL
      ↓
Database
```

---

# 55. Why are migration files needed?

Imagine five developers working on the same project.

Without migrations:

```text
Developer A changes database
Developer B changes database
Developer C changes database
```

It becomes difficult to reproduce the exact schema.

With migrations:

```text
0001_initial
0002_add_age
0003_add_email
0004_add_index
```

the schema evolution is tracked.

Migrations therefore provide:

* Schema versioning
* Reproducibility
* Collaboration
* Dependency management
* Deployment support
* Historical database changes

---

# 56. Migration Files and Git

Migration files should normally be committed to version control.

Example:

```text
entity/
└── migrations/
    ├── __init__.py
    ├── 0001_initial.py
    ├── 0002_add_age.py
    └── 0003_add_email.py
```

Another developer can then run:

```bash
python manage.py migrate
```

and bring their database schema up to date.

---

# 57. Migration Dependencies

Migration files can depend on earlier migrations.

Example:

```python
dependencies = [
    ("entity", "0001_initial"),
]
```

This creates a migration graph.

Conceptually:

```text
0001
 ↓
0002
 ↓
0003
 ↓
0004
```

Django determines the correct order based on dependencies.

---

# 58. SQL Transactions

Transactions are a database concept, not specifically a Django ORM concept.

A transaction is a group of database operations treated as one logical unit.

Example:

```text
Transfer ₹1000
-------------------------
1. Debit Account A
2. Credit Account B
```

We don't want:

```text
Debit succeeds
Credit fails
```

because money would disappear.

Instead:

```text
BEGIN
   Debit A
   Credit B
COMMIT
```

If something fails:

```text
BEGIN
   Debit A
   Credit B
ROLLBACK
```

The database returns to the previous consistent state.

---

# 59. ACID

Transactions are commonly explained using ACID.

## Atomicity

All operations happen or none happen.

```text
All
or
Nothing
```

---

## Consistency

A transaction moves the database from one valid state to another valid state.

---

## Isolation

Concurrent transactions should not incorrectly interfere with each other.

For example:

```text
Transaction A
Transaction B
```

should observe data according to the database's isolation rules.

---

## Durability

Once a transaction is committed, its changes should survive failures according to the database's durability guarantees.

---

# 60. Django `transaction.atomic()`

Django provides:

```python
from django.db import transaction
```

Then:

```python
with transaction.atomic():
    account.balance -= 1000
    account.save()

    account2.balance += 1000
    account2.save()
```

The operations inside the block are treated as one transaction.

If an exception occurs:

```python
with transaction.atomic():
    ...
    raise Exception("Something failed")
```

the transaction is rolled back.

---

# 61. `atomic()` Example

Suppose we have:

```python
@transaction.atomic
def transfer_money(source, destination, amount):

    source.balance -= amount
    source.save()

    destination.balance += amount
    destination.save()
```

The decorator:

```python
@transaction.atomic
```

means Django executes the function inside an atomic transaction.

Conceptually:

```text
BEGIN
   ↓
Debit source
   ↓
Credit destination
   ↓
COMMIT
```

If an error occurs:

```text
BEGIN
   ↓
Debit source
   ↓
ERROR
   ↓
ROLLBACK
```

---

# 62. `atomic()` vs Database Transaction

An important interview distinction:

```text
Database transaction
```

is the underlying database concept.

```python
transaction.atomic()
```

is Django's API for controlling transactions.

So Django does not invent transactions.

It provides a convenient interface over the database transaction mechanism.

---

# 63. Nested `atomic()` Blocks

Django allows nested atomic blocks.

Example:

```python
with transaction.atomic():

    operation_1()

    with transaction.atomic():
        operation_2()

    operation_3()
```

Django uses transaction management mechanisms such as savepoints for nested atomic blocks when supported.

Conceptually:

```text
Outer Transaction
│
├── operation_1
│
├── Savepoint
│    └── operation_2
│
└── operation_3
```

This is useful when part of a larger transaction needs its own rollback boundary.

---

# 64. Transaction Isolation

Transaction isolation determines what one transaction can observe about another concurrent transaction.

Common SQL isolation levels include:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

The exact behavior depends on the database.

For example, PostgreSQL commonly uses:

```text
READ COMMITTED
```

as its default isolation level.

---

# 65. Transaction vs Locking

Transactions do not automatically solve every concurrency problem.

Consider:

```text
Account balance = 100
```

Two requests simultaneously execute:

```text
Request A reads 100
Request B reads 100
```

Both subtract 80.

Both may attempt to save:

```text
20
```

This is a concurrency problem.

Depending on the use case, techniques such as:

```python
select_for_update()
```

can be used inside an appropriate transaction.

Example:

```python
with transaction.atomic():
    account = Account.objects.select_for_update().get(
        id=account_id
    )

    account.balance -= amount
    account.save()
```

This can lock the selected database row according to the database's locking semantics.

---

# 66. Complete Django ORM Flow

The overall flow can be understood as:

```text
Python Model
     ↓
Django ORM
     ↓
QuerySet
     ↓
SQL Compiler
     ↓
SQL Query
     ↓
Database Driver
     ↓
Database
```

For example:

```python
Matches.objects.filter(
    season=2015
)
```

conceptually becomes:

```sql
SELECT *
FROM matches
WHERE season = 2015;
```

The developer generally works with Python objects rather than manually constructing SQL.

---

# 67. Complete Django Request Flow

A useful interview-level architecture is:

```text
Client / Browser
       ↓
Web Server
       ↓
WSGI / ASGI
       ↓
Django
       ↓
Middleware
       ↓
URL Resolver
       ↓
View
       ↓
Django ORM
       ↓
Database
       ↓
ORM Result
       ↓
View
       ↓
Response
       ↓
Middleware
       ↓
WSGI / ASGI
       ↓
Web Server
       ↓
Client
```

For a POST request:

```text
POST
 ↓
SecurityMiddleware
 ↓
SessionMiddleware
 ↓
CommonMiddleware
 ↓
CSRF Middleware
 ↓
AuthenticationMiddleware
 ↓
URL Resolver
 ↓
View
 ↓
ORM
 ↓
Database
```

---

# 68. Security Summary

For Django security, remember these major areas:

```text
SECRET_KEY
     ↓
Cryptographic security

CSRF
     ↓
Prevents forged state-changing requests

XSS
     ↓
Prevents malicious scripts from executing

Clickjacking
     ↓
X-Frame-Options / framing protection

SQL Injection
     ↓
Parameterized ORM queries

HTTPS
     ↓
Encrypts communication

HSTS
     ↓
Encourages browsers to use HTTPS

ALLOWED_HOSTS
     ↓
Host-header validation

Secure Cookies
     ↓
Protect cookies over HTTPS

Input Validation
     ↓
Reject invalid/untrusted data
```

---

# 69. Interview Questions You Should Be Able to Answer

## Settings

### Q1. What is `settings.py`?

It is Django's central configuration module containing settings for applications, middleware, database, security, templates, static files, authentication, and other project behavior.

### Q2. What is `SECRET_KEY`?

It is a secret cryptographic value used by Django for security-sensitive signing and related cryptographic operations.

### Q3. Should `SECRET_KEY` be committed to Git?

No. Production secrets should be stored securely outside source control.

### Q4. What is `INSTALLED_APPS`?

It is the list of Django applications enabled in the project.

---

## Middleware

### Q5. What is middleware?

Middleware is a processing layer that can inspect and modify requests and responses before or after the view executes.

### Q6. Why does middleware ordering matter?

Because middleware can depend on state produced by other middleware.

### Q7. What does `AuthenticationMiddleware` do?

It associates the authenticated user with the request through:

```python
request.user
```

### Q8. What does `SessionMiddleware` do?

It enables session handling through:

```python
request.session
```

### Q9. What does `CsrfViewMiddleware` do?

It validates CSRF protection for unsafe requests.

### Q10. What does `XFrameOptionsMiddleware` do?

It provides clickjacking protection using the `X-Frame-Options` response header.

---

## Security

### Q11. What is CSRF?

An attack where an attacker tricks an authenticated user's browser into making an unwanted state-changing request.

### Q12. What is XSS?

An attack where malicious client-side script is injected into content and executed in another user's browser.

### Q13. What is clickjacking?

An attack where a malicious site frames another site and tricks a user into interacting with the framed site unintentionally.

### Q14. How does Django protect against SQL injection?

Django's ORM generally uses parameterized queries so user-provided values are handled as query parameters rather than executable SQL.

---

## Models

### Q15. What is a Django model?

A Python class representing application data and its database structure.

### Q16. What does `on_delete=models.CASCADE` mean?

When the referenced object is deleted, related objects using that relationship are also deleted according to Django's cascade behavior.

### Q17. Difference between `null=True` and `blank=True`?

```text
null → database
blank → validation
```

### Q18. What is a validator?

A callable that checks whether a value satisfies a specified validation rule.

---

## Python

### Q19. What is a module?

A Python module is generally a `.py` file containing Python definitions.

### Q20. What is a class?

A class is a blueprint from which objects can be created.

---

## ORM

### Q21. What is ORM?

Object Relational Mapping allows application code to interact with relational database data using programming-language objects and APIs.

### Q22. What is a QuerySet?

A QuerySet represents a collection of database query operations/results and is generally evaluated lazily.

### Q23. How do you see SQL generated by a QuerySet?

```python
print(queryset.query)
```

### Q24. What is `aggregate()`?

It calculates a summary over a QuerySet.

```python
Player.objects.aggregate(
    total=Count("id")
)
```

### Q25. What is `annotate()`?

It adds calculated values to individual query results/groups.

```python
Team.objects.annotate(
    player_count=Count("players")
)
```

### Q26. Difference between `aggregate()` and `annotate()`?

```text
aggregate()
    → summary result

annotate()
    → calculated value attached to each result/group
```

---

## Migrations

### Q27. What is a migration?

A version-controlled Python representation of database schema changes.

### Q28. Difference between `makemigrations` and `migrate`?

```text
makemigrations
    → creates migration files

migrate
    → applies migrations to the database
```

### Q29. Why commit migrations to Git?

So other developers and deployment environments can reproduce the database schema evolution.

---

## Transactions

### Q30. What is a database transaction?

A group of database operations treated as one logical unit.

### Q31. What is ACID?

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

### Q32. What is `transaction.atomic()`?

Django's API for executing database operations within an atomic transaction.

### Q33. What happens if an exception occurs inside `atomic()`?

The transaction is rolled back according to the transaction/savepoint boundary involved.

---

# 70. One-Minute Interview Explanation

If an interviewer asks:

> "Explain Django from settings to database."

A strong answer is:

Django is a Python web framework following an MVT architecture. The `settings.py` file contains project configuration such as installed applications, middleware, database configuration and security settings. `INSTALLED_APPS` defines the applications enabled in the project, while middleware forms a processing layer around requests and responses. For example, `SecurityMiddleware` provides security-related protections, `SessionMiddleware` enables sessions, `AuthenticationMiddleware` provides `request.user`, `CsrfViewMiddleware` protects against CSRF, and `XFrameOptionsMiddleware` provides clickjacking protection.

Django models represent database entities as Python classes. The Django ORM allows us to query those models using Python instead of manually writing SQL. A QuerySet represents database query operations and is generally evaluated lazily. We can use `filter()`, `get()`, `exclude()`, `order_by()`, `aggregate()` and `annotate()` to retrieve and analyze data.

When the model structure changes, Django's migration framework records those schema changes in migration files. `makemigrations` creates the migration files and `migrate` applies them to the database.

For operations that must succeed or fail together, Django provides `transaction.atomic()`, which maps to database transaction semantics. This gives us atomicity and helps maintain data consistency.

At deployment time, Django can expose a WSGI application for traditional synchronous Python web servers, or an ASGI application when asynchronous capabilities are required.

# 71. Mental Model to Remember

The entire topic can be reduced to this diagram:

```text
                 DJANGO
                    │
       ┌────────────┴────────────┐
       │                         │
   settings.py                security
       │                         │
       ├── INSTALLED_APPS        ├── CSRF
       ├── DATABASES             ├── XSS
       ├── MIDDLEWARE            ├── Clickjacking
       ├── TEMPLATES             ├── HTTPS
       └── STATIC_FILES          └── SECRET_KEY
                    │
                    ↓
               Middleware
                    │
                    ↓
               URL Resolver
                    │
                    ↓
                  View
                    │
                    ↓
                 ORM
                    │
              ┌─────┴─────┐
              │           │
          QuerySet      Transactions
              │           │
              ↓           ↓
             SQL       atomic()
              │
              ↓
           Database
              │
              ↓
           Response
```

The most important relationships to remember are:

```text
settings.py
    ↓
configures Django

middleware
    ↓
processes requests/responses

models.py
    ↓
defines data structure

ORM
    ↓
converts Python operations into SQL

migrations
    ↓
version-control database schema changes

transactions
    ↓
protect groups of database operations

WSGI/ASGI
    ↓
connect Django application to application servers
```

These relationships are more important for an interview than memorizing isolated definitions.
