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
from .serializers import PersonAPIGetSchema


# external API
class PersonAPI(APIView):
    def get(self, request, id):
        # communicate with services
        person  = services.get_person(id)
        # use serializer
        serializer = PersonAPIGetSchema(person)
        data = serializer.data
        return Response(data)


# internal API
class InternalPersonAPI:
    @static_method
    def get_person(id: int) -> 'Person':
        return services.get_person(id)
```
