.. Deconst documentation master file, created by
   sphinx-quickstart on Tue Mar 10 08:03:51 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Deconst
=======

*Deconstruct your Documentation*

`Deconst <https://github.com/deconst>`_ is a continuous delivery pipeline used to assemble documentation from a heterogenous set of source repositories. Individual GitHub repositories containing content in :abbr:`.rst (reStructuredText)` or :abbr:`.md (Markdown)` formats are **prepared** by being partially rendered to a common JSON format, then mapped to subtrees of the final page by a **control repository**, a plaintext file stored in its own repository.

This guide serves two purposes:

#. It's documentation for deconst itself. If you want to write documentation for a deconst-managed site, or if you want to run a deconst cluster yourself, this will help you get started.
#. It's also used as a concrete example for deconst's development! We use this to dogfood the deconst contribution and renderer workflow.

.. warning::

  We're still prototyping the deconst process and getting things up and running! Most of the content here is purely speculative at the moment and is likely to change heavily as soon as we start writing real code and plugging things into other things. It'll change even *more* heavily once we introduce users into the mix. Take this for what it is: a glimpse into our minds at a particular point in time, which will probably lag reality somewhat until we have something running and stable.

  With that being said, if you read something and it's *really* WTF, `pop open an issue <https://github.com/deconst/deconst-docs/issues/new>`_ and let's talk about it!

Contents:

.. toctree::

   writing-docs/index
   running/index
   developing/index

Indices and tables
==================

* :ref:`genindex`
* :ref:`search`
