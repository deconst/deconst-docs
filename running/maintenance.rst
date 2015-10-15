Maintenance
===========

Once you've got a deconst cluster provisioned and running, you'll want to monitor its health and
have some idea what you can fix when things go wrong.

Prerequisites
-------------

Before you can effective maintain a cluster, you'll want to verify that you have these things.

 * **Credentials for the cloud account** that's being used to manage the cluster's resources.
 * **SSH access to the cluster.** An administrator can arrange this by adding your GitHub username to the ``github_usernames:`` list within the cluster's ``credentials.yml`` file.
 * **A clone of the `deconst/deploy <https://github.com/deconst/deploy>`_ repository**. If you monitor more than one Deconst cluster, it's helpful to have a separate clone for each, and name the clone's directory after the cluster rather than just calling them all "deploy."
 * **A copy of the ``credentials.yml`` file** for the instance. Your ops team should arrange for an out-of-band mechanism to securely distribute this file and stay up to date. Put it in the root directory of your ``deconst/deploy`` clone.

Logs
----

Application logs are consolidated and shipped to an Elasticsearch cluster external to the Deconst cluster, so that you can quickly see what's happening across the entire system. The Deconst ELK node hosts Logstash (to manipulate the logs right before they're persisted) and Kibana for visualization. Point your browser to port 80 of the ELK node to access Kibana. You can find credentials in your ``credentials.yml`` file:

.. code-block:: bash

  grep elasticsearch_username credentials.yml
  grep elasitcsearch_password credentials.yml

.. image:: /_images/kibana.jpg

I recommend configuration an additional DNS record of ``logs.`` to point to this host, for convenience.

See the `Kibana documentation <https://www.elastic.co/guide/en/kibana/current/index.html>`_ for more information about using Kibana effectively.

Scripts
-------

Within your ``deconst/deploy`` clone, there are a number of scripts that are useful for diagnosing and correcting problems on the cluster.

 * ``script/status`` runs ``docker ps -a`` on all worker hosts. This is a good way to make sure that none of the services have unexpectedly died, or are flapping.
 * ``script/ips`` will show you the hosts and IPs of each system in the cluster. It's occasionally useful to save a trip to the control panel.
 * ``script/lb`` runs a diagnostic check on the load balancers' node membership, ensuring that requests are being forwarded to the correct ports on the worker hosts, based on currently living containers.

    It can run in either a reporting mode (``--report``) that prints a summary of the load balancer health, or a corrective mode (``--fix``) that deletes old nodes and adds new ones.
 * ``script/ssh <host>`` will give you a shell on a host whose name matches the pattern you provide.

Systemd
-------

If you have to nuke it from orbit
---------------------------------

Take a deep breath: it's okay.

When things go so terribly that a cluster is unrecoverable, remember: Deconst stores *all* of its persistent data off-cluster, in Cloud Files, MongoDB and Elasticsearch. The worker hosts are designed to be ephemeral. If you lose ssh access or someone deletes libc or services start flapping and you decide that the system can't recover, you can delete the cloud servers directly, re-provision a new system with the same ``deconst/deploy`` setup (leaving the ``credentials.yml`` file unchanged), and all will be well, no data loss. It takes maybe ten to fifteen minutes.
