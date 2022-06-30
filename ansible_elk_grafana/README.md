ARA
```
Checar configs
# python3 -m ara.setup.action_plugins
# python3 -m ara.setup.callback_plugins

# vim /root/.ara/server/settings.yaml

default:
  ALLOWED_HOSTS:
  - ::1
  - 127.0.0.1
  - localhost
  - 0.0.0.0
  BASE_DIR: /root/.ara/server
  CORS_ORIGIN_ALLOW_ALL: false
  CORS_ORIGIN_REGEX_WHITELIST: []
  CORS_ORIGIN_WHITELIST:
  - http://127.0.0.1:8000
  - http://192.168.0.180:8000
  - http://localhost:3000
  CSRF_TRUSTED_ORIGINS: []
  DATABASE_CONN_MAX_AGE: 0
  DATABASE_ENGINE: django.db.backends.sqlite3
  DATABASE_HOST: null
  DATABASE_NAME: /root/.ara/server/ansible.sqlite
  DATABASE_OPTIONS: {}

# ara-manage runserver

# ara result list

```