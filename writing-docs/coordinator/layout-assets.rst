.. _control-layout-assets:

Assets
------

Raw HTML isn't very exciting on its own. To make a site look good and behave sensibly, you'll need to include assets: CSS, JavaScript, images and possibly fonts. Deconst sites store these with a specific structure beneath the ``assets`` subdirectory of the control repository.

CSS and JavaScript assets are grouped into *bundles*: sets of files that will be preprocessed together in some way to produce a single, optimized artifact, published on the CDN. Each bundle corresponds to, and is takes its name from, one immediate subdirectory of the ``assets/js`` or ``assets/less`` directories.

CSS
^^^

Deconst supports the generation of CSS from source files written in `Less <http://lesscss.org/>`_. Write your .less files within a subdirectory of ``assets/less`` with a name that will be used to name that bundle. You **must** include a file called ``main.less`` to serve as the entry point for that bundle. From ``main.less``, you can ``@include`` whatever other files you wish to include.

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

JavaScript files should be placed in a subdirectory of ``assets/js`` with a directory name that will become the name of your bundle. You **must** include a file called ``index`` that lists the JavaScript files that should be included in the bundle,

Given a JavaScript bundle containing:

.. code-block:: text

  assets/js/books/index
  assets/js/books/app.js
  assets/js/books/components/dropdown.js

Where the "index" file contains:

.. code-block:: text

  components/dropdown.js
  app.js

These files will be concatenated (including ``dropdown.js`` first, then ``app.js``), minified, fingerprinted, and published to the CDN. The resulting public https URL will be made available to your layouts as a Handlebars variable called ``{{ assets.js_books_url }}``. You can then include it within the appropriate layouts using something like:

.. code-block:: html

  <script type="text/javascript" src="{{ assets.js_books_url }}"></script>

Images and Fonts
^^^^^^^^^^^^^^^^

All files beneath ``assets/images`` and ``assets/fonts`` will be published as-is to the CDN. Their https URLs can then be used by both the layouts and the Less stylesheets.

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

.. warning::

  Storing assets within the control repository will add a bit of overhead to *every* page render! Only put assets here that are used by your layout templates. Content repository assets should be added to the content repository that uses them.
