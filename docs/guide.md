# Core Structure

<!-- app_name
  management
    __init__.py
    commands
      __init__.py
  migrations
    __init__.py
  tests
    __init__.py
    integration
      __init__.py
    unit
      __init__.py
  apis.py
  managers.py
  models.py
  serializers.py
  services.py
  urls.py -->

<!-- generated with https://tree.nathanfriend.io/ -->
```
app_name/
├── management/
│   ├── __init__.py
│   └── commands/
│       └── __init__.py
├── migrations/
│   └── __init__.py
├── tests/
│   ├── __init__.py
│   ├── integration/
│   │   └── __init__.py
│   └── unit/
│       └── __init__.py
├── apis.py
├── managers.py
├── models.py
├── serializers.py
├── services.py
└── urls.py
```

# Guides
- [management](management.md)
- [migrations](migrations.md)
- [tests](tests.md)
- [apis.py](apis.md)
- [managers.py](managers.md)
- [models.py](models.md)
- [serializers.py](serializers.md)
- [services.py](services.md)
- [urls.py](urls.md)

# Notes

You can remove any files you don't need.

You can add any files as needed, such as:
- [apps.py](apps.md)
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

There are files I recommend **against** creating:

- `tests.py`
  - This style guide is in favor of breaking tests into smaller and more specific files and, rather than a single generic test file.
- `utils.py` / `helpers.py`
  - These files can become bloated because it isn't clear what code should live in there
  - They just become a drawer to dump code into
  - You should try to put code into well-named files that describe what the code does.
- `views.py`
  - This style guide is in favor of APIs
