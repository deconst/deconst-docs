reStructuredText content in Sphinx
==================================

`Sphinx <http://sphinx-doc.org/contents.html>`_ is a documentation builder that assembles `reStructureText <http://docutils.sourceforge.net/rst.html>`_ source files into cohesive output that includes tables of contents, cross-references, and integrated navigation.

Specifying a Layout
-------------------

Deconst allows each page to be assocated with :ref:`a layout <control-layout>` from the control repository by specifying a *key*. Sphinx doesn't include the concept of applying a layout to an individual page. The Sphinx preparer works around this by recognizing a piece of `per-page metadata <http://sphinx-doc.org/markup/misc.html#file-wide-metadata>`_ in each file.

You can see which layouts are available within your instance by consulting :ref:`layout mapping files within the control repository. <control-layout>` To support a consistent experience across an entire deconst instance, you should coordinate your choice of layout with your instance's information architects.

To use a specific layout for a page, include the following as the *first line* of any ``.rst`` file:

.. code-block:: rst

  :deconstlayout: this-pages-layout-key

If most of the pages in your repository will be using a common layout, you can also specify a default layout key by adding the following to your ``conf.py`` file:

.. code-block:: python

  deconst_default_layout = "this-repos-default-layout-key"

Assets
------

To integrate properly with Deconst's asset pipeline:

 1. Place any images in an `_images` directory at the top level of your Sphinx documentation.
 2. Reference images in ``.rst`` files by including a ``.. image`` macro.

.. code-block:: rst

  .. image:: /_images/deconst-initial.png

Continuous Deployment
---------------------

To configure the continuous deployment process for your content repository, place a file called ``.travis.yml`` with the following contents at the root of your repository.

.. code-block:: yaml

  ---
  language: python
  python:
  - "3.4"
  install:
  - "pip install -e git+https://github.com/deconst/preparer-sphinx.git#egg=deconstrst"
  script:
  - deconst-preparer-sphinx

Now, visit `Travis <https://travis-ci.org/>`, and choose "Accounts" from the drop-down menu in the upper right:

.. image:: /_images/travis-account-menu.png

On the accounts page, locate the GitHub organization and repository for your content repository, and toggle the box to activate the integration:

.. image:: /_images/travis-enable-build.png

Next, click on the gear icon to navigate to the build's configuration, and set the following environment variables:

* ``CONTENT_ID_BASE`` is the common prefix that will be used to produce :term:`content IDs` for the rendered content. Set this to the URL of your GitHub repository.
* ``CONTENT_STORE_URL`` is the URL of the content store that the prepare should target. Consult with your site administrators for this value.
* ``CONTENT_STORE_APIKEY`` is an API key issued by the content store for your repository. Ask a site administrator to generate one of these for you.

.. image:: /_images/travis-envvars.png

.. note::

  Eventually, this will be configured for you automatically as soon as your content repository is mapped. For now, you'll need to do it by hand.
