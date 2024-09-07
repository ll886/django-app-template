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

You can run integration and unit tests separately with pytest:
- `pytest -k "integration"`
- `pytest -k "unit"`

These commands will run tests located in "integration" or "unit" directories.