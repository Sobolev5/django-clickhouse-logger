# django-clickhouse-logger

Logging Django errors to the Clickhouse database with daily rotation.

```no-highlight
https://github.com/Sobolev5/django-clickhouse-logger
```

![](https://github.com/Sobolev5/django-clickhouse-logger/blob/master/screenshots/screenshot.png)   
> screenshot from Dbeaver

# How to use it

To install run:
```python
pip install django-clickhouse-logger # stable version
pip install -U git+https://github.com/Sobolev5/django-clickhouse-logger.git@master # development version
```

Add Clickhouse logger to INSTALLED_APPS:
```python
INSTALLED_APPS = INSTALLED_APPS + ("django_clickhouse_logger",)
```

Set Clickhouse logger environment variables in a settings.py:
```python
DJANGO_CLICKHOUSE_LOGGER_HOST = "127.0.0.1" 
DJANGO_CLICKHOUSE_LOGGER_PORT = 9000
DJANGO_CLICKHOUSE_LOGGER_USER = "default"
DJANGO_CLICKHOUSE_LOGGER_PASSWORD = ""
DJANGO_CLICKHOUSE_LOGGER_TTL_DAY = 1 # Log rotation (in days).
DJANGO_CLICKHOUSE_LOGGER_REQUEST_EXTRA = "session" 
# Means request.session. 
# Extra attribute of django.core.handlers.wsgi.WSGIRequest object for logging. 
# For example you define request.company in your custom middleware
# and set DJANGO_CLICKHOUSE_LOGGER_REQUEST_EXTRA = "company" in this case.
```

Run Clickhouse tables creation script:
```sh
python manage.py shell --command="from django_clickhouse_logger.db import *; create_logger_table(); create_capture_exception_table();" 
```
This script will create the database `django_clickhouse_logger` with tables `logger` (Django errors) and `capture_exception` (Captured exceptions).
  
  
Add Clickhouse logger to your logger configuration in settings.py:
```python
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "filters": {
        "require_debug_true": {"()": "django.utils.log.RequireDebugTrue",}, 
        "require_debug_false": {"()": "django.utils.log.RequireDebugFalse"}
    },
    "formatters": {
        "console": {"format": "%(asctime)s - %(levelname)s - %(message)s"},
    },
    "handlers": {
        "console": {"level": "INFO", "filters": ["require_debug_true"], "class": "logging.StreamHandler", "formatter": "console"},
        "django_clickhouse_logger": {"level": "ERROR", "filters": ["require_debug_false"], "class": "django_clickhouse_logger.handlers.LoggerHandler"},              
    }, 
    "loggers": {
        "django": {"handlers": ["console"], "level": "INFO",},
        "django.request": {"handlers": ["django_clickhouse_logger"], "level": "ERROR", 'propagate': False},
    },
}
```

If you want to test just change filter `require_debug_false` to `require_debug_true` 
for `django_clickhouse_logger` handler and raise error in any django view.  
For visual interface to the clickhouse table `django_clickhouse_logger.logger` i recommend to using [Dbeaver](https://dbeaver.io/).
  

If you want to truncate tables `logger` or `capture_exception` just run:
```sh
python manage.py shell --command="from django_clickhouse_logger.db import *; truncate_logger_table();"
python manage.py shell --command="from django_clickhouse_logger.db import *; truncate_capture_exception_table();"
```

# Capture exception
To catch exceptions manually:
```python

from django_clickhouse_logger import capture_exception   

try:
    print(undefined_variable)
except Exception as e:
    capture_exception(e)

try:
    print(undefined_variable)
except Exception as e:
    capture_exception(e, "add some text here")
```
**Note:** You can integrate `capture_exception` function in any python project.
Django is not necessary in this case.



# Integrations
`django_clickhouse_logger` is default logger for `upserver` project.  
Go to https://github.com/Sobolev5/upserver for further instructions.


# Time tracker for developers
Use [Workhours.space](https://workhours.space/) for your working time tracking. It is free.

