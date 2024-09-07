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


class Person(models.Model):
    # attribute representing database column
    first_name = models.CharField()
    last_name = models.CharField()
    birthdate = models.DateField()
    email = models.EmailField()

    # property for simple information logic for a single model instance
    @property
    def age(self):
        # <insert code to return age based on self.birthdate>

    # method for simple information logic for a single model instance
    def get_age_in_x_months(self, x)
        #  <insert code to return age based on self.birthdate and x argument>

    # validation logic
    def clean(self):
        if self.birthdate is None:
            raise ValidationError({'birthdate': _('A person must have a birthday')})
        if self.birthdate > datetime.now().date():
            raise ValidationError({'birthdate': _('Birthday cannot be in the future')})
```
