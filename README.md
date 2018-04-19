# google-gke-couchbase

This walkthrough will illustrate how simple it is to setup and manage Couchbase clusters using the [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) within the [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/) on the [Google Cloud Platform (GCP)](https://cloud.google.com/sdk/). 

## Install and Setup the gcloud CLI

We will need a [GCP](https://cloud.google.com/sdk/) account and then we will [install glcoud](https://cloud.google.com/sdk/).

## Setup your GCP Environment
We setup our environment with a single command (be sure to choose a default region):

    gcloud init

## Deploy a GKE Cluster
Deploying a GKE cluster is straightfoward, using the command:

    gcloud container clusters create mycluster --machine-type=n1-standard-4    

## Install and Setup kubectl

Now we have to install and set up kubectl, so it can connect to the GKE cluster. Two commands will do that for us:

    gcloud components install kubectl
    gcloud container clusters get-credentials mycluster

![getcredentials](/images/GKE_get_credentials.png)

We verify our kubectl with:

    kubectl get nodes

We now see that three node are running:

![getnodes](/images/GKE_getnodes.png)

## Setup GKE Role-based Access Control (RBAC)
GKE supports RBAC in order to limit permissions.  Since the Couchbase Operator creates resources in our GKE cluster, we will need to grant it the permission to do so.  

We do that with the following commands:

    kubectl create clusterrolebinding cluster-admin-binding \
      --clusterrole cluster-admin \
      --user $(gcloud config get-value account)

    kubectl create clusterrolebinding my-couchbase \
      --clusterrole=cluster-admin \
      --serviceaccount=default:default

## Deploying the Couchbase Operator

Now that we have all the environment prerequisites in place, we are ready to deploy the Couchbase Operator.

We deploy the Couchbase Operator and then verify it with the following commands:

    kubectl apply -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/operator.yaml
    kubectl get deployments

We now can see that the Couchbase Operator is deployed.

![operatordeployed](/images/GKE_operator_get_deployments.png)

## Deploying a Couchbase Cluster

We are now in the final stretch!

Next we will create a Couchbase cluster with the following commands:

    kubectl apply -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/secret.yaml
    kubectl apply -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/couchbase-cluster.yaml

We should be able to see something like this:

![couchbasecreated](/images/GKE_cluster_created.png)

We can view the all of the pods by running:

    kubectl get pods

![getpods](/images/GKE_get_pods.png)

## Accessing the Couchbase Web UI

Now we have a Couchbase cluster running!
To use the web console we will need setup port forwarding.  

We do that with the kubectl command:

    kubectl port-forward cb-example-0000 8091:8091

We need to make sure we leave the command running in the terminal:

![portforward](/images/GKE_port_forward.png)

Now we can open up a browser at http://localhost:8091

![loginscreen](/images/GKE_loginscreen.png)

We will login using username=`Administrator` and password=`password`.  

We are in! Click the 'Servers' link on the left side and we should see our clusters running.

![webui](/images/GKE_webui.png)

Fin
