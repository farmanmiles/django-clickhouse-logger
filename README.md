# django-clickhouse-logger

Logging django errors to the clickhouse database with daily rotation.

```no-highlight
https://github.com/Sobolev5/django-clickhouse-logger
```

![](https://github.com/Sobolev5/django-clickhouse-logger/blob/master/screenshots/screenshot.png)   
> screenshot from dbeaver

# How to use it

To install run:
```no-highlight
pip install django-clickhouse-logger
```

Add the clickhouse logger to INSTALLED_APPS:
```python
INSTALLED_APPS = INSTALLED_APPS + ("django_clickhouse_logger",)
```

Set clickhouse logger environment variables in a settings.py:
```python
DJANGO_CLICKHOUSE_LOGGER_HOST = "127.0.0.1" 
DJANGO_CLICKHOUSE_LOGGER_PORT = 9000
DJANGO_CLICKHOUSE_LOGGER_USER = "default"
DJANGO_CLICKHOUSE_LOGGER_PASSWORD = ""
DJANGO_CLICKHOUSE_LOGGER_TTL_DAY = 1 # Log rotation (in days).
DJANGO_CLICKHOUSE_LOGGER_REQUEST_EXTRA = "session" # Means request.session. 
# Extra attribute of django.core.handlers.wsgi.WSGIRequest object for logging. 
# You can define own attribute in your custom middleware. 
```

Run the clickhouse database creation script:
```sh
>>> python manage.py shell --command="import django_clickhouse_logger; django_clickhouse_logger.clickhouse.create_clickhouse_table()"
```
This script will create the database `django_clickhouse_logger` with the table `records` for django errors store.


Add the clickhouse logger to your logger configuration in a settings.py:
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
        "django_clickhouse_logger": {"level": "ERROR", "filters": ["require_debug_false"], "class": "django_clickhouse_logger.handlers.ClickhouseLoggerHandler"},              
    }, 
    "loggers": {
        "django": {"handlers": ["console"], "level": "INFO",},
        "django.request": {"handlers": ["django_clickhouse_logger"], "level": "ERROR", 'propagate': False},
    },
}
```

If you want to test just change filter `require_debug_false` to `require_debug_true` for `django_clickhouse_logger` handler and raise error in any django view.
For visual interface to the clickhouse table `django_clickhouse_logger.records` i recommend using [Dbeaver](https://dbeaver.io/).

If you want to truncate table `django_clickhouse_logger.records` just run:
```sh
>>> python manage.py shell --command="import django_clickhouse_logger; django_clickhouse_logger.clickhouse.truncate_clickhouse_table()"
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
    capture_exception(e, "add some text")
```

# Try my free time tracker
My free time tracker for developers [Workhours.space](https://workhours.space/). 

