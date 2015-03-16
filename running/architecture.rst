Architecture
============

Deconst is distributed as a set of Docker containers, deployed by an Ansible playbook.

Terminology
-----------

.. glossary::

  content repository
    Location containing material to be included as a subset of the completed site. Often, the
    material within will be written in a friendlier human-editable markup format such as
    reStructuredText or Markdown.

    Initially, we will support support content stored in git repositories on
    `GitHub <github.com>`_, although our architecture will be flexible enough to integrate
    content stored anywhere on a network reachable from the build system.

  map repository
    Version controlled repository containing plain-text documents that associate subtrees of
    indexed content, identified by a :term:`content ID` prefix, with subtrees of :term:`paths`
    on the presented site.

  content ID
    Unique identifier assigned to a single page of content. By convention, these are URLs that
    join the https URL of the GitHub repository, ``tree/branch`` if "branch" is not master, and
    the subpath of the page within the *rendered* output of the documentation build tool. Most of
    the architecture should treat these as opaque strings. Examples:
    this page would be ``https://github.com/deconst/deconst-docs/running/architecture`` on master,
    or ``https://github.com/deconst/deconst-docs/tree/glossary/running/architecture`` until this
    pull request is merged.

  path
  paths
    Subpath within the final presented content of a deconst site. Includes the fully-qualified
    domain name of the page. For example:
    ``developer.rackspace.com/sdks/cloud-servers/getting-started/``.

  layout
    Template of common markup that surrounds each presented page with navigation, brand identity,
    copyright information and anything else that's shared among some subset of each site.

Components
----------

.. glossary::

  preparer
    Process responsible for converting a :term:`content repository` into a directory tree of JSON
    documents, each of which contains one page of rendered HTML and associated metadata.

    If the current branch is live, the generated JSON documents are then submitted to the
    :term:`content service` for storage and indexing. Otherwise, a local :term:`presenter` is
    invoked to complete a full build of this subtree of the final site, which is then published to
    CDN and linked on the pull request.

    There will be one preparer for each supported format of :term:`content repository`; initially,
    Sphinx and Jekyll. The preparer will be executed by a CI/CD system on each commit to the
    repository.

  content service
    Service that accepts submissions and queries for the most recent page content associated with a
    specific :term:`content ID`. Content submitted here will have its structure validated and
    indexed.

  mapping service
    Given a :term:`path`, return the corresponding :term:`content ID`, or an alternate path to use
    as a redirect target. Uses the latest version of the :term:`map repository` as a source of truth
    for performing the association.

  presenter
    Accept HTTP requests from users. Map the requested :term:`path` to :term:`content ID` by
    querying the :term:`mapping service`, then access the requested content using the
    :term:`content service`. Inject the content into an approriate :term:`layout` and send the
    final HTML back in an HTTP response.
