# review-app

This is the Django project which provides the App Service for the Review App.

## Containers

From this repository, 2 Docker containers are built:

### onetouchapps/review-app

This is a production-ready Apache process hooked up to the Django WSGI module.

### onetouchapps/review-static

This is the collected static files from across the Django project, packaged
into a folder structure and served up by nginx.

## Development mode

### Getting set up

You will need these 2 installed before you can do any dev work with this repo:

  - [Docker](https://docs.docker.com/installation/#installation)

  - [Fig](http://www.fig.sh/install.html)

After you've installed these, you will need to run

``fig build``

You can then run a dev server, or run browser (BDD) tests, or run backend
(Django nose) tests.

### Running a dev server

You don't need to install any Python stuff or database things; to run a dev
server you:

``fig up``

This will run a dev server, accessible in a browser from your local machine
at port 8000 on your local Docker host:

  - if you are running a (U|Li)nix system, the address will most likely be
    127.0.0.1 [you will also likely have to ``alias docker='sudo docker'`` for
    scripts in this repo to work]

  - if you are running a Mac, the address can be obtained with ``boot2docker
    ip``

Note that this will use a local sqlite3 database file on your machine, inside
the repo, to allow for persistence of your test / dev data.

### Running BDD Browser tests

To run the BDD tests which will drive the code, written in Jasmine and
executed (in dev) against the Django dev server, you need:

  1. [node](http://nodejs.org/download/) installed

  2. all the node depenencies set up locally: ``npm install``

Then you can run the BDD tests with:

``developer_test_scripts/run_browser_tests.sh``

This script preps a Postgres database and takes care of starting and stopping
the Django dev server (in its Fig container). Because it runs a fresh Postgres
container every time, it will not interfere with your usual dev data. *[Note:
you do not need Postgres installed on your host]*

Note also that before it will run, you will need the APP_SERVICE_URL
environment variable set to point to the location where the dev server normally
runs (e.g. ``http://192.168.59.103:8000``) See the previous section for how to
find the IP address.

### Running Django-only backend tests

Django Nose tests may be run from your host with:

``developer_test_scripts/run_backend_tests.sh``

## Logs from dev and dev testing

Like the web servers in the containers, the Django logging system has also
been configured to send all output to stdout on the container. Therefore you
can capture all the log output from all the app container (and the postgres
container during browser tests) by running a Logspout container and directing
its output to the log aggregator of your choice (e.g. papertrail):

``docker pull progrium/logspout``
``docker run -d -v=/var/run/docker.sock:/tmp/docker.sock progrium/logspout syslog://logs.papertrailapp.com:12345`` (where "12345" is yout port at papertrail)

## Environment Variables

For the various phases of this project's delivery cycle to function correctly,
certain environment variables need to be made available to the running
context.

Note: This is not an exhaustive list of env vars at play in this set up, this
is a list of the vars for which you are responsbible for making available in
the named contexts here.

### Django tests and Django dev server

  All env vars are handled for you in the various fig*.yml files

### Developer Browser tests

  - *APP_SERVICE_URL* should point to the open dev server port on your Fig
    container's IP address (If you are running boot2docker you can get this
    with ``boot2docker ip``)

### CI environment

  - *APP_SERVICE_URL* should point to the open dev server port on the app
    container (most likely ``http://127.0.0.1`` in CI environment)

  - *AWS_ACCESS_KEY_ID* and *AWS_SECRET_ACCESS_KEY* IAM keypair with the
    rights to run ``create_deployment`` API tasks at OpsWorks

  - *DOCKERHUB_API_URL*, *DOCKERHUB_EMAIL* and *DOCKERHUB_AUTH* URL and
    credentials for an account with rights to push to both the review-app and
    review-static repositories at Dockerhub

  - *DOCKERHUB_APP_IMAGE_NAME* and *DOCKERHUB_APP_IMAGE_TAG* the name and tag
    which will get pushed to DockerHub by the CI service, and which will get
    pulled from DockerHub by the OpsWork deployment process. Typically
    "onetouchapps/review-app" and "latest", respectively

  - *DOCKERHUB_STATIC_IMAGE_NAME* and *DOCKERHUB_STATIC_IMAGE_TAG* the name and
    tag which will get pushed to DockerHub by the CI service, and which will
    get pulled from DockerHub by the OpsWork deployment process. Typically
    "onetouchapps/review-static" and "latest", respectively

  - *DOCKER_LOG_ROUTER_IMAGE_NAME* the DockerHub image which contains the log
    router (which is currently Logspout)

  - *LOG_AGGREGATE_HOST* and *LOG_AGGREGATE_PORT* the hostname and port of the
    log aggregation service (e.g. papertrail)

  - *LOG_ROUTER_SYSTEM_NAME* this is the name given at the start of each
    routed log entry

  - *OW_APP_STACK_ID* and *OW_APP_APP_ID* IDs for the OpsWork app which runs
    the review-app containers and the deploy of which CI will trigger

  - *OW_STATIC_STACK_ID* and *OW_STATIC_APP_ID* IDs for the OpsWork app which
    runs the review-static containers and the deploy of which CI will trigger

### Production environment

  - *DJANGO_ALLOWED_HOST* This is required to match the HOST header of http
    requests which are to be serviced by the WSGI (Django) app

  - *DJANGO_STATIC_URL* should be set to the full prefix of the delivery URL
    for static files (for example ``https://static.foo.com/``). If not present
    in the env vars, defaults to ``/static/``.

  - *DJANGO_DB_HOST*, *DJANGO_DB_NAME*, *DJANGO_DB_USER*, *DJANGO_DB_PASSWORD*
    location of and credentials for a (Postgres) database instance for this app
