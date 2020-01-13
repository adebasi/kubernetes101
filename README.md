# Kubernetes 101

This Kubernetes 101 workshop teaches you some basic kubernetes concepts like pods, deployments, and services. (And it is mostly stolen from https://github.com/aws-samples/aws-workshop-for-kubernetes/tree/master/01-path-basics/103-kubernetes-concepts ;-) )

## Prerequisite
We will use a local running kubernetes cluster in this workshop. An easy way to get a kubernetes cluster up and running is via Docker for Mac. We also need a command line tool to interact with kubernetes, kubectl. 
* Install (via brew: https://brew.sh/index_de)
  * Docker (`brew cask install docker`)
  * Kubectl (`brew install kubernetes-cli`)
* Start Docker
* Start Kubernetes (via Docker: Click Docker icon -> Kubernetes -> Enable local cluster)


## Introduction 
Now that we have a cluster up and running we can start exploring the Kubernetes CLI via the `kubectl` (pronounced "cube control") command.

`kubectl` interacts with the Kubernetes API Server, which runs on the master nodes in the cluster.

Kubernetes as a platform has a number of abstractions that map to API objects. These Kubernetes API Objects can be used to describe your cluster's desired state - including information such as applications and workloads running, container images, networking resources, and more. This section explains the most-used Kubernetes API concepts and how to interact with them via `kubectl`.

## Display Nodes

This command will show all the nodes available in your kubernetes cluster:

    $ kubectl get nodes

It will show an output similar to:

    NAME             STATUS   ROLES    AGE   VERSION
    docker-desktop   Ready    master   27h   v1.14.8

## Todo
* Start a pod
* Access pod
* Get in touch with a deployment manifest file and apply it
* Scale deployment
* Rolling update an application
