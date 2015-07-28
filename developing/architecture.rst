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

  presenter
    Accept HTTP requests from users. Map the requested :term:`presented URL` to :term:`content ID`
    using the latest known version of the content mapping within the control repository, then access the requested :term:`metadata envelope` using the :term:`content service`. Inject the envelope into an appropriate :term:`layout` and send the final HTML back in an HTTP response.

Lifecycle of an HTTP Request
----------------------------

When an HTTP request hits the :term:`presenter`:

1. The :term:`presenter` queries its content map with the :term:`presented URL` to discover the :term:`content ID` of the content that should be rendered at that path.
2. Next, the presenter queries the :term:`content service` to acquire the content for that ID. The content service locates the appropriate :term:`metadata envelope`, all site-wide assets, and performs any necessary post-processing.
3. Armed with the content ID and a layout key from the metadata envelope, the presenter locates the Nunjucks :term:`layout` that should be used to decorate the raw content. If no layout key is present, this request is skipped and a null layout (that renders the envelope's body directly) is used.
4. Meanwhile, any "related documents" that are requested by the envelope will be queried from the :term:`content service`.
5. The presenter renders the metadata envelope using the layout. The resulting HTML document is returned to the user.

Lifecycle of a Control Repository Update
----------------------------------------

When a change is merged into the live branch of the :term:`control repository`:

1. A Travis CI build executes the asset :term:`preparer` on the latest commit of the repository. Stylesheets, javascript, images, and fonts found within the ``assets`` directory are compiled, concatenated, minified, and submitted to the :term:`content service` to be fingerprinted, stored on the CDN-enabled asset container, and made available as global assets to all metadata envelopes.
2. Meanwhile, a GitHub webhook is fired. One :term:`webhook service` within the deconst cluster receives a ``POST`` and validates its payload.
3. The webhook service writes to an etcd key on the etcd cluster.
4. *Every* :term:`etcd watcher` service across the cluster is notified, and sends a ``POST`` to the ``/refresh`` endpoint of the :term:`mapping service` and :term:`layout service` within the same pod.
5. Each :term:`mapping service` and :term:`layout service` performs a shallow git clone of the control repository's new state and parses the relevant mapping information from certain files. As soon as the parsing completes successfully, the new state is live.

Lifecycle of a Content Repository Update
----------------------------------------

When a change is merged into the live branch of a :term:`content repository`:

1. A Travis CI build executes the appropriate :term:`preparer` on the latest commit of the repository.
2. The preparer generates a :term:`metadata envelope` for each page that would be rendered, assigns it a :term:`content ID` using a configured base ID, and submits it to the :term:`content service`.
3. Each static resource (images, mostly) are submitted to the :term:`content service` and published to the CDN as non-global assets. The response includes the CDN URL, which is then used within the generated envelopes.
