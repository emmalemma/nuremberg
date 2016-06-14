# HLS Nuremberg Trials Project

> This is a Django client for the digital archives of the Nuremberg Trials Project maintained by the Harvard Law Library.
> It is intended as a open-access web app for scholars, researchers, and members of the public, exposing the digitized documents, full-text trial transcripts, and rich search features for the same in a friendly, modern interface.

## Project Structure

The project is organized into several feature-oriented modules ("apps" in Django parlance). Each module includes all URL routing, model and view code, tests, templates, JavaScript code, and static assets for the corresponding feature set:

- `nuremberg`: Top-level namespace for organizational purposes only.
  - `.core`: Top-level URL routing, test frameworks, base templates and middleware, and site-wide style files.
  - `.settings`: Environment-specific Django settings files.
  - `.content`: Files for static pages with project information, etc.
  - `.documents`: Files for displaying digitized document images.
  - `.transcripts`: Files for full-text transcripts and OCR documents.
  - `.search`: Files for the search interface and API.

This document covers the following topics:

- [Setting up a development environment](#setup)
- [Running the test suite](#testing)
- [Configuring project settings](#project-settings)
- [Packaging static assets](#static-assets)

### Setup

To run the app in a development environment, you'll need:

- Python 3.5
- Python dependencies
- MySQL

#### Python

You may want to run Python in a virtualenv wrapper to keep your dependencies clean.

```bash
virtualenv -p python3 venv
source venv/bin/activate
```

#### Dependencies
```bash
pip install -r requirements.txt
```

If you will be running the test suite, you need to install test dependencies:

```bash
pip install -r nuremberg/core/tests/requirements.txt
```

In order to compile static assets (which is configured to happen automatically while running the development server), you will need to install `lessc` from npm:

```bash
npm install -g less
```

#### MySQL

The easiest way to set up the dev database is loading the test fixtures:

```bash
mysql -uroot -e "CREATE DATABASE IF NOT EXISTS nuremberg_dev"
mysql -uroot -e "CREATE USER nuremberg; GRANT ALL ON nuremberg_dev.* TO nuremberg"
mysql -unuremberg nuremberg_dev < nuremberg/core/tests/data.sql
```

Again, if you want to run the test suite, you should do the same for the test database. (Our tests run on a persistent database):

```bash
mysql -uroot -e "CREATE DATABASE IF NOT EXISTS test_nuremberg_dev"
mysql -uroot -e "GRANT ALL ON test_nuremberg_dev.* TO nuremberg"
mysql -unuremberg test_nuremberg_dev < nuremberg/core/tests/data.sql
```

#### Run the server

You should now be all set to run the local server:

```bash
python manage.py runserver
```

Then visit [localhost:8000](http://localhost:8000).

### Testing

Tests in the project are generally high-level integration acceptance tests that exercise the full app stack in a deployed configuration. Since the app has the luxury of running off of a largely static dataset, the test database is persistent, greatly speeding up setup and teardown time.

#### Running tests

Make sure you have installed test dependencies and initialized the test database in [Setup](#setup) above. Then simply:

```bash
py.test
```

Pytest is configured in `pytest.ini` to run all files named `tests.py`.

There is also a Selenium/PhantomJS suite to test the behavior of the document viewer front-end. These tests take a while to run, don't produce coverage data, and are relatively unreliable, so they aren't run as part of the default suite. However they are still useful, as they exercise the full stack all the way through image downloading and preloading. They can be run explicitly when necessary:

```bash
py.test nuremberg/documents/browser_tests.py
```

The browser tests require PhantomJS to be installed (`npm install -g phantomjs`), and they generate screenshots in `/screenshots` to aid in debugging.


#### Coverage

Running the test suite will print a code coverage report to the terminal, as well as an interactive HTML report in `coverage/index.html`. Template code is included in the report. Coverage settings are configured in `.coveragerc`.

#### Pre-commit hook

To improve test compliance, there is a git pre-commit hook to run the test suite before each commit. It's self-installing, so just run:

```bash
bash ./nuremberg/core/tests/pre-commit-hook.sh
```

Now if any test fails, or test coverage is below 95%, the hook will cancel the commit.

### Project Settings

Environment-specific Django settings live in the `nuremberg/settings` directory, and inherit from `nuremberg.settings.generic`. The settings module is configured by the `DJANGO_SETTINGS_MODULE` environment variable; the default value is `nuremberg.settings.dev`.

Secrets (usernames, passwords, security tokens, nonces, etc.) should not be placed in a settings file or committed into git. The proper place for these is an environment variable configured on the host, and read via `os.environ`. If they must live in a `.py` file, they should be included in the environment settings file via an `import` statement and uploaded separately as part of the deployment process.

(The only exception to this is the defaults used in the dev environment.)

### Static Assets

#### CSS

CSS code is generated from `.less` files that live in `nuremberg/core/static/styles`. The styles are built based on Bootstrap 3 mixins, but don't bundle any Bootstrap code directly to ensure a clean semantic design.

Compilation is handled automatically by the `django-static-precompiler` module while the development server is running.

#### In Production

When deploying, you should run `manage.py compress` to bundle, minify and compress CSS and JS files, and `manage.py collectstatic` to move the remaining assets into `static/`. This folder should not be committed to git.

For deployment to Heroku, these static files will be served by the WhiteNoise server. In other environments it may be appropriate to serve them directly with Nginx or Apache. If necessary, the output directory can be controlled with an environment-specific override of the `STATIC_ROOT` settings variable.
