.. _control-map:

Content Mapping Files
---------------------

The **content mapping file** is a `JSON <http://www.json.org/>`_ file located at the path ``config/content.json``. It works by placing a subtree of content from a certain content repository at a subtree of a domain within the overall Deconst site.

For example, suppose we have a content repository at ``https://github.com/user/book1`` that creates the following pages:

.. code-block:: text

  introduction
  chapter-1/getting-started
  chapter-1/and-then

And another content repository at ``https://github.com/user/book2`` that creates these pages:

.. code-block:: text

  welcome
  chapter-1/the-basics
  chapter-1/more-detail

If we create mapping entries that map ``library/my-book/`` to ``https://github.com/user/book1/`` and ``library/another-book/`` to ``https://github.com/user/book2/``, both on the domain *books.horse*, these pages will be available at the following URLs:

.. code-block:: text

  https://books.horse/library/my-book/introduction/
  https://books.horse/library/my-book/chapter-1/getting-started/
  https://books.horse/library/my-book/chapter-1/and-then/
  https://books.horse/library/another-book/welcome/
  https://books.horse/library/another-book/chapter-1/the-basics/
  https://books.horse/library/another-book/chapter-1/more-detail/

The **longest prefix** that matches an incoming URL is used to decide which mapping is used to locate the content to render. For example, if ``/base/`` is mapped to ``https://github.com/user/base``, but ``/base/subpage/`` is mapped to ``https://github.com/user/subpage``, requests will be mapped as follows:

  * **https://books.horse/base** will render **https://github.com/user/base/**.
  * **https://books.horse/base/something** will render **https://github.com/user/base/something**.
  * **https://books.horse/base/subpage** will render **https://github.com/user/subpage** because the ``/base/subpage/`` mapping now takes precendence, *even if https://github.com/user/base/subpage exists within that content repository.*
  * **https://books.horse/base/subpage/anything** will render **https://github.com/user/subpage/anything**.

.. note::

  Technically, content mappings work with the :term:`content IDs` that are produced by the :term:`preparer` that "builds" each content repository. To do more complicated mappings, it's helpful to know the :ref:`details of exactly how they're produced <control-content-ids>`, but to get started you can assume that the content repository's URL is a prefix for the content IDs of all of its content.

Changes to the content mapping files will take effect as soon as they're merged into the ``master`` branch of the control repository. Huzzah for continuous delivery!

.. _control-map-syntax:

Content Map Syntax
^^^^^^^^^^^^^^^^^^

The content mapping file syntax looks like this:

.. code-block:: json

  {
    "books.horse": {
      "content": {
        "/": "https://github.com/user/library-welcome/",
        "/library/my-book/": "https://github.com/user/book1/",
        "/library/another-book/": "https://github.com/user/book-2/"
      }
    },
    "nextbigthing.io": {
      "content": {
        "/": "https://github.com/someone-else/nextbigthing-index/",
        "/product/": "https://github.com/someone-else/product-sdk/"
      }
    }
  }

It's an error to map the exact same prefix on the same domain more than once. This is to prevent you from accidentally clobbering your own mappings by mistake!

.. note::

  End each URL prefix and each content ID prefix with a trailing slash. Deconst is smart enough to do the right thing for content at the root of each mapping: the URL **https://books.horse/library/my-book** will render the content at **https://github.com/user/book1/**, not **https://github.com/user/library-welcome/my-book**.
