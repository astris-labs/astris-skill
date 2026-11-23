# Python recipes

Use these recipes for a FastAPI, Django or Flask app that deploys with the default builder. Each file block shows the file name and the lines it must hold. Change the module names to the app's own names.

## FastAPI

Omit `build`, because the default builder needs no Dockerfile.

Pass `port` `8000` at creation. In step 3 of the deploy flow, call `env` and set `PORT` to `8000`, then redeploy in step 4. The start command reads `PORT`, so the first deploy does not start until that redeploy.

Gunicorn with `--bind :$PORT` listens on all addresses, and the Uvicorn worker runs the FastAPI app.

```requirements.txt
fastapi
gunicorn
uvicorn-worker
```

```Procfile
web: gunicorn main:app --worker-class uvicorn_worker.UvicornWorker --bind :$PORT
```

```main.py
from fastapi import FastAPI

app = FastAPI()
```

## Django

Omit `build`, because the default builder needs no Dockerfile.

Pass `port` `8000` at creation. In step 3 of the deploy flow, call `env` and set `PORT` to `8000`, then redeploy in step 4. The start command reads `PORT`, so the first deploy does not start until that redeploy.

Gunicorn with `--bind :$PORT` listens on all addresses. The Procfile collects the static files and runs the migrations before the server starts.

Keep `DEBUG` off. Set `DJANGO_SECRET_KEY` with `env` to a new random value. Set `DJANGO_DEBUG` only on a local machine.

Add the lines below to the `settings.py` that Django made, and keep its other lines. If the app uses the Postgres database from step 2, set `DATABASES` from the variables that hold those values.

```requirements.txt
django
gunicorn
whitenoise
```

```Procfile
web: python manage.py collectstatic --noinput && python manage.py migrate && gunicorn mysite.wsgi --bind :$PORT
```

```settings.py
import os

SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]
DEBUG = os.environ.get("DJANGO_DEBUG") == "1"
ALLOWED_HOSTS = [".astris.live"]
CSRF_TRUSTED_ORIGINS = ["https://*.astris.live"]
STATIC_ROOT = BASE_DIR / "staticfiles"
STORAGES = {"default": {"BACKEND": "django.core.files.storage.FileSystemStorage"}, "staticfiles": {"BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage"}}
```

Put `"whitenoise.middleware.WhiteNoiseMiddleware",` in `MIDDLEWARE` directly after `"django.middleware.security.SecurityMiddleware",`.

For the app's own domain, add its hostname to `ALLOWED_HOSTS` and its `https://` address to `CSRF_TRUSTED_ORIGINS`.

## Flask

Omit `build`, because the default builder needs no Dockerfile.

Pass `port` `8000` at creation. In step 3 of the deploy flow, call `env` and set `PORT` to `8000`, then redeploy in step 4. The start command reads `PORT`, so the first deploy does not start until that redeploy.

Gunicorn with `--bind :$PORT` listens on all addresses. Do not start the app with `flask run` or `app.run()`.

```requirements.txt
flask
gunicorn
```

```Procfile
web: gunicorn app:app --bind :$PORT
```

```app.py
from flask import Flask

app = Flask(__name__)
```
