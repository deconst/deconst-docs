.. Deconst documentation master file, created by
   sphinx-quickstart on Tue Mar 10 08:03:51 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Deconst
=======

*Deconstruct your Documentation*

`Deconst <https://github.com/deconst>`_ is a continuous delivery pipeline used
to assemble documentation from a heterogenous set of source repositories.
Individual GitHub repositories containing content in :abbr:`.rst
(reStructuredText)` or :abbr:`.md (Markdown)` formats are **prepared** by being
partially rendered to a common JSON format, then mapped to subtrees of the final
page by a **control repository**.

This guide serves two purposes:

#. It's documentation for deconst itself. If you want to write documentation for
   a deconst-managed site, or if you want to run a deconst cluster yourself, this
   will help you get started.

#. It's also used as a concrete example for deconst's development! We use this
   to dogfood the deconst contribution and renderer workflow.

Contents:

.. toctree::

   writing-docs/index
   running/index
   developing/index

Indices and tables
==================

* :ref:`genindex`
* :ref:`search`
