.. _site-coordinator:

Coordinating a Deconst Site
===========================

A Deconst *site coordinator* has several responsibilities, including
management of the site's high-level information architecture, graphic
design, and maintenance of the layouts and assets for each domain
within the site.

The Control Repository
----------------------

Every deconst installation is configured to point to a single
**control repository**, a version-controlled repository that's used to
manage site-wide concerns. It's a GitHub repository containing mostly
plain-text files that you can edit however you wish, even directly
with GitHub's web editor!

While changes to assets will go live automatically after a short
delay, changes to content or template mappings requires administrator
action to take effect.

The control repository is expected to include certain contents:

 * At least one :ref:`content mapping file <control-map>` that tells
   Deconst which content to display where.

 * :ref:`Templates <control-template>` that give individual pages
   visual identity.

 * :ref:`Template mapping files <control-template-map>` that specify
   which template should be used to render a specific page.

 * :ref:`Global assets <control-template-assets>` such as stylesheets,
   JavaScript files, or images that are referenced by the layout
   templates.


.. toctree::

   mapping
   templates
   template-assets
   search
   control-advanced
