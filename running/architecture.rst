Architecture
============

Deconst is distributed as a set of Docker containers, deployed by an Ansible playbook.

This is our initial architecture:

.. image:: /_images/deconst-initial.png

Terminology
-----------

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
    Most of the architecture should treat these as opaque strings, although the :term:`mapping
    service` may need to assume that they are hierarchal.

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
    JSON document that contains a single page's worth of content as a rendered HTML fragment, along with
    any additional information necessary for the presentation of that page. See :ref:`the schema section
    <envelope-schema>` for a description of the expected structure.

Components
----------

.. glossary::

  preparer
    Process responsible for converting a :term:`content repository` into a directory tree of
    :term:`metadata envelopes`, each of which contains one page of rendered HTML and associated
    metadata.

    If the current branch is live, the generated envelopes are then submitted to the
    :term:`content service` for storage and indexing. Otherwise, a local :term:`presenter` is
    invoked to complete a full build of this subtree of the final site, which is then published to
    CDN and linked on the pull request.

    There will be one preparer for each supported format of :term:`content repository`; initially,
    Sphinx and Jekyll. The preparer will be executed by a CI/CD system on each commit to the
    repository.

  mapping service
    Given a :term:`presented URL`, return the corresponding :term:`content ID`, or an alternate
    destination to use as a redirect target. Uses the latest version of the :term:`control
    repository` as a source of truth for performing the association.

  content service
    Service that accepts submissions and queries for the most recent :term:`metadata envelope`
    associated with a specific :term:`content ID`. Content submitted here will have its structure
    validated and indexed.

  layout service
    Given a :term:`presented URL`, return the Handlebars template that should be used to render the
    corresponding final page. Uses the latest version of the :term:`control repository` as a source
    of truth for both associating a layout with a specific page, and for the layout templates
    themselves.

  presenter
    Accept HTTP requests from users. Map the requested :term:`presented URL` to :term:`content ID`
    by querying the :term:`mapping service`, then access the requested :term:`metadata envelope`
    using the :term:`content service`. Inject the envelope into an approriate :term:`layout` and send the
    final HTML back in an HTTP response.

.. _envelope-schema:

Metadata Envelope Schema
------------------------

Much of the deconst system involves the manipulation of :term:`metadata envelopes`, the JSON documents
produced by each :term:`preparer` that contain the actual content to render. To be presented properly,
envelopes must adhere to a common schema.

Here's a `JSON schema <http://json-schema.org/>`_ document that describes its expected structure:

.. code-block:: json

  {
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Deconst Metadata Envelope",
    "type": "object",
    "properties": {
      "body": {
        "description": "Partially rendered HTML to be injected into a selected layout.",
        "type": "string"
      },
      "layout_key": {
        "description": "Associate this page with a layout in the control repository by a key. The exact layout chosen will be determined by the layout mapping service at page-rendering time. If absent, the body will be rendered as-is with no decoration.",
        "type": "string"
      },
      "title": {
        "description": "The page title or blog post name used for this document.",
        "type": "string"
      },
      "author": {
        "description": "Name of the author who wrote this content.",
        "type": "string"
      },
      "bio": {
        "description": "Brief paragraph describing the author.",
        "type": "string"
      },
      "publish_date": {
        "description": "Approximate timestamp on which this piece of content was published, formatted as an RFC2822 string.",
        "type": "string",
      },
      "tags": {
        "description": "Content classification strings that may be normalized or supplemented with machine-generated information.",
        "type": "array",
        "items": { "type": "string" },
      },
      "categories": {
        "description": "Content classification strings that are explicitly user-provided and chosen from a list fixed in the control repository.",
        "type": "array",
        "items": { "type": "string" },
      },
      "disqus": {
        "type": "object",
        "properties": {
          "include": {
            "type": "boolean",
            "description": "If true, a layout may render Disqus integration Javascript."
          },
          "short_name": {
            "type": "string",
            "description": "The 'short name' assigned to the Disqus account."
          },
          "embed": {
            "type": "boolean",
            "description": "If true, Javascript will be generated to embed a Disqus comment form on this page. Otherwise, the script to generate comment counts will be injected instead."
          }
        },
      },
      "next": {
        "type": "object",
        "properties": {
          "title": { "type": "string" },
          "url": { "type": "string" }
        },
        "required": ["title", "url"]
      },
      "previous": {
        "type": "object",
        "properties": {
          "title": { "type": "string" },
          "url": { "type": "string" }
        },
        "required": ["title", "url"]
      },
      "queries": {
        "description": "Render-time queries for other content to perform dynamically, during page render. See 'results' in the content document below.",
        "type": "object"
      },
      "required": ["body"]
    }
  }

This is an example envelope that demonstrates the full document structure in a more concrete way:

.. code-block:: json

  {
    "body": "<h1>Rendered HTML</h1>",
    "title": "SDKs &amp; Tools",
    "author": "Ash Wilson",
    "bio": "He's just this guy, you know?",
    "publish_date": "Fri, 15 May 2015 18:32:45 GMT",
    "disqus": {
      "include": true,
      "short_name": "devblog",
      "embed": true
    },
    "next": {
      "title": "The next article",
      "url": "/blog/next-article"
    },
    "previous": {
      "title": "The previous article",
      "url": "/blog/previous-article"
    }
  }

The documents retrieved from the content store consist of the requested envelope, plus a number of additional attributes that are derived and injected at retrieval time. The full content document looks like this:

.. code-block:: json

  {
    "envelope": {},
    "assets": {
      "page_css_url": "https://...",
      "page_js_url": "https://..."
    },
    "has_next_or_previous": true,
    "presented_url": "https://...",
    "results": {
      "queryname": []
    }
  }
