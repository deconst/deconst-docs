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
 * :ref:`Global assets <control-layout-assets>` such as stylesheets, JavaScript files, or images that are referenced by the layout templates.

.. _control-map:

Content Mapping Files
---------------------

To include a new content repository on a deconst site, you'll need to add it to one of the **content mapping files** within the control repository. Content mapping files are plain-text files found anywhere within your control repository that have a filename ending in ``.map.txt``.

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

It's an error to map the exact same prefix on the same domain more than once. This is to prevent you from accidentally clobbering your own mappings by mistake once your site spans many mapping files! You'll see build errors in the Travis build for your pull request, along with any other syntax problems that were discovered.

.. note::

  End each URL prefix and each content ID prefix with a trailing slash. The mapping service is smart enough to do the right thing for content at the root of each mapping: the URL **https://books.horse/library/my-book** will render the content at **https://github.com/user/book1/**, not **https://github.com/user/library-welcome/my-book**.

Adding and Removing Content Repositories
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When you add any mappings that use a new content repository to the content map files, Deconst will automatically register the webhooks that are necessary to notice any changes and send you a pull request that adds the necessary ``.travis.yml`` configuration file.

You can remove a content repository by removing all mappings that reference its content IDs. You will need to delete the repository's webhooks and disable its Travis build manually.

.. warning::

  This doesn't actually work yet; I've been doing it by hand so far. There's a `GitHub issue <https://github.com/deconst/deconst-docs/issues/8>`_ open for it, though.

.. _control-layout:

Layouts
-------

The visual identity, navigation, and HTML boilerplate used for each page rendered by Deconst is provided by a set of *layout templates* that are managed within the control repository. Layout templates are written in `Handlebars <http://handlebarsjs.com/>`_ syntax and must be placed in a directory called ``layouts`` at the root of the control repository. Layout template files should usually end with a ``.hbs`` extension.

.. _control-layout-syntax:

Layout Syntax Extensions
^^^^^^^^^^^^^^^^^^^^^^^^

There are several special helpers and variables that are made available to each layout as it's rendered. Use these to indicate where content is to be rendered.

 * ``{{{ envelope.body }}}``: This one is very important: it'll be replaced by the actual content of the page.
 * ``{{ envelope.title }}``: The name of the page, if one has been provided.
 * ``{{{ assets.js_xyz_url }}}``: The final https CDN URL of the JavaScript asset bundle from the "xyz" subdirectory. See :ref:`the assets section <control-layout-assets>` for more detail.
 * ``{{{ assets.css_xyz_url }}}``: The same thing for CSS assets.
 * ``{{{ assets.image_xyz_jpg_url }}}``: The asset URL for an image asset.
 * ``{{{ assets.font_xyz_tff_url }}}``: The asset URL for a font asset.

Additionally, Deconst accepts a small number of preprocessing directives that you can use to eliminiate redundancy among your templates.

 * ``[@ path/to/outer.hbs @]`` will embed the current layout within another layout. This layout will be placed whereever the *{{{ envelope.body }}}* directive is found within the outer layout -- this allows you to outer template directly, if you so choose. This directive **must** be the first line in the template file.
 * ``[+ path/to/common.hbs +]`` will include the contents of another layout at this point within the current layout.

.. note::

  For both of these directives, the path to the other layout must be relative to the ``layouts/`` directory within the control repository.

As a complete example, this set of layouts provides basic HTML5 boilerplate, a common sidebar that may be shared among several layouts, and a specialized layout for blog posts.

``layouts/books.horse/boilerplate.hbs``

.. code-block:: html

  <!doctype html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>{{ envelope.title }}</title>
      <link href="{{ assets.css_books_url }}" rel="stylesheet" type="text/css">
    </head>
    <body>
      {{{ envelope.body }}}

      <script src="{{ assets.js_books_url }}"></script>
    </body>
  </html>

``layouts/books.horse/common/sidebar.hbs``

.. code-block:: html

  <ul class="sidebar">
    <li>First Item</li>
    <li>Second Item</li>
    <li>Third Item</li>
  </ul>

``layouts/books.horse/blog-post.hbs``

.. code-block:: html

  [@ books.horse/boilerplate.hbs @]

  <div class="blog">
    <h1>This is a Blog Post</h1>

    <div class="content">
      {{{ envelope.body }}}
    </div>

    [+ books.horse/common/sidebar.hbs +]
  </div>

.. _control-layout-map:

Mapping Layouts to Pages
^^^^^^^^^^^^^^^^^^^^^^^^

Once you have :ref:`layouts to render, <control-layout-syntax>`, you'll need to specify which layout will be used for any specific page.

Because the :ref:`content mapping service <control-map>` only operates on *subtrees* of content, not specific pages, Deconst doesn't have enough context for you to fully map layouts to individual pages. (You don't actually *want* to, anyway: if it was done that way, authors would need to update the control repository for every single page!) Instead, you manage the mapping of **layout keys** to Handlebars layouts within a given domain and URL prefix, and the content repositories offer mechanisms to set a layout key on each page.

