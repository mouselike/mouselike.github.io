---
layout: post
title:  "Django not loading environment variables correctly"
categories: python, django
---

I had the following issue: I have a Django app with a `.env`-file in the root directory of my project. In this file I had the following variables:
```shell
DB_NAME='db-dev'
DB_USER='???'
DB_PASSWORD='???'
DB_HOST='???'
DJANGO_SECRET_KEY='???'
DEBUG=False
```

For testing purposes, I wanted to change the variable `DB_NAME` in `.env` do `db-prod`. After changing the value I realised that Django was still connecting to the old database (`db-dev`). 

To investigate this issue further, I changed the value of the variable `DEBUG` from `False` to `True` and added the following command to `settings.py`:
```python
print(f"DEBUG={os.getenv("DEBUG")}")
```

After running `python manage.py runserver` I saw `DEBUG=True` in my terminal. This meant, that `.env` is loaded correctly and it must have to do something with the variable `DB_NAME`.

I ran a couple of tests and finally found the solution. In my terminal I executed the following command:
```bash
echo $DB_NAME
```

The result I got was `db-dev`. 

This meant, I had an environment variable set on my OS with the exact same name (`DB_NAME`). So, instead of using the variable from my `.env`-file, [[Django]] was using the environment variable on OS-level.

To solve this issue I had two options. First, I could unset the variable running `unset DB_NAME` in my terminal. As I didn't know what system added `DB_NAME` as a variable to my system, this could only be a temporary fix. 

So, I decided to tweak the `load_dotenv()` function in `settings.py` and tell it to overwrite any existing variables. 
```python
load_dotenv(find_dotenv(), override=True)
```