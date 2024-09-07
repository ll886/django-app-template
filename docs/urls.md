# URLs

_URLs_ define the locations of our external APIs.

## URLs Guide

- URLs must only define the endpoints for external APIs
- URLs must all maintain a consistent pattern, whether it's REST, SOAP, etc.

Example `urls.py`:

```python
from django.urls import path

from .apis import PersonAPI


urlpatterns = [
    path('person/<int:id>/', PersonAPI.as_view()),
]
```
