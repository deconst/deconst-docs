.. _control-template:

Templates
---------

The visual identity, navigation, and HTML boilerplate used for each page rendered by Deconst is provided by a set of *templates* that are managed within the control repository. Templates are written in `Nunjucks <https://mozilla.github.io/nunjucks/>`_ syntax and must be placed in a subdirectory of ``templates`` named after the domain in which they're used. Template files should usually end with an ``.html`` extension.

.. _control-template-syntax:

Template Syntax Extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^

There are several special helpers and variables that are made available to each template as it's rendered. Use these to indicate where content from the :term:`metadata envelope` is to be placed.

 * ``{{ deconst.content.envelope.body }}``: This one is very important: it'll be replaced by the actual content of the page.
 * ``{{ deconst.content.envelope.title }}``: The name of the page, if one has been provided.
 * ``{{{ deconst.assets.js_xyz_url }}}``: The final https CDN URL of the JavaScript asset bundle from the "xyz" subdirectory. See :ref:`the assets section <control-template-assets>` for more details.
 * ``{{{ deconst.assets.css_xyz_url }}}``: The same thing a CSS asset bundle.
 * ``{{{ deconst.assets.image_xyz_jpg_url }}}``: The asset URL for an image asset.
 * ``{{{ deconst.assets.font_xyz_tff_url }}}``: The asset URL for a font asset.

As a complete example, this set of templates provides basic HTML5 boilerplate, a common sidebar that may be shared among several templates, and a specialized template for blog posts.

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

Mapping Templates to Pages
^^^^^^^^^^^^^^^^^^^^^^^^^^

Once you have :ref:`templates to render <control-template-syntax>`, you'll need to specify which template will be used for any specific page. Deconst maps templates using a JSON **template mapping file** found within the control repository at ``config/routes.json``. The template mapping file uses regular expressions to apply templates to pages that are :ref:`currently mapped <control-map>` to any matching URL.

.. code-block:: json

  {
    "books.horse": {
      "routes": {
        "^/": "default.html",
        "^/blog/.*": "blog-post.html"
      }
    }
  }

Templates are specified as paths relative to the site's subdirectory of the ``templates/`` directory, so with these mappings:

#. The page ``https://books.horse/docs/info/`` will be rendered with the template at ``templates/books.horse/default.html``.
#. The page ``https://books.horse/blog/hello-world/`` will be rendered with the template at ``templates/books.horse/blog-post.html``.
