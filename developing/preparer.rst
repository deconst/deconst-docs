.. _preparer:

Writing a Preparer
==================

If you want to include content from a new :term:`content repository` format, you'll need to create a new :term:`preparer`. Generally, a preparer needs to:

#. Parse the markup language, configuration files, and other metadata for some content format. When possible, you should use the format's native libraries and tooling to do so.
#. Parse the ``_deconst.json`` file. Consult the :ref:`new content repository section <adding-new-content-repository>` for its schema.
#. Submit assets (usually images) to the :term:`content service` API's `/asset endpoint <https://github.com/deconst/content-service#post-assetnamedtrue>`_. Omit the `?named=true` parameter; it's used by the control repository's preparer. The response payload maps each uploaded asset to a final URL that the preparer should remember.
#. Use the markup to produce rendered HTML. The preparer should use the final asset URLs provided before.

   As a rule, the rendered HTML *should omit any layouts* from the content repository itself and only render the page content, unadorned. In Deconst, templates will be applied :ref:`later, from the control repository <control-template>`. This is important to ensure a consistent look and feel across many content repositories published to the same site, as well as allowing users to take advantage of presenter-implemented features like :ref:`search <control-search>`.

#. Assemble the content into one or more :term:`metadata envelopes` that match the :ref:`envelope schema <envelope-schema>`.
#. Submit each prepared envelope to the :term:`content service` API's `/content endpoint <https://github.com/deconst/content-service#put-contentid>`_.

Each HTTP request sent to the content service should be accompanied by an ``Authorization`` header containing a valid API key:

.. code-block:: text

  PUT /content/https%3A%2F%2Fgithub.com%2Fsomeuser%2Fsomerepo%2Fsomeid
  Authorization: deconst apikey="12345"

Docker Container Protocol
-------------------------

If you're running your preparer in an independent environment (like a non-Deconst continuous integration server), anything that implements the process above will work fine. If you want your preparer to work within the Deconst client or to be available to :ref:`automatically created Strider builds <adding-new-content-repository>`, you'll need to package your preparer in a Docker container image that obeys the container protocol described here.

Deconst preparer containers should respect the following configuration values:

* ``CONTENT_STORE_URL``: The base URL of the :term:`content service`, with a trailing slash. For example: ``"https://deconst.horse:9000/"``.
* ``CONTENT_STORE_APIKEY``: A valid API key for the content service.
* ``CONTENT_STORE_TLSVERIFY``: If set to ``"false"``, TLS certificate validity should not be checked for content store connections. **Never use this option in production,** as it potentially allows your connection to be subjected to a `man-in-the-middle attack <https://en.wikipedia.org/wiki/Man-in-the-middle_attack>`_.
* ``CONTENT_ID_BASE``: If set, this should *override* the content ID base specified in ``_deconst.json`` for this preparation run, preferably with some kind of message if they differ.
* ``CONTENT_ROOT``: If specified, the preparer should prepare content mounted to a volume at this path within the container. Otherwise, it should default to preparing ``/usr/content-repo``.

When run with no arguments, the preparer container should prepare the content as described above, then exit with an exit status of 0 if preparation was successful, or nonzero if it was not.
