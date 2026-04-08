# Django ORM

This document explains Django ORM concepts with examples you can run in the Django shell.

## Using ORM queries in Django Shell

Start the shell in your project:

```bash
python manage.py shell
```

Import a model and query it:

```python
from myapp.models import Book

# Get all records
books = Book.objects.all()

# Filter records
english_books = Book.objects.filter(language='en')

# Get a single record
book = Book.objects.get(pk=1)

# Create a new record
new_book = Book.objects.create(title='Django Guide', author='Alice', price=499)

# Update a record
book.price = 599
book.save()

# Delete a record
book.delete()
```

Common query methods:
- `all()`
- `filter()`
- `exclude()`
- `get()`
- `order_by()`
- `values()`
- `values_list()`
- `distinct()`

Example with chaining:

```python
recent_books = Book.objects.filter(published_year__gte=2020).order_by('-published_year')
```

## Turning ORM to SQL in Django Shell

Django QuerySets are lazy. You can inspect the generated SQL:

```python
from myapp.models import Book

qs = Book.objects.filter(author='Alice').order_by('title')
print(qs.query)
```

Example output:

```sql
SELECT "myapp_book"."id", "myapp_book"."title", ...
FROM "myapp_book"
WHERE "myapp_book"."author" = Alice
ORDER BY "myapp_book"."title" ASC
```

This is useful for debugging and understanding how ORM queries map to SQL.

## What are Aggregations?

Aggregations compute values over a QuerySet.

Import aggregation helpers:

```python
from django.db.models import Count, Sum, Avg, Max, Min
```

Examples:

```python
total_price = Book.objects.aggregate(total=Sum('price'))
average_price = Book.objects.aggregate(avg=Avg('price'))
count_books = Book.objects.aggregate(count=Count('id'))
max_price = Book.objects.aggregate(max_price=Max('price'))
min_price = Book.objects.aggregate(min_price=Min('price'))
```

Result:

```python
{'total': 1497}
```

Use aggregation to summarize data without loading all rows in Python.

## What are Annotations?

Annotations add calculated values to each object in a QuerySet.

Example:

```python
from django.db.models import Count, F

authors = Author.objects.annotate(book_count=Count('book'))
for author in authors:
    print(author.name, author.book_count)
```

Another example with expressions:

```python
from django.db.models import F, ExpressionWrapper, DecimalField

books = Book.objects.annotate(
    discounted_price=ExpressionWrapper(F('price') * 0.9, output_field=DecimalField())
)
```

Difference:
- `aggregate()` returns a summary for the whole QuerySet.
- `annotate()` adds values per row.

## What is a migration file? Why is it needed?

A migration file is a Python file Django generates to describe database schema changes.

Create migrations:

```bash
python manage.py makemigrations
```

Apply migrations:

```bash
python manage.py migrate
```

Example migration file contents:

```python
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [
        ('myapp', '0001_initial'),
    ]

    operations = [
        migrations.CreateModel(
            name='Book',
            fields=[
                ('id', models.BigAutoField(primary_key=True)),
                ('title', models.CharField(max_length=200)),
                ('price', models.IntegerField()),
            ],
        ),
    ]
```

Why needed:
- Keeps database schema in sync with models.
- Tracks changes over time.
- Enables versioned schema changes and rollback.

## What are SQL transactions? (non ORM concept)

A transaction is a unit of work performed against a database.

Key properties (ACID):
- Atomicity: all operations succeed or none.
- Consistency: database moves from one valid state to another.
- Isolation: operations are isolated from other transactions.
- Durability: committed changes persist.

Example SQL transaction:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

If one statement fails, the whole transaction can be rolled back:

```sql
ROLLBACK;
```

Transactions are important for data integrity in multi-step updates.

## What are atomic transactions?

In Django, `transaction.atomic()` ensures a block of code runs inside a database transaction.

Example:

```python
from django.db import transaction
from myapp.models import Account

with transaction.atomic():
    sender = Account.objects.get(pk=1)
    receiver = Account.objects.get(pk=2)
    sender.balance -= 100
    receiver.balance += 100
    sender.save()
    receiver.save()
```

If any error occurs inside the block, Django rolls back the entire transaction.

Decorator style:

```python
from django.db import transaction

@transaction.atomic
def transfer_money(sender_id, receiver_id, amount):
    ...
```

Savepoints allow partial rollback inside a transaction:

```python
with transaction.atomic():
    ...
    sid = transaction.savepoint()
    try:
        ...
    except Exception:
        transaction.savepoint_rollback(sid)
```

Use atomic transactions when multiple database writes must either all succeed or all fail.

## Summary

- Django ORM lets you query models in Python instead of SQL.
- Use the shell for quick tests and debugging.
- Inspect generated SQL with `query`.
- Aggregations summarize data.
- Annotations add computed fields to query results.
- Migration files track schema changes.
- SQL transactions guarantee atomic updates.
- Django `transaction.atomic()` wraps multiple ORM operations safely.