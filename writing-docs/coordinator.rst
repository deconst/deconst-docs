Coordinating a Deconst Site
===========================

A Deconst *site coordinator* has several responsibilities, including management of the site's high-level information architecture, graphic design, and maintenance of the layouts and assets for each domain within the site.

The Control Repository
----------------------

Every deconst installation is configured to point to a single **control repository**, a version-controlled repository that's used to manage site-wide concerns. It's a GitHub repository containing mostly plain-text files that you can edit however you wish, even directly with GitHub's web editor! Deconst uses :ref:`webhooks and a continuous integration "build" <webhooks>` to bring changes that are merged into the ``master`` branch directly to the live site. Keeping this in mind, there are many workflows that you can adopt to manage your site, but the `GitHub Flow <https://guides.github.com/introduction/flow/>`_ is a simple and well-understood starting point that you can use.

The control repository is expected to include certain contents:

 * At least one :ref:`content mapping file <control-map>` that tells Deconst which content to display where.
 * :ref:`Layout templates <control-layout>` that give individual pages visual identity.
 * :ref:`Layout mapping files <control-layout-map>` that specify which layout template should be used to render a specific page.
 * :ref:`Global assets <control-layout-assets>` such as stylesheets, JavaScript files, or images that are referenced by the layout templates.

.. toctree::

  mapping
  layouts
  layout-assets
  control-advanced
