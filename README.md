## About

This repository contains (most of) the object definitions necessary for deploying the `Subway Explorer` application via 
the [Kubernetes](https://kubernetes.io/docs/concepts/storage/volumes/) SaaS platform.

This document is a WIP.

## Quickstart

This guide assumes that you have a working Kubernetes instance somewhere,and an attached copy of `kubectl` running in the command line.

The instructions are somewhat Google Cloud Platform specific because that's where I deployed this application to. I will point out where there may be differences.

### Step 1: Clone this repository

From the root of whatever folder you are performing your configuration from, `git clone` this repository:

    git clone https://github.com/ResidentMario/subway-explorer-deploy.git

### Step 2: Initialize the reverse proxy service

The first service you will need to deploy is the [Google Directions API](https://github.com/ResidentMario/subway-explorer-gmaps-proxy) reverse proxy service, [`subway-explorer-gmaps-proxy`](https://github.com/ResidentMario/subway-explorer-gmaps-proxy).

If you haven't already, log into your Google account. Then, visit the ["Get API Key"](https://developers.google.com/maps/documentation/directions/get-api-key) section of the Google Directions API documentation, and follow the dialogue there to create a new key.

You need to include this key in your application as a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/). Kubernetes requires secrets be encoded in base 64 (it decodes them for you automatically before passing them to the environment). To encode your secret do:

    echo -n YOUR_API_KEY

Open `google-maps-directions-api-key-secret.yaml`. Copy paste the encoded key you got from the operation above into the file, following the following format:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: google-maps-directions-api-secret
type: Opaque
data:
  GOOGLE_MAPS_DIRECTIONS_API_KEY: YOUR_ENCODED_API_KEY
```

Now populate the Kubernetes resources:

```sh
kubectl create -f google-maps-directions-api-key-secret.yaml
kubectl create -f subway-explorer-gmaps-proxy-deployment.yaml
kubectl create -f subway-explorer-gmaps-proxy-service.yaml
```

That's it---the service is deployed! To verify everything went correctly, run `kubectl get pods` and find the `gmaps-proxy` pod. `kubectl exec -it POD_NAME bash` to get inside the pod, then run `wget https://localhost:9000/status; cat status`. If all is well the result will be `{status: "OK"}`. You can then run `rm status` to get rid of the junk file, and then `exit` the pod.

### Step 3: Mount a volume for the database

Next we need to create a disk and mount it. These instructions mostly follow from the ["Add persistent disk"](https://cloud.google.com/compute/docs/disks/add-persistent-disk#create_disk) section of the official documentation. These instructions are GCP specific; if you are using some other cloud provider you need to figure out and follow their procedures for mounting a disk instead.

Start by running the following: 

    gcloud compute disks create subway-explorer-datastore --size 2

Select a node in your cluster which will run the `subway-explorer-api` pod. Any node that isn't already under high load will do. You can pick one by e.g. visiting your [Cloud Console](https://console.cloud.google.com/) and looking around.

Once you have the ID of the node you want to run the database on, follow the instructions at ["Add persistent disk"](https://cloud.google.com/compute/docs/disks/add-persistent-disk#create_disk) on formatting the disk. The `[MNT_DIR]` in this document is the node ID you just looked up.
 
 
Once the disk is formatted, run the following command to attach that disk to the node:

    gcloud compute instances attach-disk subway-explorer-datastore --disk NODE_ID

...substituting `NODE_ID` for the ID of the chosen node. For example, in my case `gke-webapp-default-pool-49338587-d78l` was what I needed. Now you have a persistent disk attached to your node!

### Step 4: Copy the database to the volume

Next you need to copy the database off of your local machine and onto the disc. You can do this using the
[`gcloud scp`](https://cloud.google.com/sdk/gcloud/reference/compute/scp) command (itself a thin wrapper on UNIX 
`scp`, shorthand for "secure copy").

This should look something like this:

    gcloud compute scp ~/Desktop/subway-explorer-api/logbooks.sqlite gke-subway-explorer-default-pool-2bef4d1d-f7dn:/mnt/disks/subway-explorer-datastore

Substituting the node ID for the ID of the volume-mounted node in your own cluster.

For instructions on generating your own database refer to the [`subway-explorer-api`](https://github.com/ResidentMario/subway-explorer-api) `README.md`.

### Step 5: Initialize the database service

Now that you have a database set up, mounted to your chosen node, and seeded wth data, it's time to deploy [`subway-explorer-api`](https://github.com/ResidentMario/subway-explorer-api), the database service.

First we need to assign a label to the node we mounted the data on in the previous step. This will make the node 
locatable. We'll use this label to ensure our database pod is assigned to this node only when it's created (as per 
"[Assinging Pods to Nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)").

To label the node, run the following command:

    kubectl label nodes gke-webapp-default-pool-49338587-d78l datastore_node=true

Substitute in the name of your database node.

Next we need to populate an environment variable in the pod pointing to the database. You can do this by running the 
following:

    kubectl create configmap database-config-map --from-env-file=api-database-config-map.env

Now deploy the API service.

```sh
kubectl create configmap database-config-map --from-env-file=api-database-config-map.env
kubectl create -f subway-explorer-api-deployment.yaml
kubectl create -f subway-explorer-api-service.yaml
```

To check that things are operating correctly, step into the pod (substitute the name for your pod name):

```sh
kubectl exec -it subway-explorer-api-deployment-7889bf8988-zl82b bash
```

To check that the environment variable is populating correctly:

```sh
node -e "console.log(process.env.DATABASE_FILEPATH)"
```

To check that the container is functioning correctly:

```
npx mocha test
```

### Step 6: Initialize the front-end application

It's finally time to spin up the application front-end, [`subway-explorer-webapp`](https://github.com/ResidentMario/subway-explorer-webapp).

TODO: Routing.

Now deploy the application.

```sh
kubectl create configmap webapp-config-map --from-env-file=webapp-config-map.env
kubectl create -f subway-explorer-api-deployment.yaml
kubectl create -f subway-explorer-api-service.yaml
```
```
