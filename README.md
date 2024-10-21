# Django-Python-Docker
 Use Docker to host a Django APP

Steps to build it:

-. Install **Python** from (here)[https://www.python.org/downloads/]

-. Install `pip` from (here)[https://pip.pypa.io/en/latest/installation/]

-. Create a folder `mkdir <project_name>` and `cd <project_name>`

-. Create a virtual environment `python3 -m venv env` ("env" is the virtual environment name)

-. Activate the virtual environment using `source env/bin/activate`

-. Start with `git init`

-. Install **Django** `python -m pip install Django`

-. Crate a **Django** project `django-admin startproject <project_name> .`

-. Update `pip` and install packages
```bash
(env) $ pip install --upgrade pip
(env) $ pip install gunicorn python-dotenv
(env) $ pip freeze > requirements.txt
```

-. Craete `.env` file (and update this values using strong and hard to get password and secret key):
```bash
# Postgres
POSTGRES_DB=postgres # this is the default name
POSTGRES_PASSWORD=DjangoPW
# Django settings
DJANGO_SETTINGS_SECRET_KEY=Password1234!
DJANGO_SETTINGS_DEBUG=True
DJANGO_SETTINGS_ALLOWED_HOSTS=0.0.0.0 localhost 127.0.0.1 [::1]
```
-. Open `<project_name>/settings.py` and add
```bash
from dotenv import load_dotenv
import os

load_dotenv()
```
-. Update the next variables using
```bash
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.environ.get("DJANGO_SETTINGS_SECRET_KEY")

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = bool(os.environ.get("DJANGO_SETTINGS_DEBUG"))

ALLOWED_HOSTS = os.environ.get("DJANGO_SETTINGS_ALLOWED_HOSTS").split(" ")
```
-. Start docker with `docker init` and answer the questions, since there is a Python program in here, that's the one selected by default, the next questions choose the defult

-. Update the `Dockerfile` with the next file:
```bash
# syntax=docker/dockerfile:1

# Comments are provided throughout this file to help you get started.
# If you need more help, visit the Dockerfile reference guide at
# https://docs.docker.com/go/dockerfile-reference/

# Want to help us make this template better? Share your feedback here: https://forms.gle/ybq9Krt8jtBL3iCk7

ARG PYTHON_VERSION=3.11.6
FROM python:${PYTHON_VERSION}-slim as base

# Prevents Python from writing pyc files.
ENV PYTHONDONTWRITEBYTECODE=1

# Keeps Python from buffering stdout and stderr to avoid situations where
# the application crashes without emitting any logs due to buffering.
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Create a non-privileged user that the app will run under.
# See https://docs.docker.com/go/dockerfile-user-best-practices/
ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser

# Download dependencies as a separate step to take advantage of Docker's caching.
# Leverage a cache mount to /root/.cache/pip to speed up subsequent builds.
# Leverage a bind mount to requirements.txt to avoid having to copy them into
# into this layer.
RUN --mount=type=cache,target=/root/.cache/pip \
    --mount=type=bind,source=requirements.txt,target=requirements.txt \
    python -m pip install -r requirements.txt

# Switch to the non-privileged user to run the application.
USER appuser

# Copy the source code into the container.
COPY . .

# Expose the port that the application listens on.
EXPOSE 8000

# Run the application.
CMD gunicorn 'Django.wsgi' --bind=0.0.0.0:8000
```
-. Update `compose.yaml` with the next file:
```bash
services:
  server:
    build:
      context: .
    ports:
      - 8000:8000
    env_file:
      - .env
    volumes:
      - .:/app
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres
    restart: always
    user: postgres
    volumes:
      - db-data:/var/lib/postgresql/data
    # environment:
    #   - POSTGRES_DB=example
    #   - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    env_file:
      - .env
    expose:
      - 5432
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
```
-. Start the Project using `docker compose up --build` and go to `http://localhost:8000`

Happy Coding!
