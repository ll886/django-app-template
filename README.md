## Django App Template

Django App Template is a custom template for the django-admin startapp command.

https://docs.djangoproject.com/en/4.1/ref/django-admin/#cmdoption-startapp-template

## Usage

Activate a virtual environment with `django` / `django-admin` installed.

Option 1: Create app in the current directory:

```
django-admin startapp app_name --template=https://github.com/ll886/django-app-template/archive/template.zip
```

Option 2: Create app in a subdirectory:

```
mkdir -p folder/app_name

django-admin startapp app_name folder/app_name --template=https://github.com/ll886/django-app-template/archive/template.zip
```

You can also create an app at an even deeper level (Ex. replace `folder/app_name` with `folder/folder/app_name` or `folder/folder/folder/app_name` and so on).

## Style Guide

You can view the style guide in the [docs](docs/) folder.

## Acknowledgments

See [here](docs/acknowledgements.md).
