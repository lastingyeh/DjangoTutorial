### Advanced tutorial: How to write reusable apps

##### Installing some prerequisites

[To Build Package: Use setuptools](https://pypi.python.org/pypi/setuptools)

##### Ensure pip, setuptools, and wheel are up to date:

    > python -m pip install --upgrade pip setuptools wheel

##### Packaging your app

s1. create DIR /django-polls/

*package-named: it must be unique in INSTALLED_APPS. Avoid using the same label as any of the Django contrib packages*

s2. Move the polls directory into the django-polls directory.

s3. Create a file django-polls/README.rst with the following contents:

`django-polls/README.rst`

    =====
    Polls
    =====

    Polls is a simple Django app to conduct Web-based polls. For each
    question, visitors can choose between a fixed number of answers.

    Detailed documentation is in the "docs" directory.

    Quick start
    -----------

    1. Add "polls" to your INSTALLED_APPS setting like this::

        INSTALLED_APPS = [
            ...
            'polls',
        ]

    2. Include the polls URLconf in your project urls.py like this::

        path('polls/', include('polls.urls')),

    3. Run `python manage.py migrate` to create the polls models.

    4. Start the development server and visit http://127.0.0.1:8000/admin/ to create a poll (you'll need the Admin app enabled).

    5. Visit http://127.0.0.1:8000/polls/ to participate in the poll.

s4. Create a django-polls/LICENSE file.

`django-polls/LICENSE`

s5. Create a setup.py /django-polls/setup.py

`django-polls/setup.py`

    import os
    from setuptools import find_packages, setup

    with open(os.path.join(os.path.dirname(__file__), 'README.rst')) as readme:
        README = readme.read()

    # allow setup.py to be run from any path
    os.chdir(os.path.normpath(os.path.join(os.path.abspath(__file__), os.pardir)))

    setup(
        name='django-polls',
        version='0.1',
        packages=find_packages(),
        include_package_data=True,
        license='BSD License',  # example license
        description='A simple Django app to conduct Web-based polls.',
        long_description=README,
        url='https://www.example.com/',
        author='Your Name',
        author_email='yourname@example.com',
        classifiers=[
            'Environment :: Web Environment',
            'Framework :: Django',
            'Framework :: Django :: X.Y',  # replace "X.Y" as appropriate
            'Intended Audience :: Developers',
            'License :: OSI Approved :: BSD License',  # example license
            'Operating System :: OS Independent',
            'Programming Language :: Python',
            'Programming Language :: Python :: 3.5',
            'Programming Language :: Python :: 3.6',
            'Topic :: Internet :: WWW/HTTP',
            'Topic :: Internet :: WWW/HTTP :: Dynamic Content',
        ],
    )

s6. Create a file /django-polls/MANIFEST.in

`django-polls/MANIFEST.in`

    include LICENSE
    include README.rst
    recursive-include polls/static *
    recursive-include polls/templates *

s7. Create an empty directory django-polls/docs for future documentation. Add additional line recommended.

`django-polls/MANIFEST.in`

    recursive-include docs *

s8. Building package with 'python setup.py sdist (run from inside django-polls).

`django-polls/`

    > python setup.py sdist

*create dist and builds your new package, django-polls-0.1.tar.gz. like: django-polls/dist/django-polls-0.1.tar.gz*

------------------------------------------------------------------------------------
### Using your own package

s1. To install the package, use pip

`./`

*at virtualenv remove '--user', install at /devweb/lib/python3.6/site-packages/polls/...*

    > pip install [--user] django-polls/dist/django-polls-01.tar.gz

s2. runserver

`mysite/`

    > python manage.py runserver

s3. as uninstall

`./`

    > pip uninstall django-polls

### Installing Python packages with virtualenv 

[virtualenv](https://virtualenv.pypa.io/en/stable/)