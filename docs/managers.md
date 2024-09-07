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

class PersonManager(models.Manager):
    # complex query
    def people_with_birthdays_in_the_next_30_days(self):
        # <insert query for people whose birthday (month and day only) are within the next 30 days>
        pass

    # query used in multiple places
    def adults(self):
        # <insert query for people whose birthdate was more than 18 years ago>
        pass
```
