.. _envelope-schema:

Metadata Envelope Schema
========================

Much of the deconst system involves the manipulation of :term:`metadata envelopes`, the JSON documents produced by each :term:`preparer` that contain the actual content to render. To be presented properly, envelopes must adhere to a common schema.

This is an example envelope that demonstrates the full document structure, including all optional fields:

.. code-block:: json

  {
    "body": "<h1>Rendered HTML</h1>",
    "title": "SDKs &amp; Tools",
    "author": "Ash Wilson",
    "bio": "He's just this guy, you know?",
    "toc": "<ul>\n<li><a href=\"#\">Top</a></li><li><a href=\"#section\">Section</a></li></ul>",
    "publish_date": "Fri, 15 May 2015 18:32:45 GMT",
    "tags": ["tag one", "tag two"],
    "categories": ["category one", "category two"],
    "keywords": ["keyword one", "keyword two"],
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

.. glossary::

  body
    The only required field for a valid envelope. It contains the pre-rendered HTML of the page.

  title
    The page title or blog post name used for this document.

  toc
    The table of contents for this page as a fragment of rendered HTML.

  content_type
    If specified, set the Content-Type of the response containing this document. Defaults to text/html; charset=utf-8.

  author
    Name of the author who wrote this content.

  bio
    A brief paragraph describing the :term:`author`.

  publish_date
    Approximate timestamp on which this piece of content was published, formatted as an RFC2822 string.

  tags
    An array of content classification strings that may be normalized or supplemented with machine-generated information.

  categories
    An array of content classification strings that are explicitly user-provided and chosen from a list fixed in the control repository.

  keywords
    An array of terms to supplement full-text search indexing.

  disqus
    An object that controls the inclusion of Disqus comments on the current page. If present, must be an object with the following structure:

    .. code-block:: json

      "disqus": {
        "include": true,
        "short_name": "devblog",
        "embed": true
      }

    **include** toggles the inclusion of any Disqus content at all. **short_name** is used to link to a specific Disqus account. **embed** toggles the included script between an *embedding script* that injects a Disqus comment form on this page and a *count script* that decorates links with a comment count.

  next
  previous
    These objects, if included, provide navigational links to adjacent documents in a sequence. If present, must be an object with the following structure:

    .. code-block:: json

      "next": {
        "title": "page title",
        "url": "../next-page"
      }

    If the ``url`` key is absolute (rooted at the document root, like ``/blog/other-post``), the presenter will re-root it based on the current mapping of the content repository. If it's relative, it will be left as-is.

The documents retrieved from the content store consist of the requested envelope and a number of additional attributes that are derived and injected at retrieval time. The full content document looks like this:

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
