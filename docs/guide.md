# Models

_Models_ defines the structure of the data we're storing.

Django has recommended that models follow the [active record pattern](https://docs.djangoproject.com/en/4.1/misc/design-philosophies/#models) and include all relevant domain logic. This can be referred to as
"fat models".

This style guide alternatively recommends the concept of "skinny models". Instead of **all** of the logic living in the
model, we can move some of it out into other areas (see upcoming sections).

Some logic should still remain in the models. Otherwise, we'll have anemic domain models that go against a basic idea
of object oriented design (to collect and process data).

## Models Guide

- Models can only have:
  - attributes representing database columns
  - properties or methods for
    - simple information logic for a single model instance
    - validation logic

Example `models.py`:

```python
from datetime import datetime, timedelta
from django.core.exceptions import ValidationError
from django.db import models
from django.db.models.functions import Now
from .managers import PersonManager


class Person(models.Model):
    datetime_created = models.DateTimeField(db_default=Now(), null=False, auto_now_add=True)
    first_name = models.CharField(max_length=255)
    last_name = models.CharField(max_length=255)
    birthdate = models.DateField()
    email = models.EmailField()

    objects = PersonManager()

    @property
    def age(self):
        today = datetime.now().date()
        age = today.year - self.birthdate.year - ((today.month, today.day) < (self.birthdate.month, self.birthdate.day))
        return age

    def get_age_in_x_months(self, x):
        birthdate_plus_x_months = self.birthdate + timedelta(days=30 * x)
        return (datetime.now().date() - birthdate_plus_x_months).days // 365

    def clean(self):
        if self.birthdate is None:
            raise ValidationError({'birthdate': 'A person must have a birthday'})
        if self.birthdate > datetime.now().date():
            raise ValidationError({'birthdate': 'Birthday cannot be in the future'})
```

# Managers

[_Managers_](https://docs.djangoproject.com/en/4.1/topics/db/managers/#custom-managers) should be used for additional functionality around querying data.

## Managers Guide

- Managers can only contain query functionality:
  - complex queries
    - can concern multiple related models
  - queries used in multiple places

Example `managers.py`:

```python
from django.db import models
from datetime import datetime, timedelta

class PersonManager(models.Manager):
    def people_with_birthdays_in_the_next_30_days(self):
        today = datetime.now().date()
        thirty_days_later = today + timedelta(days=30)
        return self.filter(
            birthdate__month=today.month,
            birthdate__day__range=(today.day, thirty_days_later.day)
        )

    def adults(self):
        eighteen_years_ago = datetime.now().date() - timedelta(days=365 * 18)
        return self.filter(birthdate__lte=eighteen_years_ago)
```

# Services

_Services_ should have most of the business/domain logic in the app.

Services will primary be accessed by APIs but can also be used in other places such as CLIs, asynchronous tasks, cron jobs, or
other services.

Also, to elaborate on why this style guide moves some logic from "fat models" to this service layer, the advantages are:

- Improved unit testing. We can more easily mock database calls from the ORM and test business logic in isolation.
- Improved code readability (in my opinion). Everything isn't in one place and is organized by their roles.
- Separation of concerns
  - For example, if we went with fat models and there's business logic that encapsulates multiple models,
    which model do we put that business logic into? With services, models will continue to handle simple logic related to its
    own instance. For anything more complex, like logic that relies on multiple models or makes 3rd party API calls, we can have a
    "service" be concerned with handling that.

## Services Guide

- Services can only have business/domain logic
- Services can access
  - models/managers and their functionality
  - internal or external APIs
  - other services
- Services should primarily live in functions but classes are allowed

Example `services.py`:

```python
from .models import Person
from ..messages.apis import InternalMessageApi


# access models
def get_person(id: int) -> Person:
    person = Person.objects.get(id=id)
    return person


def get_people() -> List[Person]:
    return Person.objects.all()


def create_person(*, first_name, last_name, birthdate, email) -> Person:
    obj = Person(first_name=first_name, last_name=last_name, birthdate=birthdate, email=email)
    obj.full_clean()
    obj.save()
    return obj


# business/domain logic
def send_person_email_message(person_id: int, message: str) -> None:
    # access other services
    person = get_person(person_id)

    # access another app's API to send a message 
    InternalMessageApi.send_message(person.email, message)
```

# Serializers

[_Serializers_](https://www.django-rest-framework.org/api-guide/serializers/) can convert complex data types (python objectd or models) to and from native python data types, which can then be rendered in other formats like JSON.

Note: these examples use [Django Rest Framework](https://www.django-rest-framework.org/)

## Serializers Guide

- Serializers can only have serialization/deserialization functionality
- Serializers should explicitly define and validate API contracts
- Serializers must not be Model Serializers since they tightly couple the presentation and data layers
- Serializers should not be reused to define different API contracts
  - It can lead to unexpected changes
    - For example, let's say two APIs happen to have the same contract at one point and then suddenly we need to change the contract of only one of those APIs. If the two APIs used the same serializer, making the contract change would apply the change to both APIs and that side effect on the second API may not be obvious

Example `serializers.py`:

```python
from rest_framework import serializers

# serializes a person model
class PersonApiGetSchema(serializers.Serializer):
    id = serializers.IntegerField()
    datetime_created = serializers.DateTimeField(format="%Y-%m-%d")
    first_name = serializers.CharField()
    last_name = serializers.CharField()
    birthdate = serializers.DateField()
    email = serializers.EmailField()

# defines API contract for creating a person
class PersonApiPostSchema(serializers.Serializer):
    first_name = serializers.CharField()
    last_name = serializers.CharField()
    birthdate = serializers.DateField()
    email = serializers.EmailField()
```

# APIs

_APIs_ are the entry points to your app and contain presentation logic.
They define the external and internal interfaces for your app.
An external API could be an HTTP API using REST or GraphQL.
An internal API is an access point for an app so other Django apps could interact with that app's functionality.

APIs should communicate with services and not directly with the ORM.

## APIs Guide

- APIs must used as the entry point for the Django app
- APIs must only communicate with services
- APIs should use serializers
- APIs should live in classes

Example `apis.py`:

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from . import services
from .serializers import PersonApiGetSchema


# external APIs
class PersonApi(APIView):
    def get(self, request):
        people = services.get_people()
        serializer = PersonApiGetSchema(people, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = PersonApiPostSchema(data=request.data)
        if serializer.is_valid():
            try:
                people = services.create_person(**serializer.data)
            except ValidationError as e:
                return Response(e, status=status.HTTP_400_BAD_REQUEST)
            serializer = PersonApiGetSchema(institution)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class PersonByIdApi(APIView):
    def get(self, request, id):
        # communicate with services
        person  = services.get_person(id)
        # use serializer
        serializer = PersonApiGetSchema(person)
        data = serializer.data
        return Response(data)


# internal API
class InternalPersonAPI:
    @static_method
    def get_person(id: int) -> 'Person':
        return services.get_person(id)
```

# URLs

_URLs_ define the locations of our external APIs.

## URLs Guide

- URLs must only define the endpoints for external APIs
- URLs must all maintain a consistent pattern, whether it's REST, SOAP, etc.

Example `urls.py`:

```python
from django.urls import path

from .apis import PersonApi, PersonByIdApi


urlpatterns = [
    path('', PersonApi.as_view()),
    path('<int:id>/', PersonByIdApi.as_view()),
]

# In top level urls.py you can do something like this
# urlpatterns = [
#     path(
#         'api/',
#         include(
#             [
#                 path("people/", include("person.urls")),
#             ]
#         )
#     )
# ]
```

# App Config

_App Config_ store metadata for an application. We're sticking with Django's convention and defining app configurations in `apps.py`

## App Config Guide

- refer to Django's [documentation](https://docs.djangoproject.com/en/4.1/ref/applications/#configuring-applications)

# Migrations

_Migrations_ propogate changes we make to our models. We're sticking with Django's convention a `migrations` directory.

## Migrations Guide

- refer to Django's [documentation](https://docs.djangoproject.com/en/4.1/topics/migrations/)

# Tests

_Tests_ are extremely important for verifying the code that we write works as intended and also ensuring that we know when code breaks so we can fix it.

In this style guide, tests are broken down into smaller files based on the file of the code it's testing and whether it's a unit test or integration test. By organizing tests in this way, we can:

- More easily find tests
  - For example, we can find all the test for `services.py` in a `tests_services.py`
- Run specific tests
  - For example, we can choose to run only the tests around services, or only the unit tests, or only the integration tests

Test Driven Development is also encouraged to ensure there's high test coverage (since we have to write the tests first before we code the functionality).

## Tests Guide

- There should be a `tests` directory in every app. This is where all the tests will live.
- Under the `tests` directory, there will be two subdirectories:
  - `unit`
    - This is where all the unit tests will live
  - `integration`
    - This is where all the integration tests will live
- Tests should live in files that mirror the files of the code it's testing
  - For example:
    - `models.py` -> `test_models.py`
    - `services.py` -> `test_services.py`
    - `apis.py` -> `test_apis.py`
  - These files should be placed appropriately in the `unit` or `integration` subdirectories
    - Each subdirectory can have a file with the same name as a file in the other subdirectory
      - For example: You can have two `test_services.py`. One in `unit` and one in `integration`
- Tests should primarily be unit tests instead of integration tests
- Use [mocks](https://docs.python.org/3/library/unittest.mock.html) or [patching](https://docs.python.org/3.5/library/unittest.mock.html#unittest.mock.patch)
  - These tool are useful to mock out things like external API or database calls so we can test our "units" in isolation

Example tests directory:

```
app_name/
├── apis.py
├── models.py
├── serializers.py
├── services.py
├── urls.py
└── tests/
    ├── integration/
    │   ├── test_apis.py
    │   └── test_services.py
    └── unit/
        ├── test_models.py
        ├── test_serializers.py
        └── test_services.py
```

# Other Notes

You can remove any files you don't need.

You can add any files as needed, such as:

- `enums.py`
- `constants.py`
- `exceptions.py`
- `middleware.py`
- `query_sets.py`
- `admin.py`

For any additional files that you create, it is highly recommended that they:

- serve a single purpose
- do not contain code that should actually belong in the other files
  - For example, if you create an `exceptions.py` file, don't start putting models or APIs in there

However, there are files I recommend **against** creating:

- `tests.py`
  - This style guide is in favor of breaking tests into smaller and more specific files, rather than a single generic tests file
- `utils.py`
  - This file can get bloated because it isn't clear what code should live in there
- `views.py`
  - This style guide is in favor of APIs
