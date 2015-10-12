.. _control-template:

Templates
---------

The visual identity, navigation, and HTML boilerplate used for each page rendered by Deconst is provided by a set of *templates* that are managed within the control repository. Templates are written in `Nunjucks <https://mozilla.github.io/nunjucks/>`_ syntax and must be placed in a subdirectory of ``templates`` named after the domain in which they're used. Layout template files should usually end with an ``.html`` extension.

.. _control-template-syntax:

Layout Syntax Extensions
^^^^^^^^^^^^^^^^^^^^^^^^

There are several special helpers and variables that are made available to each layout as it's rendered. Use these to indicate where content from the :term:`metadata envelope` is to be placed.

 * ``{{ deconst.content.envelope.body }}``: This one is very important: it'll be replaced by the actual content of the page.
 * ``{{ deconst.content.envelope.title }}``: The name of the page, if one has been provided.
 * ``{{{ deconst.assets.js_xyz_url }}}``: The final https CDN URL of the JavaScript asset bundle from the "xyz" subdirectory. See :ref:`the assets section <control-layout-assets>` for more details.
 * ``{{{ deconst.assets.css_xyz_url }}}``: The same thing a CSS asset bundle.
 * ``{{{ deconst.assets.image_xyz_jpg_url }}}``: The asset URL for an image asset.
 * ``{{{ deconst.assets.font_xyz_tff_url }}}``: The asset URL for a font asset.

As a complete example, this set of layouts provides basic HTML5 boilerplate, a common sidebar that may be shared among several layouts, and a specialized layout for blog posts.

``templates/books.horse/_layouts/base.html``

.. code-block:: html

  <!doctype html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>{{ deconst.content.envelope.title }}</title>
      <link href="{{ deconst.assets.css_books_url }}" rel="stylesheet" type="text/css">
    </head>
    <body>
      {% block content %}{{ deconst.content.envelope.body }}{% endblock %}

      <script src="{{ deconst.assets.js_books_url }}"></script>
    </body>
  </html>

``templates/books.horse/_includes/sidebar.html``

.. code-block:: html

  <ul class="sidebar">
    <li>First Item</li>
    <li>Second Item</li>
    <li>Third Item</li>
  </ul>

``templates/books.horse/blog-post.html``

.. code-block:: html

  {% extends "_layouts/base.html" %}

  {% block content %}
    <div class="blog">
      <h1>This is a Blog Post</h1>

      <div class="content">
        {{ deconst.content.envelope.body }}
      </div>

      {% include "_includes/sidebar.html" %}
    </div>
  {% endblock %}

.. _control-template-map:

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
