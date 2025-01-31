# Types

## Output types

Output types are generated from models. `auto` type is used for field type auto resolution. Relational fields are described by referencing to other type genedated from Django model. Many to many relation is described by using List type. `strawberry.django` will automatically generate resolvers for relational fields. More information about that can be read from [resolvers](resolvers.md) page.

```python
from strawberry.django import auto
from typing import List

@strawberry.django.type(models.Fruit)
class Fruit:
    id: auto
    name: auto
    color: 'Color'

@strawberry.django.type(models.Color)
class Color:
    id: auto
    name: auto
    fruits: List[Fruit]
```

## Input types

Input types can be generated from models by using `strawberry.django.input` decorator. First parameter is always model, where the type is converted from.

```python
@strawberry.django.input(models.Fruit)
class FruitInput:
    id: auto
    name: auto
    color: 'ColorInput'
```

Partial input type, which all fields are optional, is generated by adding `partial=True` parameter for `input`. Partial input can be generated from existing input type by using class inheritance.

```python
@strawberry.django.input(models.Color, partial=True)
class FruitPartialInput(FruitInput):
    color: List['ColorPartialInput']

@strawberry.django.input(models.Color, partial=True)
class ColorPartialInput:
    id: auto
    name: auto
    fruits: List[FruitPartialInput]
```

## Django Model Types

Django Models can be converted in Strawberry Types with the `@strawberry_django.type` decorator

```python
import strawberry

@strawberry.django.type(models.Fruit)
class Fruit:
    ...
```

### Fields

By default, no fields are implemented on the new type. For details on adding fields,
see the [Fields](fields.md) documentation.

### QuerySet setup

By default, a Strawberry Django type will pull data from the default manager for it's Django Model.
You can implement a `get_queryset` to your type to do some extra processing to the queryset,
like further filtering it.

```python
@strawberry.django.type(models.Fruit)
class Berry:

    def get_queryset(self, queryset, info):
        return queryset.filter(name__contains="berry")
```

The `get_queryset` method is given a QuerySet to filter and
a Strawberry `Info` object, containing details about the request.

You can use that `info` parameter to, for example,
limit access to results based on the current user in the request:

```python
@strawberry.django.type(models.Fruit)
class Berry:

    def get_queryset(self, queryset, info):
        if not info.context.request.user.is_staff:
            # Limit access to top secret if the user is not a staff member
            queryset = queryset.filter(
                is_top_secret=False,
            )
        return queryset.filter(
            name__contains="berry",
        )
```
