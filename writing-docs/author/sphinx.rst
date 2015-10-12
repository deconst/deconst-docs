reStructuredText content in Sphinx
==================================

`Sphinx <http://sphinx-doc.org/contents.html>`_ is a documentation builder that assembles `reStructureText <http://docutils.sourceforge.net/rst.html>`_ source files into cohesive output that includes tables of contents, cross-references, and integrated navigation.

Specifying a Layout
-------------------

Deconst allows each page to be assocated with :ref:`a layout <control-layout>` from the control repository by specifying a *key*. Sphinx doesn't include the concept of applying a layout to an individual page. The Sphinx preparer works around this by recognizing a piece of `per-page metadata <http://sphinx-doc.org/markup/misc.html#file-wide-metadata>`_ in each file.

You can see which layouts are available within your instance by consulting :ref:`layout mapping files within the control repository. <control-layout>` To support a consistent experience across an entire deconst instance, you should coordinate your choice of layout with your instance's information architects.

To use a specific layout for a page, include the following as the *first line* of any ``.rst`` file:

.. code-block:: rst

  :deconstlayout: this-pages-layout-key

If most of the pages in your repository will be using a common layout, you can also specify a default layout key by adding the following to your ``conf.py`` file:

.. code-block:: python

  deconst_default_layout = "this-repos-default-layout-key"

Assets
------

To integrate properly with Deconst's asset pipeline:

 1. Place any images in an `_images` directory at the top level of your Sphinx documentation.
 2. Reference images in ``.rst`` files by including a ``.. image`` macro.

.. code-block:: rst

  .. image:: /_images/deconst-initial.png
