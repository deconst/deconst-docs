.. _envelope-schema:

Metadata Envelope Schema
========================

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
        "description": "Associate this page with a layout in the control repository by a key. The exact layout chosen will be determined by the layout mapping service at page-rendering time. If absent or falsy, the body will be rendered as-is with no decoration.",
        "type": "string"
      },
      "title": {
        "description": "The page title or blog post name used for this document.",
        "type": "string"
      },
      "toc": {
        "description": "The table of contents for this page as rendered HTML.",
        "type": "string"
      },
      "content_type": {
        "description": "If specified, set the Content-Type of the response containing this document. Defaults to text/html; charset=utf-8.",
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
    "toc": "<ul>\n<li><a href=\"#\">Top</a></li><li><a href=\"#section\">Section</a></li></ul>",
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
