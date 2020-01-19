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

Normally a Kubernetes cluster has master and worker nodes. The Kubernetes setup provided by Docker for Mac only has a master node, which also does a worker's job.

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

### Port forwarding

You can't access the pod from the outside of the cluster at the moment. To do so, you have to forward the pod's port.

    $ kubectl port-forward nginx-pod 8080:80

You can access the nginx page now with your browser on `http://localhost:8080`


### Logs

Take a look at the nginx container log. You can see the access log there:

    $ kubectl logs nginx-pod
    127.0.0.1 - - [15/Jan/2020:18:42:37 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.54.0" "-"
    127.0.0.1 - - [15/Jan/2020:18:43:26 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.54.0" "-"
    127.0.0.1 - - [15/Jan/2020:18:43:45 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36" "-"

### Execute a shell on the running pod

This command will open a TTY to a shell in your pod. It is like SSH for containers.

    $ kubectl exec nginx-pod -it bash

This opens a bash shell and allows you to look around the filesystem of the container.

* TODO Do something in the container

### Editing the pod

You can change the image tag e.g. or other parameters and the pod will be recreated (`kubectl edit pod nginx-pod`). But this would cause a downtime. It is kind of simply running `docker run` on a server. 

The fun using kubernetes comes with `deployments`!


## Deployments

This command instantiates an nginx container in your cluster, inside a pod, which is managed by a deployment:

    $ kubectl create -f deployment.yml
    deployment.apps/nginx-deployment created

Get the list of deployments:

    $ kubectl get deployments
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           11s

Get the list of running pods:

    $ kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    nginx-deployment-f76fdbc94-6wplq   1/1     Running   0          65m
    nginx-deployment-f76fdbc94-7gj8h   1/1     Running   0          65m
    nginx-deployment-f76fdbc94-wspj5   1/1     Running   0          65m

Get additional details for the pod by using the <pod-name> from the above output:

    $ kubectl describe pod nginx-deployment-f76fdbc94-wspj5

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


## Service

A pod is ephemeral. Each pod is assigned a unique IP address. If a pod that belongs to a replication controller dies, then it is recreated and may be given a different IP address. Further, additional pods may be created using Deployment or Replica Set. This makes it difficult for an application server, such as WildFly, to access a database, such as MySQL, using its IP address.

A Service is an abstraction that defines a logical set of pods and a policy by which to access them. The IP address assigned to a service does not change over time, and thus can be relied upon by other pods. Typically, the pods belonging to a service are defined by a label selector. This is similar mechanism to how pods belong to a replica set.

This abstraction of selecting pods using labels enables a loose coupling. The number of pods in the deployment may scale up or down but the application server can continue to access the database using the service.

A Kubernetes service defines a logical set of pods and enables them to be accessed through microservices.

### Create a Service

You can create a service with a manifest file or let `kubectl` do the work:

    $ kubectl expose deployment nginx-deployment --type=NodePort
    service/nginx-deployment exposed

    $ kubectl get service
    NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        3d
    nginx-deployment   NodePort    10.107.148.24   <none>        80:30849/TCP   5m44s

Kubernetes provided by Docker for Mac lets you access NodePort services via `localhost`. The service in the example can be accessed by `localhost:30849`. Give it a try.

You can also take a look at the created manifest file.

    $ kubectl get service nginx-deployment -o yaml

## Tasks
* TODO: Change Deployment (Replicas, Image, Resources)

## Clean up
Delete all the Kubernetes resources created so far:

    $ kubectl get all
    $ kubectl delete deployment my-deployment
    $ kubectl delete service nginx-deployment
    $ ...
