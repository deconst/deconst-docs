.. _webhooks:

Webhooks and Integration
------------------------

Deconst uses a combination of webhooks and continuous integration "builds" to stay up to date with changes made to the control and content repositories. Although it *should* manage them itself, if anything isn't updating correctly, these are the first places you should check.

You can verify that they're installed correctly by visiting the ``settings/hooks`` page within the relevant GitHub repository.

These are the integrations that need to be installed on a **control repository**:

 * A ``.travis.yml`` file that clones and executes the *asset preparer*. It should have the following contents:

  .. code-block:: yaml

    ---
    language: node_js
    node_js:
    - "0.12"
    install:
    - git clone --depth 1 https://github.com/deconst/preparer-asset.git /tmp/preparer-asset
    script:
    - /tmp/preparer-asset/build.sh

  .. end the code block.

These are the integrations that need to be installed on each **content repository**:

 * A ``.travis.yml`` file that clones and executes the appropriate :term:`preparer` for that repository type. Here's an example for a Sphinx repository:

  .. code-block:: yaml

   ---
    language: python
    python:
    - "3.4"
    install:
    - "pip install -e git+https://github.com/deconst/preparer-sphinx.git#egg=deconstrst"
    script:
    - deconst-prepare-sphinx

.. end the code block.
