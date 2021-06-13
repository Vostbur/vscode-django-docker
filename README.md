# Creating dockerized Django app with VSCode

## Creating a Django project

Start by creating a new directory for your project (eg: **vscode-django-docker**), and open it in VSCode.

    mkdir vscode-django-docker
    cd vscode-django-docker
    git init
    code .

Then, add a *.gitignore* for your project. You can use [template for Python](https://github.com/github/gitignore/blob/master/Python.gitignore) provided by GitHub or 
generate on [gitignore.io](https://www.toptal.com/developers/gitignore).

Now let's create a Django project by running the one-line-command below

**for Windows:**

    docker run -v %cd%:/app -w /app python:3.9-alpine sh -c "pip install Django==3.2.4 && django-admin startproject config ."

**for Linux**

    docker run -v ${PWD}:/app -w /app python:3.9-alpine sh -c "pip install Django==3.2.4 && django-admin startproject config ."

## Creating auto generating Docker templates for Django with VSCode

Press **Ctrl + Shift + P** add start typing *add docker*. Choose **Docker: Add Docker Files to Workspace** at command pallet.

After answering the questions, the following files should be added to your project.

    .vscode/
        launch.jsonn
        tasks.json
    .dockerignore
    docker-compose.yml
    Dockerfile
    requirements.txt

Update the **requirements.txt** file to look like this (choose the versions for your needs):

    django>=3.2,<3.3
    gunicorn>=20.0.4,<20.1

Update the **Dockerfile**. Change base Docker image to `python:3.9-alpine3.13` because it is more lightweight.

    FROM python:3.9-alpine3.13

Modified the gunicorn command to use the **config.wsgi** file for running our app. Change the last string to
    
    CMD ["gunicorn", "--bind", "0.0.0.0:8000", "config.wsgi"]

Now you can build and run docker container by running the following commands:

    docker-compose build
    docker-compose up

You should be able to view your Django app by visiting [http://127.0.0.1:8000](http://127.0.0.1:8000) in your favourite browser.

## Debugging a dockerized Django app with VSCode

Create a new file at **./config/views.py** and fill it with the following contents:

    from django.http import HttpResponse


    def index(request):
        return HttpResponse('Hello World!')

Then modify **./config/urls.py** to the following code:

    from django.contrib import admin
    from django.urls import path

    from .views import index


    urlpatterns = [
        path('', index),
        path('admin/', admin.site.urls),
    ]

For testing debugger add a breakpoint on the return line of **./config/views.py** by clicking the area left of the line number.

Now start the debugger by selecting **Run > Start Debugging**. Debugger running in VSCode and you can inspect how your dockerized app works.

## Setup a basic workflow using GitHub Actions

Add to the **requirements.txt** file the following line to check with linter:

    flake8>=3.9.2,<3.10

Create a new file at **.github/workflows/cd.yml** and fill it with the following contents:

    ---
    name: Code checks

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

    jobs:
    test:
        name: Test
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v2
        - name: Test
            run: docker-compose run --rm . sh -c "python manage.py test"

    lint:
        name: Lint
        runs-on: ubuntu-20.04
        steps:
        - uses: actions/checkout@v2
        - name: Lint
            run: docker-compose run --rm . sh -c "flake8"


[![Build Status](https://github.com/Vostbur/vscode-django-docker/actions/workflows/cd.yml/badge.svg&branch=master)](https://github.com/Vostbur/vscode-django-docker/actions/workflows/cd.yml)

----
### Links:

- [1] [DEBUGGING A DOCKERIZED DJANGO APP WITH VSCODE](https://londonappdeveloper.com/debugging-a-dockerized-django-app-with-vscode/)
- [2] [Python in a container](https://code.visualstudio.com/docs/containers/quickstart-python)
- [3] [GitHub Actions для автоматической проверки кода](https://www.youtube.com/watch?v=NijFSs03Pd4)
- [4] [HOW TO USE GITHUB ACTIONS](https://londonappdeveloper.com/how-to-use-github-actions/)