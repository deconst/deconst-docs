.. _preparer:

Writing a Preparer
==================

If you want to include content from a new :term:`content repository` format, you'll need to create a new :term:`preparer`. Generally, a preparer needs to:

#. Parse the markup language, configuration files, and other metadata for some content format. When possible, you should use the format's native libraries and tooling to do so.
#. Parse the ``_deconst.json`` file. Consult the :ref:`new content repository section <adding-new-content-repository>` for its schema.
#. Copy assets (usually images) to the directory specified by the environment variable ``ASSET_DIR``. It's best to preserve as much of the local directory structure as possible from the source repository, unless two assets in different subdirectories have the same filename.
#. Use the markup to produce rendered HTML. The preparer should use a single-character placeholder for each asset URL. As it does so, it should generate a map that associates the path of each asset relative to ``ASSET_DIR`` to a collection of character offsets within the body text at which that asset is referenced.

   As a rule, the rendered HTML *should omit any layouts* from the content repository itself and only render the page content, unadorned. In Deconst, templates will be applied :ref:`later, from the control repository <control-template>`. This is important to ensure a consistent look and feel across many content repositories published to the same site, as well as allowing users to take advantage of presenter-implemented features like :ref:`search <control-search>`.

#. Assemble the content into one or more :term:`metadata envelopes` that match the :ref:`envelope schema <envelope-schema>`. If any assets were referenced, include the asset offset map as the ``asset_offsets`` element. Write each completed envelope to the directory specified by the environment variable ``ENVELOPE_DIR`` as a file with the filename pattern ``<content ID, URL-encoded>.json``.

Docker Container Protocol
-------------------------

If you run your preparer in an independent environment (like a non-Deconst continuous integration server), anything that implements the process above will work fine. If you want your preparer to work within the Deconst client or to be available to :ref:`automatically created Strider builds <adding-new-content-repository>`, you need to package your preparer in a Docker container image that obeys the container protocol described here.

Deconst preparer containers should respect the following configuration values:

* ``ASSET_DIR``: The preparer must copy assets to this directory tree.
* ``ENVELOPE_DIR``: The preparer must write completed envelopes to this directory.
* ``CONTENT_ID_BASE``: *(optional)* If set, this should *override* the content ID base specified in ``_deconst.json`` for this preparation run, preferably with some kind of message if they differ.
* ``CONTENT_ROOT``: *(optional)* If specified, the preparer should prepare content mounted to a volume at this path within the container. Otherwise, it should default to preparing ``/usr/content-repo``.

When run with no arguments, the preparer container should prepare the content as described above, then exit with an exit status of 0 if preparation was successful, or nonzero if it was not.
