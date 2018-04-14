# google-gke-couchbase

This is a walkthrough of setting the [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) up on [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/).

## Setup glcoud CLI

You will need a GCP account.  We also need to install glcoud.  Instructions for installing the Google Cloud SDK that includes gcloud are [here](https://cloud.google.com/sdk/).

To set up your Google environment, run the command (be sure to select a default region):

    gcloud init

## Deploy a GKE Cluster

    gcloud container clusters create mycluster --machine-type=n1-standard-4    

## Configure kubectl

Now that we have a cluster, the next step is to install and set up kubectl up so it can connect to the cluster.

    gcloud components install kubectl
    gcloud container clusters get-credentials mycluster

    kubectl get nodes

That should show three nodes:

![getnodes](/images/GKE_getnodes.png)

GKE includes sophisticated RBAC to limit permissions.  The Operator creates resources in your cluster.  We'll need to grant it permissions to do so.

    kubectl create clusterrolebinding cluster-admin-binding \
      --clusterrole cluster-admin \
      --user $(gcloud config get-value account)

    kubectl create clusterrolebinding my-couchbase \
      --clusterrole=cluster-admin \
      --serviceaccount=default:default

## Deploying the Operator

Once you have an GKE cluster deployed and a running kubectl, you're ready to deploy the Operator.  The documentation on that is [here](http://docs.couchbase.com/prerelease/couchbase-operator/beta/overview.html).

To create the deployment and check it deployed, run this:

    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/operator.yaml
    kubectl get deployments

You should see something like this:

![operatordeployed](/images/GKE_operator-get_deployments.png)

## Deploying a Couchbase Cluster

We're there!  Time to get a live cluster.  Run this:

    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/secret.yaml
    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/couchbase-cluster.yaml

That should give this:

![couchbasecreated](/images/GKE_Cluster_creation.png)

You can view the Couchbase and operator pods by running:

    kubectl get pods

## Accessing the Couchbase Web UI

You've now got a cluster.  But to use it you probably want to set up port forwarding.  To do that run:

    kubectl port-forward cb-example-0000 8091:8091

Leave that command running:

![portforward](/images/GKE_port_forward.png)

Now open up a browser to http://localhost:8091

![loginscreen](/images/GKE_loginscreen.png)

The username is `Administrator` and password is `password`.  And now you're in!

![webui](/images/GKE_webui.png)
