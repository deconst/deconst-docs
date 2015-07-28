Terminology
===========

It's important to have a shared vocabulary when talking about complicated software systems. We generally try to consistently use these terms in our code, comments, issues and chat.

.. glossary::

  content repository
  content repositories
    Location containing material to be included as a subset of the completed site. Often, the
    material within will be written in a friendlier human-editable markup format such as
    reStructuredText or Markdown.

    Initially, we will support support content stored in git repositories on
    `GitHub <github.com>`_, although our architecture will be flexible enough to integrate
    content stored anywhere on a network reachable from the build system.

  control repository
    Version controlled repository used to organize and administer a deconst platform. It contains:

    * Plain-text documents that associate subtrees of indexed content, identified by a
      :term:`content ID` prefix, with subtrees of :term:`presented URLs` on the presented site.
    * Layout templates in `handlebars <http://handlebarsjs.com/>`_ format.
    * Plain-text documents that associate subsets of :term:`presented URLs` with specific layouts.

  content ID
  content IDs
    Unique identifier assigned to a single page of content generated from a :term:`content
    repository`. It's important to note that a content ID is assigned to each *output* page, not
    each source document. Depending on the :term:`preparer` and its configuration, these may differ.
    Most of the architecture should treat these as opaque strings, although the content map may need to assume that they are hierarchal.

    By convention, these are URLs that join the base URL of the :term:`content repository` with the
    relative path of the rendered output page.

    Examples:

    * The version of this page on the default "master" branch:
      ``https://github.com/deconst/deconst-docs/running/architecture``.
    * The version of this page on a branch called "glossary":
      ``https://github.com/deconst/deconst-docs/tree/glossary/running/architecture``.
    * A specific post in a Jekyll blog, generated from (theoretical) content at
      ``https://github.com/rackerlabs/developer-blog/_posts/mongodb-3.0-getting-started.md``:
      ``https://github.com/rackerlabs/developer-blog/blog/mongodb-3.0-getting-started``.

  presented URL
  presented URLs
    URL of a page within the final presented content of a deconst site. This should be the full URL,
    including the scheme and domain.

    Example: ``https://developer.rackspace.com/sdks/cloud-servers/getting-started/``.

  layout
    Template of common markup that surrounds each presented page with navigation, brand identity,
    copyright information and anything else that's shared among some subset of each site.

  metadata envelope
  metadata envelopes
    JSON document that contains a single page's worth of content as a rendered HTML fragment, along with any additional information necessary for the presentation of that page. See :ref:`the schema section <envelope-schema>` for a description of the expected structure.
