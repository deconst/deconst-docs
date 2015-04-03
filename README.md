[![Build Status](https://travis-ci.org/deconst/deconst-docs.svg?branch=master)](https://travis-ci.org/deconst/deconst-docs)

# Deconst documentation

Documentation for the *deconst* project: a continuous delivery pipeline for heterogenous documentation. It's also used to develop deconst itself, like some kind of documentation ouroboros.

## Building

To build this documentation standalone, run:

```bash
make html
```

## Running

This repository also includes a configuration file for *docker-compose*, to let you run the full system on a single host. To get started, you'll need to install:

 * [Docker](https://docs.docker.com/installation/#installation) to build and launch the containers.
 * [docker-compose](https://docs.docker.com/compose/install/) to manage the containers' configurations.

Then, you can build and run the service with:

```bash
export RACKSPACE_USERNAME=...
export RACKSPACE_APIKEY=...

docker-compose build && docker-compose up -d
```

To push content into your local system, you'll need to install the *preparer* with `pip`:

```bash
# If you're using virtualenv and virtualwrapper
mkvirtualenv deconst-preparer-rst --python=$(which python3)

# Install the preparer
pip install -e git+https://github.com/deconst/preparer-rst.git#egg=deconstrst

# Configure the preparer to submit to the locally running content service
export CONTENT_STORE_URL=http://$(boot2docker ip):9000/
export CONTENT_ID_BASE=https://github.com/deconst/deconst-docs/
```

Then build and publish content and assets with:

```bash
deconst-prepare-rst
```
