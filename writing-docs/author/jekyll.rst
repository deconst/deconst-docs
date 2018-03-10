Markdown content in Jekyll
==========================

`Jekyll <http://jekyllrb.com/>`_ is a static site engine that's
specialized for blog authoring. Although Jekyll can support content in
many markup formats, it's most commonly used to render `markdown
<http://daringfireball.net/projects/markdown/>`_.

Frontmatter
-----------

Certain frontmatter keys have meaning to both Jekyll and Deconst. These include:

 * ``title`` will be made available to site layouts as ``{{{
   metadata.title}}}``. Usually, this will be used within an ``<h1>``
   element on the page and as the browser title.

 * ``categories`` is a YAML list of strings used to manually identify related
   content. These are meant to be chosen by hand from a small, fixed list of
   possibilities. Site layouts should generally render categories rather than
   tags.

 * ``tags`` is another YAML list of strings. These are more ad-hoc, but may be
   manipulated by the content service. Additional tags may be appended by latent
   semantic indexing or normalization processes.

 * ``author`` should be set to the name of a blog post's author.

 * ``bio`` may be set to a short paragraph introducing the author.

 * ``date`` is used to specify the publish date of a blog post in
   **YYYY-mm-dd HH:MM:SS** format.

 * ``disqus`` is a subdictionary used to control the inclusion of a
   Disqus comment field, if supported by the layout. It should have
   two subkeys: ``short_name``, as provided by your Disqus account,
   and ``mode``, which may be either ``count`` or ``embed`` to control
   the Disqus script injected into this page.

All of these are optional and only have meaning if the equivalent
metadata attributes are used in the page's Deconst layout.

Assets
------

To properly include images or other static assets in your Jekyll
content, use the `jekyll-assets plugin
<http://jekyll-assets.github.io/jekyll-assets/>`_. The Jekyll preparer
will hook the assets plugin and override it to properly submit assets
to the content service.

 #. Add an ``_assets/images`` directory to your Jekyll repository.
    Place any image assets that you reference within this directory.

 #. Reference images within a post or a page with the ``asset_path``
    Liquid tag.

With Markdown:

.. code-block:: text

   ![alt text]({% asset_path image-path.png %})

Or with raw HTML:

.. code-block:: html

   <img src="{% asset_path image-path.png %}" alt="alt text">

Plugins and Dependencies
------------------------

If you use Jekyll plugins that rely on other gems, you'll need to add
a ``Gemfile`` to the root directory of your content repository to
declare your dependencies.

It isn't necessary to list either jekyll or jekyll-assets as explicit
dependencies, because the preparer already includes them. If you do
include them, the versions you declare will be ignored during
preparation, anyway. It won't harm the build to do so, though.

.. code-block:: ruby

   source 'https://rubygems.org'

   gem 'stringex'

Be sure to run ``bundle install`` to generate an equivalent
``Gemfile.lock``, to ensure that the versions of your dependencies are
consistent from build to build.
