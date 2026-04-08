# Django models concepts

## What is `on_delete=models.CASCADE`?

`on_delete=models.CASCADE` is used with relations like `ForeignKey` and `OneToOneField`.

Example:

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
```

Explanation:
- If an `Author` is deleted, all `Book` records that reference that author are deleted too.
- This keeps the database consistent by removing child records that would otherwise refer to a missing parent.

Real-time example:
- If author "Alice" is removed, books written by Alice are removed automatically.

Other `on_delete` options:
- `models.PROTECT` — prevent deletion if related objects exist
- `models.SET_NULL` — set the foreign key to `NULL`
- `models.SET_DEFAULT` — set the foreign key to a default value
- `models.DO_NOTHING` — do nothing (can cause broken references)

## Fields and Validators

### Model fields

Django model fields define how data is stored in the database.

Common field types:
- `CharField(max_length=100)` — short text
- `TextField()` — long text
- `IntegerField()` — whole numbers
- `DecimalField(max_digits=8, decimal_places=2)` — precise decimal values
- `BooleanField()` — true/false
- `DateTimeField()` — date and time
- `ForeignKey(...)` — many-to-one relationship
- `OneToOneField(...)` — one-to-one relationship
- `ManyToManyField(...)` — many-to-many relationship

Example:

```python
class Product(models.Model):
    name = models.CharField(max_length=150)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    in_stock = models.BooleanField(default=True)
    description = models.TextField(blank=True)
```

Field options:
- `null=True` — allow database NULL
- `blank=True` — allow empty form input
- `default=...` — default value when none is provided
- `choices=[...]` — restrict valid values
- `unique=True` — enforce uniqueness
- `db_index=True` — add a database index

### Validators

Validators check field values before saving or validating a form.

Built-in validators:
- `MinValueValidator(0)`
- `MaxValueValidator(100)`
- `EmailValidator()`
- `RegexValidator(r'^[0-9]+$')`

Example:

```python
from django.core.validators import MinValueValidator, EmailValidator

class Customer(models.Model):
    email = models.EmailField(validators=[EmailValidator()])
    age = models.IntegerField(validators=[MinValueValidator(18)])
```

Custom validator example:

```python
from django.core.exceptions import ValidationError

def validate_even(value):
    if value % 2 != 0:
        raise ValidationError('Value must be even.')

class Sample(models.Model):
    number = models.IntegerField(validators=[validate_even])
```

Validators work at the model layer, so they help keep data consistent no matter where the model is used.

## Understanding the difference between Python module and Python class

### Python module
- A module is a Python file, such as `models.py`.
- It can contain classes, functions, variables, and imports.
- In Django, each app usually has modules like `models.py`, `views.py`, and `urls.py`.

Example:
- `myapp/models.py` is a module.

### Python class
- A class is a blueprint for objects and is defined inside a module.
- In Django, each model is a class that inherits from `models.Model`.

Example:

```python
# module: myapp/models.py

from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
```

Here:
- `myapp.models` is the module
- `Author` is the class inside that module

### Key difference
- Module = file containing code
- Class = type definition inside that file

Real-time usage:
- `import myapp.models` imports the module
- `from myapp.models import Author` imports the class from the module