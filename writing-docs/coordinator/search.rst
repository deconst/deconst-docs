.. _control-search:

Search
------

All content that's submitted to a Deconst instance is also indexed for search. In order to display search results in a Deconst site, you'll need to implement a search results page within your control repository.

The search results page must be defined entirely by a Nunjucks template within your control repository. It can't be submitted from any content repository. This is because, to actually perform the search and enumerate results, you need to use a `custom Nunjucks filter <https://mozilla.github.io/nunjucks/templating.html#filters>`_ that's only available to control repository templates.

First, map a search path within your :ref:`content map <control-map-syntax>` at ``config/content.json``. It doesn't need to map to an actual content ID. Instead, you'll usually want to map it to ``null`` to use a fixed, empty metadata envelope.

.. code-block:: json

  {
    "books.horse": {
      "content": {
        "/": "https://github.com/user/library-welcome/",
        "/search/": null
      }
    }
  }

Now :ref:`route <control-template-map>` this path to the search template in ``config/routes.json``.

.. code-block:: json

  {
    "books.horse": {
      "routes": {
        "^/": "default.html",
        "^/search/?": "search.html"
      }
    }
  }

Finally, you'll need to :ref:`create the template <control-template>` that displays the results of a given search. Create it as you would any other template, but rather than render ``{{ deconst.content.envelope.body }}``, invoke the ``search`` filter on the query parameter:

.. code-block:: html

  <h1>Your Search Results</h1>

  {% set r = deconst.request.query.q|search %}

  <h2>Your search had {{ r.total }} results.</h2>

  {% for result in r.results %}
    <!-- Use attributes on the result object to generate a search result. -->
    <div>
      <a href="{{ result.url }}">{{ result.title }}</a>
      <p>{{ result.excerpt }}</p>
    </div>
  {% else %}
    <!-- Remember to show something sensible if no results are found. -->
    <p>No results found.</p>
  {% endfor %}

The search filter accepts two optional parameters: the current page number and the count of entries per page. The page number defaults to 1 and the page count defaults to 10.

.. code-block:: html

  {% set query = deconst.request.query %}
  {% set r = query.q|search(query.page, query.pageSize) %}
