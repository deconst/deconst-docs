[![Build Status](https://build.deconst.horse/deconst/deconst-docs/badge?branch=master)](https://build.deconst.horse/deconst/deconst-docs/)

# Deconst documentation

Documentation for the *deconst* project: a continuous delivery pipeline for heterogenous documentation. It's also used to develop deconst itself, like some kind of documentation ouroboros. You can read the documentation itself at [deconst.horse](https://deconst.horse/).

## Building

To build this documentation standalone, use the [Deconst client](https://github.com/deconst/client). Use the Sphinx preparer and a clone of `https://github.com/deconst/deconst-docs-control` as the control repository.

### DNS and TLS

DNS entries for `deconst.horse`, `build.deconst.horse`, `staging.deconst.horse`, and `content.staging.deconst.horse` are managed by Cloud DNS entries in the "dse.ashwilson" account. They should be pointed to the appropriate load balancers.

TLS certificates are currently retrieved from Let's Encrypt by a manual, downtime-inducing process. To reissue them:

* Run `script/reissue` in your [deconst/deploy](https://github.com/deconst/deploy) clone to reissue and download the new certificates to `le_certificates/`.
* Copy and paste the new certificate as the `ssl_key:` entry in `credentials.yml`.
* Use `script/encrypt` to encrypt the modified credentials file and commit and push it to nexus-credentials.
* Invoke Ansible to roll it out:

```bash
$ script/deploy \
   -e 'credentials_update=true' \
   -e 'service_pod_restart=true' \
   -e 'staging_restart=true' \
   -e 'strider_restart=true'
```
