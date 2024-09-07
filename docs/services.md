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


# access models
def get_person(id: int) -> Person:
    person = Person.objects.get(id=id)
    return person


# business/domain logic
def send_person_email_message(person_id: int, message: str) -> None:
    # access other services
    person = get_person(person_id)

    # access internal API
    # <insert code to access a "messages" app's API>
    #   - the messages app has a service that accesses external APIs such as SendGrid to send a message
```
