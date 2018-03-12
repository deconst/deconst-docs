Authoring Content for Deconst
=============================

Deconst *content authors* write the documentation that's rendered at some domain
and path on the final instance. The content that makes up a deconst instance is
brought together from many :term:`content repositories`, each of which
contributes a single logical unit of documentation that can be maintained
independently from all of the others.

The domain and subpath that host the content from a specific content repository
is determined by a mapping that's managed within the :term:`control repository`
associated with your Deconst instance. To add a new content repository to the
instance, you or a *site coordinator* will need to add an entry to the control
repository's :ref:`content mapping file <control-map>` and configure a :abbr:`CI
(Continuous Integration)` build.

Once the content repository is fully configured, any changes merged into the
"master" branch will automatically be live.

.. _adding-new-content-repository:

Adding a New Content Repository
-------------------------------

The easiest content repositories to add to a Deconst instance are:

- Written in one of the :ref:`already-supported formats <supported-formats>`.
- Hosted in a git repository on `github.com <https://github.com/>`_, public or
  private.

If your content repository does not meet those criteria, :ref:`integrating your
content is still possible, but will likely take more work
<custom-content-integration>`. If you do qualify for the easy route, add a new
content repository to Deconst by:

#. **Ensure that the Deconst instance's GitHub account can access your
   repository.** If your repository is public, you don't have to do anything. If
   your repository is private, you'll need to grant the Deconst instance's GitHub
   account access before your build can be configured. Ask a Deconst administrator
   for the name of the bot account.

#. **Create a "_deconst.json" file within each content root directory.** This
   file tells Deconst important details about the content within this directory.
   Place it in the same directory as your ``conf.py`` or ``_config.yml`` files.

   The most important setting within this file is the *content ID base*. The
   content ID base will be used to uniquely identify the content produced from
   this directory within the system, so it must be unique across *all* content
   repositories that are published to a Deconst cluster. The easiest way to
   accomplish this is to set the content ID base to the content repository's
   GitHub URL (including the trailing slash to be consistent).

   You can specify other settings within this file as well, but they're all
   optional.

   * ``githubUrl``: Set this to the content repository's GitHub URL. If you do,
     it may be used to generate "submit an issue" or "edit on GitHub" links for
     your content.

   * ``githubBranch``: Target "edit on GitHub" links to modify content on a
     branch other than "master".

   * ``preparer``: Set this to the name of a Docker container image that
     contains the preparer for this content. Generally, Deconst will automatically
     infer the preparer to use from the contents of the directory, but you can
     override it explicitly here if needed. The container name must be on a
     whitelist that's controlled by the cluster administrators.

   * ``meta``: An object with arbitrary content that will be merged with
     document-specific metadata. This data will be available to :ref:`templates in
     the control repository <control-template>` beneath the ``meta`` key for extra
     customization. Check the README for the instance's control repository to see
     what keys have meaning for your templates.

   Here's an example of the minimum possible ``_deconst.json`` file:

   .. code-block:: json

      {
        "contentIDBase": "https://github.com/deconst/deconst-docs/"
      }

   Here's another ``_deconst.json`` example, fully populated:

   .. code-block:: json

      {
        "contentIDBase": "https://github.com/deconst/deconst-docs/",
        "githubUrl": "https://github.com/deconst/deconst-docs/",
        "preparer": "quay.io/deconst/preparer-sphinx",
        "meta": {
          "someKey": "someValue"
        }
      }

   One content repository can include many content root directories. Place a
   ``_deconst.json`` file within each one and Deconst will automatically prepare
   the content within each. Make sure that you give each directory a distinct
   content ID base! The easiest way to do this is to append a meaningful suffix
   to the GitHub repository URL for each one, like a version number:

   .. code-block:: json

      {
        "contentIDBase": "https://github.com/deconst/deconst-docs/v1/"
      }

