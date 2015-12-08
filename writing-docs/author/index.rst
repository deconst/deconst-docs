Authoring Content for Deconst
=============================

Deconst *content authors* write the documentation that's rendered at some domain and path on the final instance. The content that makes up a deconst instance is brought together from many :term:`content repositories`, each of which contributes a single logical unit of documentation that can be maintained independently from all of the others.

The domain and subpath that host the content from a specific content repository is determined by a mapping that's managed within the :term:`control repository` associated with your Deconst instance. To add a new content repository to the instance, you or a *site coordinator* will need to add an entry to the control repository's :ref:`content mapping file <control-map>` and configure a :abbr:`CI (Continuous Integration)` build.

Once the content repository is fully configured, any changes merged into the "master" branch will automatically be live.

Where Will Your Content Live
----------------------------

The final output from each content repository will be presented at a subpath of the complete site. For example, if you create the following pages:

.. code-block:: text

  welcome
  chapter-1/introduction
  chapter-1/getting-started
  chapter-2/more-advanced

And you're currently mapped to the ``books/example/`` subpath of *mysite.com* by the control repository, then your pages will be available at the following URLs:

.. code-block:: text

  https://mysite.com/books/example/welcome/
  https://mysite.com/books/example/chapter-1/introduction/
  https://mysite.com/books/example/chapter-1/getting-started/
  https://mysite.com/books/example/chapter-2/more-advanced/

As you work, you can freely create new pages and directories and they will automatically be available within that subpath.

.. warning::

  Currently, *deleting* pages doesn't actually remove the content from deconst. An administrator needs to remove documents from Cloud Files manually to delete content.

Content Repository Metadata
------------------------------------

Deconst preparers will look for a file named ``_deconst.json`` at the root of each content repository. This file allows content authors to provide metadata that may influence the way the deconst instance handles content from the repository.

- ``contentIDBase``: A string that is prepended to the local path of every submitted file in order to create a unique **content ID**. This is often the content repository's Github URL.
- ``githubUrl``: The content repository's Github URL. If present, will be used to add ``github_issues_url`` and ``github_edit_url`` to each submitted content envelope.
- ``meta``: An object with arbitrary content. This object will be merged with document-specific metadata and set as the  ``meta`` property of each submitted content envelope.

Supported Content Repository Formats
------------------------------------

Each content repository can independently choose a documentation engine that makes the most sense for the content it contains. You can choose from any format that has a matching :term:`preparer`. Preparers extend the native documentation engine to support additional functionality that Deconst needs to integrate their output with the rest of the system.

.. toctree::

  sphinx
  jekyll
