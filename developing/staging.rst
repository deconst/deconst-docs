.. _staging:

Staging Environment
===================

The staging environment is a specially configured :term:`content service` and
:term:`presenter` pair that allow users to preview content, in context with the
rest of the hosted site, before it's live and shown to end users.

A specific build of previewed content is identified by a :term:`revision ID`.
This gives each build a unique environment and enables the staging environment
to host many revisions independently of one another.

Submitting Content to the Staging Environment
---------------------------------------------

To submit content to the staging :term:`content service`, run the normal
:term:`preparer`, but:

#. Set ``CONTENT_SERVICE_URL`` to the staging environment's API endpoint.

#. Prepend a revision ID as the first URL path segment of your :term:`content
   ID` base. For example, if your content repository's normal content ID base is
   ``https://github.com/deconst/deconst-docs/``, set ``CONTENT_ID_BASE`` to
   ``https://github.com/build-abcdef/deconst/deconst-docs/`` instead.

The revision ID is arbitrary, but it should be chosen to be relatively unique
among everyone who's staging changes so you don't overwrite one another's staged
content by mistake. Good examples include something containing part of your
current git SHA, your username, or a timestamp of some kind.

To amend an existing revision's content, re-run the preparer with the same
revision ID. To append content from a different content repository to the same
staging environment, run its preparer with the revision ID.

Viewing Staged Content
----------------------

To see the content that you've just staged, visit the staging
:term:`presenter`'s address and prepend your revision ID to the URL path. For
example, if you just built content that's normally mapped to the path ``/docs/``
to a staging server that's available at ``https://staging.example.com/`` with
the revision id ``user-smashwilson``, your staged content will be visible at
``https://staging.example.com/user-smashwilson/docs/``.

The rest of the site will be *also* be visible beneath the parent
``/user-smashwilson/`` path exactly as it appears on the current production
site. Any links on any rendered page will be manipulated such that they will
point to the equivalent content within the same revision ID. This means that you
can click around the staging environment, using site navigation normally,
without accidentally jumping to the production endpoint instead.
