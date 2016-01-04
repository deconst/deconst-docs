Development Environment
=======================

Before you can contribute to Deconst development, you'll first need to prepare your local development machine with a few dependencies. We use OSX, but any operating system that can run Docker containers should be usable.

Prerequisities
--------------

Deconst packages its services and dependencies as Docker containers. This helps to minimize the number of `yaks that you need to shave <http://en.wiktionary.org/wiki/yak_shaving>`_ to get started, but you still need to shave the yak for Docker itself.

#. Follow the `installation guide for Docker <https://docs.docker.com/installation/#installation>`_ for your platform of choice.

    You'll know you're ready to continue when you can successfully execute the following from a terminal:

    .. code-block:: bash

      $ docker version
      Client version: 1.6.2
      Client API version: 1.18
      Go version (client): go1.4.2
      Git commit (client): 7c8fca2
      OS/Arch (client): darwin/amd64
      Server version: 1.6.2
      Server API version: 1.18
      Go version (server): go1.4.2
      Git commit (server): 7c8fca2
      OS/Arch (server): linux/amd64

#. We also use Docker Compose to orchestrate small numbers of local containers to make development more convenient. Follow the `installation guide for Docker Compose <https://docs.docker.com/compose/install/>`_.

#. To contribute, you'll also need a reasonable `git <https://git-scm.com/>`_ client. It's likely that you already have one: open a terminal and type ``git version`` to check.

Individual Service Development
------------------------------

Each Deconst service is developed in a separate GitHub repository within the `deconst organization <https://github.com/deconst>`_. Fork and clone the repository of the service you wish to contribute to.

To run the service, run the following from the top-level directory of your clone:

.. code-block:: bash

  docker-compose up

Compose will launch a container for the service you're focusing on right now, as well as any upstream services or infrastructure that it depends on, and link them all together correctly. You'll see the combined logs for all containers on your terminal. As you edit source code in your editor of choice, the service within the container will automatically reload with your changes, so you can explore the effects live.

.. note::

  Unless you're developing on Linux, which can run Docker containers natively, it's likely that the Docker containers you run actually live within a virtual machine. As a consequence, you won't be able to reach your services at "localhost", but rather some other IP. The exact IP depends on the way you installed docker. For example, if you're using ``docker-machine``, running ``docker-machine ip dev`` will show you the IP.

Although your local source changes will take effect immediately, you may need to periodically fetch newer versions of upstream containers, as development progresses on the other parts of the system. To ensure that you have the latest builds of each container, run ``docker-compose pull``. Also, if you need to change the service's dependencies, you may need to rebuild your working container with ``docker-compose build``.

Compose can also be used to launch its containers in the background (with ``docker-compose up -d``), explore logs for individual containers rather than aggregated, or run one-off processes in the context of any service container. Consult the `compose documentation <https://docs.docker.com/compose/cli/>`_ to see all of your options.

Each service's unit tests can also be executed within a Docker container for convenience. As a convention, the following script will launch the container and run all tests:

.. code-block:: bash

  script/test

Integration Testing
-------------------

To verify that the entire Deconst system works together, use the **integrated** repository. "Integrated" contains a compose file that executes a single "pod" of related deconst services on your local host, so you can test all of the services together.

Clone the deconst/integrated repository and run ``script/up`` to begin:

.. code-block:: bash

  git clone https://github.com/deconst/integrated.git deconst-integrated
  cd deconst-integrated

  # Customize your environment
  cp env.example env
  ${EDITOR} env

  # Launch all services
  script/up

While your services are alive, you can run ``script/add-sphinx``, ``script/add-jekyll``, and ``script/add-assets`` to invoke an appropriate :term:`preparer` and submit content to your local deconst system.

Alternative: Manual Setup of Development Environment
----------------------------------------------------

.. code-block:: bash

  # generate an API key for the content service
  APIKEY=$(hexdump -v -e '1/1 "%.2x"' -n 128 /dev/random)
  echo "Content Service Admin API Key:" $APIKEY

  # startup content service dependencies
  docker run -d --name elasticsearch elasticsearch:1.7
  docker run -d --name mongo mongo:2.6

  # build and deploy the content service
  cd {wherever you have the deconst/content-service}
  docker build --tag content-service:1.0.0 .
  docker run -d -p 9000:8080 \
                -e NODE_ENV=development \
                -e STORAGE=memory \
                -e MONGODB_URL=mongodb://mongo:27017/content \
                -e ELASTICSEARCH_HOST=http://elasticsearch:9200/ \
                -e ADMIN_APIKEY=$APIKEY \
                --link mongo:mongo \
                --link elasticsearch:elasticsearch \
                --name content \
                content-service:1.0.0 script/inside/dev

  # build and deploy the presenter service
  cd {wherever you have the deconst/presenter}
  docker build --tag presenter-service:1.0.0 .
  docker run -d -p 80:8080 \
                -e NODE_ENV=development \
                -e CONTROL_REPO_PATH=/var/control-repo \
                -e CONTROL_REPO_URL=https://github.com/j12y/nexus-control.git \
                -e CONTROL_REPO_BRANCH=setup_howtos \
                -e CONTENT_SERVICE_URL=http://content:8080 \
                -e PRESENTED_URL_PROTO=http \
                -e PRESENTED_URL_DOMAIN=support.rackspace.com \
                --link content \
                --name presenter \
                presenter-service:1.0.0 script/dev
