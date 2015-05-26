Markdown content in Jekyll
==========================

`Jekyll <http://jekyllrb.com/>`_ is a static site engine that's specialized for blog authoring. Although Jekyll can support content in many markup formats, it's most commonly used to render `markdown <http://daringfireball.net/projects/markdown/>`_.

Specifying a Layout
-------------------

Deconst allows each page to be assocated with :ref:`a layout <control-layout>` from the control repository by specifying a *key*. If a page names a Jekyll layout in its frontmatter, that layout name will also be used as its layout key for deconst. If you wish to use different keys for the Jekyll build, specify a ``deconst_layout`` key instead.

Frontmatter
-----------

Certain frontmatter keys have meaning to both Jekyll and Deconst. These include:

 * ``title`` will be made available to site layouts as ``{{{ metadata.title }}}``. Usually, this will be used within an ``<h1>`` element on the page and as the browser title.
 * ``categories`` is a YAML list of strings used to manually identify related content. These are meant to be chosen by hand from a small, fixed list of possibilities. Site layouts should generally render categories rather than tags.
 * ``tags`` is another YAML list of strings. These are more ad-hoc, but may be manipulated by the content service. Additional tags may be appended by latent semantic indexing or normalization processes.
 * ``author`` should be set to the name of a blog post's author.
 * ``bio`` may be set to a short paragraph introducing the author.
 * ``date`` is used to specify the publish date of a blog post in **YYYY-mm-dd HH:MM:SS** format.
 * ``disqus`` is a subdictionary used to control the inclusion of a Disqus comment field, if supported by the layout. It should have two subkeys: ``short_name``, as provided by your Disqus account, and ``mode``, which may be either ``count`` or ``embed`` to control the Disqus script injected into this page.

All of these are optional and only have meaning if the equivalent metadata attributes are used in the page's Deconst layout.

Assets
------

To properly include images or other static assets in your Jekyll content, use the `jekyll-assets plugin <http://jekyll-assets.github.io/jekyll-assets/>`_. The Jekyll preparer will hook the assets plugin and override it to properly submit assets to the content service.

 1. Add an ``_assets/images`` directory to your Jekyll repository. Place any image assets that you reference within this directory.
 2. Reference images within a post or a page with the ``asset_path`` Liquid tag.

With Markdown:

.. code-block:: text

  [alt text]({% asset_path image-path.png %})

Or with raw HTML:

.. code-block:: html

  <img src="{% asset_path image-path.png %}" alt="alt text">

Plugins and Dependencies
------------------------

If you use Jekyll plugins that rely on other gems, you'll need to add a ``Gemfile`` to the root directory of your content repository to declare your dependencies.

It isn't necessary to list either jekyll or jekyll-assets as explicit dependencies, because the preparer already includes them. If you do include them, the versions you declare will be ignored during preparation, anyway. It won't harm the build to do so, though.

.. code-block:: ruby

  source 'https://rubygems.org'

  gem 'stringex'

Be sure to run ``bundle install`` to generate an equivalent ``Gemfile.lock``, to ensure that the versions of your dependencies are consistent from build to build.

Continuous Deployment
---------------------

To configure the continuous deployment process for your content repository, place a file called ``.travis.yml`` with the following contents at the root of your repository.

.. code-block:: yaml

  ---
  language: ruby
  ruby:
  - "2.2.0"
  install:
  - git clone https://github.com/deconst/preparer-jekyll.git /tmp/preparer-jekyll
  - cd /tmp/preparer-jekyll && rake install
  script:
  - deconst-preparer-jekyll

Now, visit `Travis <https://travis-ci.org/>`_ and choose "Accounts" from the drop-down menu in the upper right:

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
