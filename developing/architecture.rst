Architecture
============

Deconst is distributed as a set of Docker containers, deployed to a cluster of CoreOS hosts by an Ansible playbook. Containers are organized into sets of linked services called "pods." Deconst can be scaled by both launching additional worker hosts and by starting a greater number of pods on each host.

.. note::

  The name "pod" is taken from Kubernetes, but I'm using it to mean something slightly different here. A Kubernetes pod is also a set of related containers, but they share networking and cgroup attributes, which our pods do not do. The containers within a Deconst pod are only related by regular Docker network links.

This is how the world interacts with a Deconst cluster:

.. image:: /_images/deconst-external.png

None of the service containers store any internal, persistent state: the sources of truth for all Deconst state are Cloud Files containers, MongoDB collections, or GitHub repositories. This means that you can adaptively destroy or launch Deconst worker hosts without fear of losing information.

Each pod includes the following arrangement of interlinked service containers:

.. image:: /_images/deconst-internal.png

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

  content service
    Service that accepts submissions and queries for the most recent :term:`metadata envelope`
    associated with a specific :term:`content ID`. Content submitted here will have its structure
    validated and indexed.

  mapping service
    Given a :term:`presented URL`, return the corresponding :term:`content ID`, or an alternate
    destination to use as a redirect target. Uses the latest version of the :term:`control
    repository` as a source of truth for performing the association.

  layout service
    Given a :term:`presented URL`, return the Handlebars template that should be used to render the
    corresponding final page. Uses the latest version of the :term:`control repository` as a source
    of truth for both associating a layout with a specific page, and for the layout templates
    themselves.

  webhook service
    Listen for webhook notifications from the control repository. When a push has been performed, the webhook service updates a known key in etcd. Effectively, this will broadcast the change to every :term:`etcd watcher` across the entire cluster.

  etcd watcher
    When *any* :term:`webhook service` has received a push notification, perform a ``POST`` against both the :term:`mapping service` and :term:`layout service` within the same pod, to prompt each to refresh its view of the :term:`control repository`.

  presenter
    Accept HTTP requests from users. Map the requested :term:`presented URL` to :term:`content ID`
    by querying the :term:`mapping service`, then access the requested :term:`metadata envelope`
    using the :term:`content service`. Inject the envelope into an appropriate :term:`layout` and send the
    final HTML back in an HTTP response.
