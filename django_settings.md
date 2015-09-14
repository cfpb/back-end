# Django Settings

Our primary concern when we consider the settings our apps need and provide is **obviousness**. 

- It should always be clear why a certain setting has the value it has. 
- It should always be clear what settings a particular app expects. 
- App-provided settings should play well with other app-provided settings.

## Types of settings

There are, generally, three types of Django settings:

1. *Project settings*
    
    These are the default settings that every Django "project" (a runnable instance Django instance) gets automatically. This is the `settings.py` file. Generally speaking, there should be no reason to change these except to set some sane defaults for `DATABASES`, `DEBUG` and `SECRET_KEY` (which are environment-specific).
    
2. *App settings*
    
    An app may introduce its own custom settings, with particular types of values. These should generally be defined with sane defaults in a `settings` module within your app. 
    
3. *Environment settings*

    These are settings that are particular to, say, a development environment, a staging environment, a production environment, an environment where an app lives along-side other apps within a single Django project, etc. For a typical Django project these would likely include `DEBUG`, `DATABASES`, `SECRET_KEY`, `INSTALLED_APPS`, as well as `CACHES`, and others. It may also include custom app settings. 

Each type of settings has implications for CFPB projects. The rest of this document details our recommended approach to each of these.

## Django Project settings

An app may choose to include an sample Django project in its repositority. This ensures that the app can be developed and demoed in isolation, and can be tested by tools like Travis. 

*This default project's `settings.py` file should generally not deviate from the Django defaults except where necessary.* Where is it necessary?

- `DEBUG` should be changed to `False` — environment-specific settings may chose to set it to `True`, but the choice to do so should be explicit.
- `SECRET_KEY` should be set to a dummy value, or unset completely — an actual value will be set by environment-specific settings that do not get committed.

## App settings

Apps might chose to introduce their own custom settings. These settings should be defined in the `settings.py` file within the app package. The general pattern for an app's `settings.py` file is to import the Django `settings` module and attempt to fetch the setting from there, with a default provided if it has not been set. For example:

```
from django.conf import settings

MY_SETTING_DEFAULT = 'foo'
MY_SETTING = getattr(settings, 
    'MY_SETTING',
    MY_SETTING_DEFAULT)
```

Then these settings can be accessed via this settings module:

```
import myapp.settings
...
myapp.settings.MY_SETTING
```

## Environment settings

An app may include example environment settings in its repository. There should generally be three assumed environments:

1. A local development environment where the app is run alone in a a Django project (`example_local_settings.py`)
2. A development environment where the app is run along side other apps within a single Django project (`example_dev_settings.py`)
3. A production environment where the app is run along side other apps within a single Django project `example_prod_settings.py`. 

These example environment settings files should include app-specific modifications for Django settings such as `INSTALLED_APPS` and `CACHES` that append to the settings.

```
INSTALLED_APPS += ('myapp_dependency', 'myapp')
```

Other environment settings, such as `DEBUG`, `DATABASES`, and `SECRET_KEY` can't be set in the example files, but the example files can provide templates for them. In the ideal case, these settings will be provided as Jinja2 templates, both human-readable and editable, and usable by automation tools like Ansible:

```
DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': {{ db_name }},
            'USER': {{ db_user }},
            'PASSWORD': {{ db_password }},
            'HOST': {{ db_host }},  
            'PORT': {{ db_port }},  
        },
}
```

For the `dev` and `production` example environment settings files, templates for `DEBUG`, `DATABASES`, and `SECRET_KEY` may be unnecessary and unwanted.