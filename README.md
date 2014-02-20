# Brunch for Django

This is a very simple [Brunch](http://brunch.io) skeleton for [Django](http://djangoproject.com) projects.

### Features
- JavaScript concatenation and minification
- CommonJS Modules
- Stylus / CSS compilation, concatenation and minification
- Source Maps
- Browser auto-reload

### Getting Started

First you will need to create a `static` folder on the root of your Django project, right next to `manage.py`.

```
$ mkdir static
```

Then create the brunch application folder. I use the name `src` for the brunch folder to make it obvious that's where the sources go.

```
$ brunch new gh:gcollazo/brunch-for-django src
```

Then install all of the skeleton dependencies with `npm`.

```
$ cd src
$ npm install
```

Once that's done you can test the project by running the `watch` command with the `-s` flag to enable a webserver.

```
$ brunch watch -s
```

Now navigate to `http://localhost:3333` and make sure you see a success message.

Notice that the brunch command created a `dist` folder right next to the `src` folder we created earlier. This is where all the compiled and minified files are going to go.

Now we must change our Django `settings.py` so that when we do `manage.py collectstatic` the command knows where to get the files from, the `diet` folder.

```python
# settings.py

BRUNCH_DIR = os.path.join(BASE_DIR, 'static', 'src')
STATIC_URL = '/static/dist/'

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static', 'dist'),
)
```

Now we want `brunch` to be running constantly on the background and building files as soon as we change them. For that we'll make a custom `manage.py` command.

Create an app on your project.

```
$ manage.py startapp core
$ cd core

$ mkdir management
$ cd management
$ touch __init__.py

$ mkdir commands
$ cd commands
$ touch __init__.py
```

Now create a new file in the `core/management/commands` directory called `brunchserver.py`.

```python
# This command is based on the one created
# by Brandon Konkle (@bkonkle) for Grunt
# Read more: http://bit.ly/1mdGLgv

import os
import subprocess
import atexit
import signal

from django.conf import settings
from django.contrib.staticfiles.management.commands.runserver import Command\
    as StaticfilesRunserverCommand


class Command(StaticfilesRunserverCommand):

    def inner_run(self, *args, **options):
        self.start_brunch()
        return super().inner_run(*args, **options)

    def start_brunch(self):
        self.stdout.write('--> Starting brunch')
        self.brunch_process = subprocess.Popen(
            ['cd {0} && brunch watch --server'.format(settings.BRUNCH_DIR)],
            shell=True,
            stdin=subprocess.PIPE,
            stdout=self.stdout,
            stderr=self.stderr,
        )

        self.stdout.write('--> Brunch process on pid {0}'.format(self.brunch_process.pid))

        def kill_brunch_process(pid):
            self.stdout.write('--> Closing brunch process')
            os.kill(pid, signal.SIGTERM)

        atexit.register(kill_brunch_process, self.brunch_process.pid)
```

Before running the command we must add the `core` app to `INSTALLED_APPS`.

```python
# settings.py

INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',  # <-- Here
)
```

Now you should be able to run

```
$ manage.py brunchserver
```

That will run the Django development server and the Brunch server.

There's one more thing. You need to insert the scripts and styles on your `base.html` file.

```html
<!-- base.html -->

<link rel="stylesheet" href="{% static 'stylesheets/app.css' %}">

<script src="{% static 'javascripts/app.js' %}"></script>

<script>require('initialize');</script>
```

Ok, now that's it. Enjoy.

### License

```
All of brunch-for-django is licensed under the MIT license.

Copyright (c) 2014 Giovanni Collazo

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
