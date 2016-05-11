reStructuredText content in Sphinx
==================================

`Sphinx <http://sphinx-doc.org/contents.html>`_ is a documentation builder that assembles `reStructureText <http://docutils.sourceforge.net/rst.html>`_ source files into cohesive output that includes tables of contents, cross-references, and integrated navigation.

Deconst uses native Sphinx code as much as possible, which means that you can mostly use write regular Sphinx documentation, even using extensions or custom directives, without worrying too about the Deconst integration. Exceptions to this are described below.

Assets
------

To integrate properly with Deconst's asset pipeline:

 1. Place any images in an `_images` directory at the top level of your Sphinx documentation.
 2. Reference images in ``.rst`` files by including a ``.. image`` macro.

.. code-block:: rst

  .. image:: /_images/deconst-initial.png

Tables of contents
------------------

Special per-page metadata
-------------------------

Sphinx offers page-level customization by reading `per-page metadata <http://www.sphinx-doc.org/en/stable/markup/misc.html#file-wide-metadata>`_ that may be present on each page. Certain fields can be used to customize Deconst's output.

deconsttitle
^^^^^^^^^^^^

If present, a ``:deconsttitle:`` field will be used as the page title within Deconst templates rather than the one that Sphinx assigns each document, which is always the top-level heading.

.. code-block:: rst

  :deconsttitle: Custom Title

  This heading will appear on the page, but not in the title
  ==========================================================

deconstcategories
^^^^^^^^^^^^^^^^^

Specify one or more categories to apply to an individual page with the ``:deconstcategories:`` field. The field's value is split on commas each each element is trimmed of whitespace.

.. code-block:: rst

  :deconstcategories: one, two

Categories redundant with repository-global ones will be deduplicated.

deconstunsearchable
^^^^^^^^^^^^^^^^^^^

Exclude a page from search results by marking it with a ``:deconstunsearchable:`` item. This *overrides* the :ref:`deconst_default_unsearchable` repository-wide setting for this document.

.. code-block:: rst

  :deconstunsearchable: true

Other metadata
^^^^^^^^^^^^^^

Any other fields included here are available to :ref:`template authors <control-template>` within the ``deconst.content.envelope.meta`` structure. Co-ordinate with your template designers to ascribe whatever meaning to other fields that you wish!

conf.py settings
----------------

Repository-wide settings for Sphinx are managed by a ``conf.py`` file at the root of your Sphinx content. Deconst uses several custom settings within this file for its global configuration as well.

builder
^^^^^^^

Deconst supports two distinct **builders** that alter the way that envelopes are generated, roughly corresponding to Sphinx's serial (``make html``) and single-page (``make singlehtml``) HTML builders. The ``deconst-single`` builder assembles all content from the repository into a single page, while the ``deconst-serial`` builder creates a different page for each ``.rst`` document.

The ``deconst-serial`` builder is the default. To use the single builder instead, set the ``builder`` variable within your ``conf.py``.

.. code-block:: python

  builder = 'deconst-single'
  # OR:
  builder = 'deconst-serial'

deconst_default_unsearchable
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To exclude all envelopes within a content repository from search indexing, set ``deconst_default_unsearchable`` to ``True``:

.. code-block:: python

  deconst_default_unsearchable = True

Notice that this may still be overridden by individual envelopes with per-page metadata.

.. _deconst-default-unsearchable:

deconst_categories
^^^^^^^^^^^^^^^^^^

To apply one or more :term:`categories` to all pages within your repository, specify them as ``deconst_categories``:

.. code-block:: python

  deconst_categories = ['global category one', 'global category two']
