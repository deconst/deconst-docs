[![Build Status](https://build.deconst.horse/deconst/deconst-docs/badge?branch=master)](https://build.deconst.horse/deconst/deconst-docs/)

# Deconst documentation

Documentation for the *deconst* project: a continuous delivery pipeline for heterogenous documentation. It's also used to develop deconst itself, like some kind of documentation ouroboros. You can read the documentation itself at [deconst.horse](https://deconst.horse/).

## Building

To build this documentation standalone, use the [Deconst client](https://github.com/deconst/client). Use the Sphinx preparer and a clone of `https://github.com/deconst/deconst-docs-control` as the control repository.

### DNS and TLS

DNS entries for `deconst.horse`, `build.deconst.horse`, `staging.deconst.horse`, and `content.staging.deconst.horse` are managed by Cloud DNS entries in the "drgsites" account. They should be pointed to the appropriate load balancers.

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

# Deconst Dev Env in Kubernetes with Minikube

These instructions will prepare and submit the content and assets for this deconst documentation in a dev env in Kubernetes with Minikube.

1. If necessary, deploy the [presenter service](https://github.com/deconst/presenter#deconst-dev-env-in-kubernetes-with-minikube)

1. Open a new shell

1. Prepare the content

    ```bash
    wget https://raw.githubusercontent.com/deconst/preparer-sphinx/master/deconst-preparer-sphinx.sh
    chmod u+x deconst-preparer-sphinx.sh

    ./deconst-preparer-sphinx.sh

    ls _build/deconst-envelopes/
    ls _build/deconst-assets/
    ```

1. Submit the content

    The `CONTENT_SERVICE_APIKEY` must match the `ADMIN_APIKEY` set when deploying the [content service](https://github.com/deconst/content-service#deconst-dev-env-in-kubernetes-with-minikube).

    ```bash
    export CONTENT_SERVICE_URL=$(minikube service --url --namespace deconst content)
    export CONTENT_SERVICE_APIKEY=$ADMIN_APIKEY

    wget https://raw.githubusercontent.com/deconst/submitter/master/deconst-submitter.sh
    chmod u+x deconst-submitter.sh

    ./deconst-submitter.sh
    ```

1. Prepare the staging content

    The [staging environment](https://deconst.horse/developing/staging/) is a specially configured content service and presenter pair that allow users to preview content. Normally the Strider server will push preview content to the staging environment but this is how you would manually do it.

    ```bash
    export CONTENT_ID_BASE=https://github.com/staging/deconst/deconst-docs/
    export ENVELOPE_DIR="_build/deconst-staging-envelopes"
    export ASSET_DIR="_build/deconst-staging-assets"

    ./deconst-preparer-sphinx.sh

    ls _build/deconst-staging-envelopes/
    ls _build/deconst-staging-assets/
    ```

1. Submit the staging content

    ```bash
    export CONTENT_SERVICE_URL=$(minikube service --url --namespace deconst staging-content)
    export ENVELOPE_DIR="$(pwd)/_build/deconst-staging-envelopes"
    export ASSET_DIR="$(pwd)/_build/deconst-staging-assets"

    ./deconst-submitter.sh
    ```

1. Confirm the content envelopes are in mongo

    ```bash
    kubectl run --namespace deconst --rm -it mongo-cli --image=mongo:2.6 --restart=Never -- mongo mongo.deconst.svc.cluster.local
    show dbs
    use content
    db.envelopes.count()
    db.staging_envelopes.count()
    ```

1. Deploy the [deconst control repo](https://github.com/deconst/deconst-docs-control#deconst-dev-env-in-kubernetes-with-minikube)

1. Recreate the content DB

    The content is stored in the mongo DB files at `/data/deconst/mongo` in the minikube VM. Delete the DB files and the mongo pod. Kubernetes will automatically restart the mongo pod.

    ```bash
    minikube ssh
    sudo rm -rf /data/deconst/mongo/*
    exit
    MONGO_POD_NAME=$(kubectl get pods --selector name=mongo --output=jsonpath={.items..metadata.name} --namespace deconst)
    kubectl delete po/$MONGO_POD_NAME --namespace deconst
    ```

    Now you can run through the instructions above again to prepare and submit the content.
