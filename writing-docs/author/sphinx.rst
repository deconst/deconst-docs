reStructuredText content in Sphinx
==================================

`Sphinx <http://sphinx-doc.org/contents.html>`_ is a documentation builder that assembles `reStructureText <http://docutils.sourceforge.net/rst.html>`_ source files into cohesive output that includes tables of contents, cross-references, and integrated navigation.

Assets
------

To integrate properly with Deconst's asset pipeline:

 1. Place any images in an `_images` directory at the top level of your Sphinx documentation.
 2. Reference images in ``.rst`` files by including a ``.. image`` macro.

.. code-block:: rst

  .. image:: /_images/deconst-initial.png
