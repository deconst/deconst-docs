Coordinating a Deconst Site
===========================

A Deconst *site coordinator* has several responsibilities, including management of the site's high-level information architecture, graphic design, and maintenance of the layouts and assets for each domain within the site.

The Control Repository
----------------------

Every deconst installation is configured to point to a single **control repository**, a version-controlled repository that's used to manage site-wide concerns. It's a GitHub repository containing mostly plain-text files that you can edit however you wish, even directly with GitHub's web editor! Deconst uses :ref:`webhooks and a continuous integration "build" <control-webhooks>` to bring changes that are merged into the ``master`` branch directly to the live site. Keeping this in mind, there are many workflows that you can adopt to manage your site, but the `GitHub Flow <https://guides.github.com/introduction/flow/>`_ is a simple and well-understood starting point that you can use.

The control repository is expected to include certain contents:

 * At least one :ref:`content mapping file <control-map>` that tells Deconst which content to display where.
 * :ref:`Layout templates <control-layout>` that give individual pages visual identity.
 * :ref:`Layout mapping files <control-layout-map>` that specify which layout template should be used to render a specific page.
 * :ref:`Global assets <control-assets>` such as stylesheets, JavaScript files, or images that are referenced by the layout templates.

.. _control-map:

Content Mapping Files
---------------------

To include a new content repository on a deconst site, you'll need to add it to one of the *content mapping files* within the control repository. Content mapping files are plain-text files found anywhere within your control repository that have a filename ending in ``.map.txt``.

Deconst will let you organize content mapping files however you wish. The way that you name your mapping files and how you arrange them within the repository is for *your* benefit. It's best to start with a single file, but as your site grows, you'll want to decide on some convention for this among your team, so that everyone knows where to add new mappings or find specific existing ones.

Mapping files work by placing a subtree of content from a certain content repository at a subtree of a domain within the overall Deconst site. For example, suppose we have a content repository at ``https://github.com/user/book1`` that creates the following pages:

.. code-block:: text

  introduction
  chapter-1/getting-started
  chapter-1/and-then

And another content repository at ``https://github.com/user/book2`` that creates these pages:

.. code-block:: text

  welcome
  chapter-1/the-basics
  chapter-1/more-detail

If we create mapping entries that map ``library/my-book/`` to ``https://github.com/user/book1/`` and ``library/another-book/`` to ``https://github.com/user/book2/``, both on the domain *books.horse*, these pages will be available at the following URLs:

.. code-block:: text

  https://books.horse/library/my-book/introduction
  https://books.horse/library/my-book/chapter-1/getting-started
  https://books.horse/library/my-book/chapter-1/and-then
  https://books.horse/library/another-book/welcome
  https://books.horse/library/another-book/chapter-1/the-basics
  https://books.horse/library/another-book/chapter-1/more-detail

The **longest prefix** that matches an incoming URL is used to decide which mapping is used to locate the content to render. For example, if ``/base/`` is mapped to ``https://github.com/user/base``, but ``/base/subpage/`` is mapped to ``https://github.com/user/subpage``, requests will be mapped as follows:

  * **https://books.horse/base** will render **https://github.com/user/base/**.
  * **https://books.horse/base/something** will render **https://github.com/user/base/something**.
  * **https://books.horse/base/subpage** will render **https://github.com/user/subpage** because the ``/base/subpage/`` mapping now takes precendence, *even if https://github.com/user/base/subpage exists within that content repository.*
  * **https://books.horse/base/subpage/anything** will render **https://github.com/user/subpage/anything**.

.. note::

  Technically, content mappings work with the :term:`content IDs` that are produced by the :term:`preparer` that "builds" each content repository. To do more complicated mappings, it's helpful to know the :ref:`details of exactly how they're produced <control-content-ids>`, but to get started you can assume that the content repository's URL is a prefix for the content IDs of all of its content.

Changes to the content mapping files will take effect as soon as they're merged into the ``master`` branch of the control repository. Huzzah for continuous delivery!

.. _control-map-syntax:

Content Map Syntax
^^^^^^^^^^^^^^^^^^

The content mapping file syntax looks like this:

.. code-block:: ini

  # Any line that begins with a "#" is considered a comment.
  # Lines that are empty, or that contain only whitespace, will be ignored.

  [books.horse]
  / https://github.com/user/library-welcome

  # The books that I've written
  /library/my-book/ https://github.com/user/book1/
  /library/another-book/ https://github.com/user/book-2/

  [nextbigthing.io]
  / https://github.com/someone-else/nextbigthing-index/
  /product https://github.com/someone-else/product-sdk/

Before any mappings are introduced within the file, you must specify the current *domain* by naming it within square brackets (``[ ]``). You can specify multiple domains within a single file if you wish.

Each mapping consists of the *presented URL prefix* and the *content ID prefix* on a single line, separated by whitespace.

It's an error to map the exact same prefix on the same domain more than once. This is to prevent you from accidentally clobbering your own mappings by mistake once your site is mapped across many mapping files!

.. note::

  End each URL prefix and each content ID prefix with a trailing slash. The mapping service is smart enough to do the right thing for content at the root of each mapping: the URL **https://books.horse/library/my-book** will render the content at **https://github.com/user/book1/**, not **https://github.com/user/library-welcome/my-book**.

Adding and Removing Content Repositories
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When you add any mappings that use a new content repository to the content map files, Deconst will automatically register the webhooks that are necessary to notice any changes and send you a pull request that adds the necessary ``.travis.yml`` configuration file.

You can remove a content repository by removing all mappings that reference its content IDs. You will need to delete the repository's webhooks and disable its Travis build manually.

.. warning::

  This doesn't actually work yet; I've been doing it by hand so far. There's a `GitHub issue <https://github.com/deconst/deconst-docs/issues/8>`_ open for it, though.

.. _control-layout:

Creating a Layout
-----------------

.. _control-layout-syntax:

Layout Template Syntax
^^^^^^^^^^^^^^^^^^^^^^

.. _control-layout-map:

Mapping Layouts to Pages
^^^^^^^^^^^^^^^^^^^^^^^^

.. _control-assets:

Layout Assets
-------------

Troubleshooting
---------------

.. _control-webhooks:

Webhooks and Integration
------------------------

Advanced Topics
---------------

.. _control-content-ids:

Content IDs
^^^^^^^^^^^

Strictly speaking, the way that :term:`content IDs` are assigned is an arbitrary decision made by the :term:`preparer` that's configured on that repository. However, by convention, they follow a pattern:

  *base URL of the content repository* + *subpath of the rendered page*

For example, suppose that we have a content repository hosted at https://github.com/deconst/deconst-docs that contains Sphinx documentation. A page within that repository that renders at *writing-docs/coordinator* would be assigned a content ID of ``https://github.com/deconst/deconst-docs/writing-docs/coordinator``.