Deconst maps layout keys using plain-text **layout mapping files** found within the control repository. Layout mapping files are identified by a filename suffix of ``.layout.txt``. Like content mapping files, you can split layout mappings across many files as your site grows, using whatever organization you wish.

The layout file syntax is almost identical to :ref:`the content mapping file syntax <control-map-syntax>`: the active domain must be named within square brackets (``[ ]``), then layout key mappings for that domain are listed as whitespace-separated lines. Here's an example:

.. code-block:: text

  [books.horse]

  # The three components are:
  #  path prefix; layout key; path to layout template, relative to "layouts/"
  / default shared/default.hbs
  / blog-post books/blog-post.hbs
  /other blog-post other/blog-post.hbs

With this layout file, any pages rendered on *books.horse* that name a layout key of "default" will use the layout ``shared/default.hbs``, and most pages that use "blog-post" will be rendered with ``books/blog-post.hbs``. However, any pages beneath ``other/`` that request a layout key of "blog-post" will use the layout ``other/blog-post.hbs``, instead.

 * **https://books.horse/**, which is mapped to content that uses the layout key *default*, will be rendered with ``shared/default.hbs``.
 * **https://books.horse/news/deconst-is-working**, with the layout key *blog-post*, will be rendered with ``books/blog-post.hbs``.
 * **https://books.horse/other/about**, with the layout key *default*, will still be rendered with ``shared/default.hbs``.
 * **https://books.horse/other/guest-post**, with the layout key *blog-post*, will use ``other/blog-post.hbs`` instead.

.. _control-layout-assets:

Assets
------

Raw HTML isn't very exciting on its own. To make a site look good and behave sensibly, you'll need to include assets CSS, JavaScript, images and possibly fonts. Deconst sites store these with a specific structure beneath the ``assets`` subdirectory of the control repository.

CSS and JavaScript assets are grouped into *bundles*: sets of files that will be preprocessed together in some way to produce a single, optimized artifact, published on the CDN. Each bundle corresponds to, and is named after, one immediate subdirectory of the ``assets/js`` or ``assets/less`` directories.

CSS
^^^

Deconst supports the generation of CSS from source files written in `Less <http://lesscss.org/>`_. Write your .less files within a subdirectory of ``assets/less`` named after the bundle. You **must** include a file called ``main.less`` to serve as the entry point for that bundle. From ``main.less``, you can ``@include`` whatever other files you wish to include.

Given a Less bundle containing:

.. code-block:: text

  assets/less/books/main.less
  assets/less/books/variables.less

These files will be transpiled, minified, and fingerprinted into a single stylesheet that is published to the CDN, and its public https URL will made available to your layouts as a Handlebars variable called ``{{ assets.css_books_url }}``.

These files will create a published asset that you can reference in your layouts with:

.. code-block:: html

  <head>
    <link href="{{ assets.css_books_url }}" rel="stylesheet" type="text/css">
  </head>

JavaScript
^^^^^^^^^^

JavaScript files should be placed in a subdirectory of ``assets/js`` named after the bundle. You **must** include a file called ``index`` that lists the JavaScript files that should be included in the bundle,

Given a JavaScript bundle containing:

.. code-block:: text

  assets/js/books/index
  assets/js/books/app.js
  assets/js/books/components/dropdown.js

Where the "index" file contains:

.. code-block:: text

  components/dropdown.js
  app.js

These files will be concatenated (including ``app.js`` before ``dropdown.js``), minified, fingerprinted, and published the CDN, and its public https URL will be made available to your layouts as a Handlebars variable called ``{{ assets.js_books_url }}``. You can then include it within the appropriate layouts using something like:

.. code-block:: html

  <script type="text/javascript" src="{{ assets.js_books_url }}"></script>

Images and Fonts
^^^^^^^^^^^^^^^^

All files beneath ``assets/images`` and ``assets/fonts`` will be published as-is to the CDN. Their https URLs will then be made available to both the layouts and the Less stylesheets.

Given the following asset files:

.. code-block:: text

  assets/images/banner.jpg
  assets/images/detail.png
  assets/fonts/SomeFont.ttf

The corresponding final URLs will be made available to:

 * Layouts, as ``{{ image_banner_jpg_url }}``, ``{{ image_detail_jpg_url }}``, and ``{{ font_SomeFont_ttf_url }}``; and
 * Less stylesheets, as the variables ``@image_banner_jpg_url``, ``@image_detail_jpg_url``, and ``@font_SomeFont_ttf_url``.

Bower
^^^^^

`Bower <http://bower.io/>`_ is a package management system for front-end components. It can be used to include specific versions of popular libraries like `Bootstrap <http://getbootstrap.com/>`_ or `jQuery <https://jquery.com/>`_ without needing to vendor everyone else's code.

If any CSS or JavaScript bundle directory contains a file called ``bower.json``, bower will be invoked first to install any dependencies declared that way. You can reference bower-installed files by using their full ``bower_component/`` paths in the JavaScript index file or an ``@include`` directive.

.. warning:

  Storing assets within the control repository will add a bit of overhead to *every* page render! Only put assets here that are used by your layout templates. Content repository assets should be added to the content repository that uses them.

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
