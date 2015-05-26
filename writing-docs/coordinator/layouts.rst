.. _control-layout:

Layouts
-------

The visual identity, navigation, and HTML boilerplate used for each page rendered by Deconst is provided by a set of *layout templates* that are managed within the control repository. Layout templates are written in `Handlebars <http://handlebarsjs.com/>`_ syntax and must be placed in a directory called ``layouts`` at the root of the control repository. Layout template files should usually end with an ``.hbs`` extension.

.. _control-layout-syntax:

Layout Syntax Extensions
^^^^^^^^^^^^^^^^^^^^^^^^

There are several special helpers and variables that are made available to each layout as it's rendered. Use these to indicate where content from the :term:`metadata envelope` is to be placed.

 * ``{{{ envelope.body }}}``: This one is very important: it'll be replaced by the actual content of the page.
 * ``{{ envelope.title }}``: The name of the page, if one has been provided.
 * ``{{{ assets.js_xyz_url }}}``: The final https CDN URL of the JavaScript asset bundle from the "xyz" subdirectory. See :ref:`the assets section <control-layout-assets>` for more details.
 * ``{{{ assets.css_xyz_url }}}``: The same thing a CSS asset bundle.
 * ``{{{ assets.image_xyz_jpg_url }}}``: The asset URL for an image asset.
 * ``{{{ assets.font_xyz_tff_url }}}``: The asset URL for a font asset.

Additionally, Deconst accepts a small number of preprocessing directives that you can use to eliminiate redundancy among your templates.

 * ``[@ path/to/outer.hbs @]`` will embed the current layout within another layout. This directive **must** be the first line in the template file. This layout will be placed whereever the *{{{ envelope.body }}}* directive is found within the outer layout -- this allows you to also use the outer template directly, if you so choose. If *envelope.body* is never included, a warning will be emitted.
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

Once you have :ref:`layouts to render <control-layout-syntax>`, you'll need to specify which layout will be used for any specific page.

Because the :ref:`content mapping service <control-map>` only operates on *subtrees* of content, not specific pages, Deconst doesn't have enough context for you to fully map layouts to individual pages. (You don't actually want to, anyway: if it was done that way, authors would need to update the control repository for every single page!) Instead, you manage the mapping of **layout keys** to Handlebars layouts within a given domain and URL prefix, and the content repositories offer mechanisms to set a layout key on each page.

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