#. **Send a pull request to the control repository to add your content
   repository's name to the automatic build list.** This is a file called
   ``content-repositories.json`` in the root directory of the control repository
   that looks like this:

   .. code-block:: json

      [
        { "kind": "github", "project": "deconst/deconst-docs" },
        { "kind": "github", "project": "myorg/my-content", "branches": ["current", "next"] }
      ]

   Add a new entry to the array with your project's name. Only content pushed to
   the branches listed by the ``branches`` setting will be deployed to
   production. By default, this includes only ``"master"``.

   Once your pull request is merged, a :term:`Strider` build will be created for
   your content repository, and any changes that you make to your repository
   from this point forward will automatically be submitted to Deconst.

At this point, your content is being sent to Deconst, but nobody can see it yet.
The next step is to work with a :ref:`coordinator <site-coordinator>` to decide
on a place your content should live in the context of the larger site.

Where Will Your Content Live
----------------------------

The final output from each content repository will be presented at a subpath of
the complete site. For example, if you create the following pages:

.. code-block:: text

   welcome
   chapter-1/introduction
   chapter-1/getting-started
   chapter-2/more-advanced

And you're currently mapped to the ``books/example/`` subpath of *mysite.com* by
the control repository, then your pages will be available at the following URLs:

.. code-block:: text

   https://mysite.com/books/example/welcome/
   https://mysite.com/books/example/chapter-1/introduction/
   https://mysite.com/books/example/chapter-1/getting-started/
   https://mysite.com/books/example/chapter-2/more-advanced/

As you work, you can freely create new pages and directories and they will
automatically be available within that subpath.

Content that you delete is also automatically deleted from the site. Be careful!
When you rename or delete content, you may break users' existing bookmarks or
links from other sites. Consider copying the content to its new path, creating a
redirect, then deleting it from its old path to avoid disrupting the site's user
experience.

Content mapping is determined by :ref:`content mapping configuration files
<control-map>` within the control repository. Open an issue on the control
repository to discuss the addition of new content, or modify the content mapping
files yourself in a pull request if you're also a site coordinator.

.. _pull-request-builds:

Previewing Changes
------------------

If your content repository is using a :term:`Strider` build, each time you *open
a new pull request* or *push new commits to an existing pull request*, Strider
will build a preview of your work to a staging environment. Once the build is
complete, a bot account will post a comment on your pull request including a
link to your personal preview.

While you're browsing your preview, all page links will be manipulated to keep
you within your personal preview environment, so you can navigate around the
full site without accidentally jumping to production.

.. _custom-content-integration:

Custom Content Repository Integrations
--------------------------------------

While Deconst provides automation to support content repositories that satisfy
the constraints listed above, it's flexible enough to accept content from
virtually anywhere. You can even submit content entirely by manually using
nothing but ``curl`` if you really want to. If your content repository is
different, you'll need to do more work up front.

Whatever is different about your content repository, you'll need two pieces of
information to begin:

* The **content service URL** for the Deconst instance. Generally, this will be
  port 9000 on a domain served by the instance, like
  ``https://deconst.horse:9000``.

* An **API key** issued for you by an administrator. While you can technically
  use a single key for all of your content, I recommend using a distinct key for
  each content repository, because it diminishes the impact of a key being revoked
  and makes it easier to track activity in the logs.


**If your repository is not hosted on github.com,** but is reachable from the
network that the Deconst instance is running on, you'll need to create a custom
Strider build. You can do this for any git-based provider by choosing the
"Manual Add" option under the "Projects" tab:

.. image:: /_images/strider-manual-add.jpg

**If your repository is not reachable from the network** because it's hosted
behind a firewall or **if your repository is not version controlled with git**,
you'll need to configure your own continuous integration solution, like `Jenkins
<https://jenkins-ci.org/>`_. You should set it up to run the appropriate
:term:`preparer` on your content repository each time new work is accepted.

**If your repository is not written in a supported content format,** you'll need
to write a custom :term:`preparer`. Depending on the flexibility and
architecture of the tooling, the difficulty of doing this can vary from "a few
days' work by a developer" to "a lot of time".

.. _supported-formats:

Supported Content Repository Formats
------------------------------------

Each content repository can independently choose a documentation engine that
makes the most sense for the content it contains. You can choose from any format
that has a matching :term:`preparer`. Preparers extend the native documentation
engine to support additional functionality that Deconst needs to integrate their
output with the rest of the system.

.. toctree::

  sphinx
  jekyll
