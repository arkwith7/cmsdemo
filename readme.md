# Wagtail demo project

This is a demonstration project for the amazing [Wagtail CMS](https://github.com/wagtail/wagtail).

The demo site is designed to provide examples of common features and recipes to introduce you to Wagtail development. Beyond the code, it will also let you explore the admin / editorial interface of the CMS.

Note we do _not_ recommend using this project to start your own site - the demo is intended to be a springboard to get you started. Feel free to copy code from the demo into your own project.

### Wagtail Features Demonstrated in This Demo

This demo is aimed primarily at developers wanting to learn more about the internals of Wagtail, and assumes you'll be reading its source code. After browsing the features, pay special attention to code we've used for:

- Dividing a project up into multiple apps
- Custom content models and "contexts" in the "breads" and "locations" apps
- A typical weblog in the "blog" app
- Example of using a "base" app to contain misc additional functionality (e.g. Contact Form, About, etc.)
- "StandardPage" model using mixins borrowed from other apps
- Example of customizing the Wagtail Admin via _wagtail_hooks_
- Example of using the Wagtail "snippets" system to represent bread categories, countries and ingredients
- Example of a custom "Galleries" feature that pulls in images used in other content types in the system
- Example of creating ManyToMany relationships via the Ingredients feature on BreadPage
- Lots more

**Document contents**

- [Installation](#installation)
- [Next steps](#next-steps)
- [Contributing](#contributing)
- [Other notes](#other-notes)

# Installation

- [Docker](#setup-with-docker)
- [Virtualenv](#setup-with-virtualenv)

If you want to set up Wagtail locally instead, and you're new to Python and/or Django, we suggest you run this project on [Docker](#setup-with-docker) (whichever you're most comfortable with). Docker will help resolve common software dependency issues.
Developers more familiar with virtualenv and traditional Django app setup instructions should skip to [Setup with virtualenv](#setup-with-virtualenv).

## Setup with Docker

#### Dependencies

- [Docker](https://docs.docker.com/engine/installation/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Installation

Run the following commands:

```bash
git clone https://github.com/wagtail/mysite.git
cd mysite
docker compose up --build -d
```

Wait 10 seconds for the database setup to complete. Then run:

```bash
docker compose run app /venv/bin/python manage.py load_initial_data
```
If this fails with a database error, wait 10 more seconds and re-try. Finally, run:

```bash
docker compose up
```

The demo site will now be accessible at [http://localhost:8000/](http://localhost:8000/) and the Wagtail admin
interface at [http://localhost:8000/admin/](http://localhost:8000/admin/).

Log into the admin with the credentials `admin / changeme`.

**Important:** This `docker-compose.yml` is configured for local testing only, and is _not_ intended for production use.

### Debugging

To tail the logs from the Docker containers in realtime, run:

```bash
docker compose logs -f
```

## Setup with Virtualenv

You can run the Wagtail demo locally without setting up Vagrant or Docker and simply use Virtualenv, which is the [recommended installation approach](https://docs.djangoproject.com/en/3.2/topics/install/#install-the-django-code) for Django itself.

#### Dependencies

- Python 3.7, 3.8, 3.9, 3.10 or 3.11
- [Virtualenv](https://virtualenv.pypa.io/en/stable/installation/)
- [VirtualenvWrapper](https://virtualenvwrapper.readthedocs.io/en/latest/install.html) (optional)

### Installation

With [PIP](https://github.com/pypa/pip) and [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)
installed, run:

    mkvirtualenv wagtailmysite
    python --version

Confirm that this is showing a compatible version of Python 3.x. If not, and you have multiple versions of Python installed on your system, you may need to specify the appropriate version when creating the virtualenv:

    deactivate
    rmvirtualenv wagtailmysite
    mkvirtualenv wagtailmysite --python=python3.9
    python --version

Now we're ready to set up the bakery demo project itself:

    cd ~/dev [or your preferred dev directory]
    git clone https://github.com/wagtail/mysite.git
    cd mysite
    pip install -r requirements/development.txt

Next, we'll set up our local environment variables. We use [django-dotenv](https://github.com/jpadilla/django-dotenv)
to help with this. It reads environment variables located in a file name `.env` in the top level directory of the project. The only variable we need to start is `DJANGO_SETTINGS_MODULE`:

    cp mysite/settings/local.py.example mysite/settings/local.py
    echo "DJANGO_SETTINGS_MODULE=mysite.settings.local" > .env

To set up your database and load initial data, run the following commands:

    ./manage.py migrate
    ./manage.py load_initial_data
    ./manage.py runserver

Log into the admin with the credentials `admin / changeme`.


### Storing Wagtail Media Files on AWS S3

If you have deployed the demo site to Heroku or via Docker, you may want to perform some additional setup. Heroku uses an
[ephemeral filesystem](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem), and Docker-based hosting
environments typically work in the same manner. In laymen's terms, this means that uploaded images will disappear at a
minimum of once per day, and on each application deployment. To mitigate this, you can host your media on S3.

This documentation assumes that you have an AWS account, an IAM user, and a properly configured S3 bucket. These topics
are outside of the scope of this documentation; the following [blog post](https://wagtail.org/blog/amazon-s3-for-media-files/)
will walk you through those steps.

This demo site comes preconfigured with a production settings file that will enable S3 for uploaded media storage if
`AWS_STORAGE_BUCKET_NAME` is defined in the shell environment. All you need to do is set the following environment
variables. If using Heroku, you will first need to install and configure the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli). Then, execute the following commands to set the aforementioned environment variables:

    heroku config:set AWS_STORAGE_BUCKET_NAME=changeme
    heroku config:set AWS_ACCESS_KEY_ID=changeme
    heroku config:set AWS_SECRET_ACCESS_KEY=changeme

Do not forget to replace the `changeme` with the actual values for your AWS account. If you're using a different hosting
environment, set the same environment variables there using the method appropriate for your environment.

Once Heroku restarts your application or your Docker container is refreshed, you should have persistent media storage!

Running `./manage.py load_initial_data` will copy local images to S3, but if you set up S3 after you ran it the first
time you might need to run it again.

# Next steps

Hopefully after you've experimented with the demo you'll want to create your own site. To do that you'll want to run the `wagtail start` command in your environment of choice. You can find more information in the [getting started Wagtail CMS docs](https://docs.wagtail.org/en/stable/getting_started/index.html).

# Contributing

If you're a Python or Django developer, fork the repo and get stuck in! If you'd like to get involved you may find our [contributing guidelines](https://github.com/wagtail/mysite/blob/master/contributing.md) a useful read.

### Preparing this archive for distribution

If you change content or images in this repo and need to prepare a new fixture file for export, do the following on a branch:

```bash
./manage.py dumpdata --natural-foreign --indent 2 -e auth.permission -e contenttypes -e wagtailcore.GroupCollectionPermission -e wagtailcore.revision -e wagtailimages.rendition -e sessions -e wagtailsearch.indexentry -e wagtailsearch.sqliteftsindexentry -e wagtailcore.referenceindex -e wagtailcore.pagesubscription -e wagtailcore.modellogentry -e wagtailcore.pagelogentry > mysite/base/fixtures/mysite.json
prettier --write mysite/base/fixtures/mysite.json
```

Please optimize any included images to 1200px wide with JPEG compression at 60%. Note that `media/images` is ignored in the repo by `.gitignore` but `media/original_images` is not. Wagtail's local image "renditions" are excluded in the fixture recipe above.

Make a pull request to https://github.com/wagtail/mysite

# Other notes

### Note on demo search

Because we can't (easily) use ElasticSearch for this demo, we use wagtail's native DB search.
However, native DB search can't search specific fields in our models on a generalized `Page` query.
So for demo purposes ONLY, we hard-code the model names we want to search into `search.views`, which is
not ideal. In production, use ElasticSearch and a simplified search query, per
[https://docs.wagtail.org/en/stable/topics/search/searching.html](https://docs.wagtail.org/en/stable/topics/search/searching.html).

### Sending email from the contact form

The following setting in `base.py` and `production.py` ensures that live email is not sent by the demo contact form.

`EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'`

In production on your own site, you'll need to change this to:

`EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'`

and configure [SMTP settings](https://docs.djangoproject.com/en/3.2/topics/email/#smtp-backend) appropriate for your email provider.

### Ownership of demo content

All content in the demo is public domain. Textual content in this project is either sourced from Wikimedia (Wikipedia for blog posts, [Wikibooks for recipes](https://en.wikibooks.org/wiki/Cookbook:Table_of_Contents)) or is lorem ipsum. All images are from either Wikimedia Commons or other copyright-free sources.
