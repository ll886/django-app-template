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
class PersonAPIGetSchema(serializers.Serializer):
    first_name = serializers.CharField()
    last_name = serializers.CharField()
    birthdate = serializers.DateField()
    email = serializers.EmailField()

# defines API contract for creating a person
class PersonAPIPostSchema(serializers.Serializer):
    first_name = serializers.CharField()
    last_name = serializers.CharField()
    birthdate = serializers.DateField()
    email = serializers.EmailField()

```
