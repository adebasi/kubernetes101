# Kubernetes 101

This Kubernetes 101 workshop teaches you some basic kubernetes concepts like pods, deployments, and services. (It is partly stolen from https://github.com/aws-samples/aws-workshop-for-kubernetes/tree/master/01-path-basics/103-kubernetes-concepts ;-) and adapted for local usage)

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

## Create your first Pod
Each resource in Kubernetes can be defined using a configuration file. For example, an NGINX pod can be defined with configuration file shown in below:

    $ cat pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: nginx-pod
    labels:
        name: nginx-pod
    spec:
    containers:
    - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

Create the pod as shown below:

    $ kubectl create -f pod.yml
    pod/nginx-pod created

Get the list of pods:

    $ kubectl get pod
    NAME        READY   STATUS    RESTARTS   AGE
    nginx-pod   1/1     Running   0          8s

You can't access the pod from the outside of the cluster at the moment. To do so, you have to forward the pod's port.

    $ kubectl port-forward nginx-pod 8080:80

You can access the nginx page now with your browser on `http://localhost:8080`

Take a look at the nginx container log. You can see the access log there:

    $ kubectl logs nginx-pod
    127.0.0.1 - - [15/Jan/2020:17:06:01 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36" "-"
    127.0.0.1 - - [15/Jan/2020:17:06:01 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8080/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36" "-"
    2020/01/15 17:06:01 [error] 6#6: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "http://localhost:8080/"

## Execute a shell on the running pod
This command will open a TTY to a shell in your pod:

    $ kubectl get pods
    $ kubectl exec <pod-name> -it bash

This opens a bash shell and allows you to look around the filesystem of the container.

* TODO Do something in the container

You can change the image tag e.g. or other parameters and the pod will be recreated (`kubectl edit pod <pod-name>`). But this would cause a downtime. It is close to simply running `docker run` on a server. 

The fun using kubernetes comes with `deployments`!

## Deployments

This command instantiates an nginx container into your cluster, inside a pod, which is managed by a deployment:

    $ kubectl create deployment my-deployment --image=nginx
    deployment.apps/my-deployment created

Get the list of deployments:

    $ kubectl get deployments
    NAME            READY   UP-TO-DATE   AVAILABLE   AGE
    my-deployment   1/1     1            1           12s

Get the list of running pods:

    $ kubectl get pods
    NAME                             READY   STATUS    RESTARTS   AGE
    my-deployment-5595db876d-pgdfk   1/1     Running   0          42s

Get additional details for the pod by using the <pod-name> from the above output:

    $ kubectl describe pod nginx-65899c769f-nd8qk

By default, pods are created in a default namespace. In addition, a kube-system namespace is also reserved for Kubernetes system pods. A list of all the pods in kube-system namespace can be displayed as shown:

    $ kubectl get pods --namespace=kube-system
    NAME                                     READY   STATUS    RESTARTS   AGE
    coredns-6dcc67dcbc-7xxcs                 1/1     Running   1          2d23h
    coredns-6dcc67dcbc-b5tkj                 1/1     Running   1          2d23h
    etcd-docker-desktop                      1/1     Running   1          2d23h
    kube-apiserver-docker-desktop            1/1     Running   1          2d23h
    kube-controller-manager-docker-desktop   1/1     Running   1          2d23h
    kube-proxy-h4d54                         1/1     Running   1          2d23h
    kube-scheduler-docker-desktop            1/1     Running   1          2d23h
    
* TODO: Change Deployment (Replicas, Image, Resources)

## Clean up
Delete all the Kubernetes resources created so far:

    $ kubectl delete deployment my-deployment

In the next sections, we will go into more detail about Pods, Deployments, and other commonly used Kubernetes objects.

## Todo
* Resources: 
