Advanced Topics
---------------

.. _control-content-ids:

Content IDs
^^^^^^^^^^^

Strictly speaking, the way that :term:`content IDs` are assigned is an arbitrary decision made by the :term:`preparer` that's configured on that repository. However, by convention, they follow a pattern:

  *base URL of the content repository* + *subpath of the rendered page*

For example, suppose that we have a content repository hosted at https://github.com/deconst/deconst-docs that contains Sphinx documentation. A page within that repository that renders at *writing-docs/coordinator* would be assigned a content ID of ``https://github.com/deconst/deconst-docs/writing-docs/coordinator``.
