# django-clickhouse-logger

Logging django errors to the clickhouse database.

```no-highlight
https://github.com/Sobolev5/django-clickhouse-logger
```

# How to use it

To install run:
```no-highlight
pip install django-clickhouse-logger
```

Add the clickhouse logger to the INSTALLED_APPS:
```python
INSTALLED_APPS = INSTALLED_APPS + ("clickhouse_logger",)
```

Set clickhouse logger environment variables in a settings.py:
```python
CLICKHOUSE_LOGGER_HOST = 127.0.0.1 
CLICKHOUSE_LOGGER_PORT = 9000
CLICKHOUSE_LOGGER_USER = "default"
CLICKHOUSE_LOGGER_PASSWORD = ""
CLICKHOUSE_LOGGER_TTL_DAY = 1 # Log rotation (in days).
CLICKHOUSE_LOGGER_REQUEST_EXTRA = 'session' # Means request.session. 
# Extra attribute of django.core.handlers.wsgi.WSGIRequest object for logging. 
# You can define own attribute in your custom middleware. 
```

Run the clickhouse database creation script on a server:
```sh
python manage.py shell --command="import clickhouse_logger; clickhouse_logger.proxy.clickhouse.create_clickhouse_tables()"
```
This script will create the database `clickhouse_logger` with the table `records` for django errors store.


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
        "clickhouse_logger_handler": {"level": "ERROR", "filters": ["require_debug_false"], "class": "clickhouse_logger.handlers.ClickhouseLoggerHandler"},              
    }, 
    "loggers": {
        "django": {"handlers": ["console"], "level": "INFO",},
        "django.request": {"handlers": ["clickhouse_logger_handler"], "level": "ERROR", 'propagate': False},
    },
}
```

To test you can change filter `require_debug_false` to `require_debug_true` for `clickhouse_logger_handler` and raise a error in any django view.
For visual interface to clickhouse table `clickhouse_logger.records` i recommend using a [Dbeaver](https://dbeaver.io/).


## Little advertisement
[WorkHours.Space](https://workhours.space/) - Smart working hours accounting system *.
Time tracker. Automatic payroll calculation. Uploading reports to PDF and XLS. Modern adaptive interface.
###### * This is my free service for developers.