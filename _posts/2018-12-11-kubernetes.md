---
layout: post
title: "Yet Another Kubernetes Breakdown"
type: code
image: "https://giphy.com/embed/xT9DPiWlDIld3CLnqg"
date: 2018-12-11 14:10 -0800
---

<iframe src="https://giphy.com/embed/xT9DPiWlDIld3CLnqg" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

In the spirit of [KubeCon](https://events.linuxfoundation.org/events/kubecon-cloudnativecon-north-america-2018) and all the Kubernetes content I consumed, I think it is timely to write a short explainer on [Kubernetes](https://kubernetes.io). This post aims to give a high-level overview of what Kubernetes is and does. My hope is that it'll give you enough knowledge to dive deeper into the [docs](https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational).

Without further ado, let's jump into it.

### What and why Kubernetes?

Kubernetes, pronounced koo-ber-nay-tees and also known as K8s, is the container [HBIC](https://www.urbandictionary.com/define.php?term=HBIC). In more technical terms, it is a container orchestration tool that alleviates developers from doing a lot of hands-on management with their containered applications.

Why do so many people rave about it? I'm glad you ask.

**Vox engineering storytime!**

For a bit, we had our applications running on a VM and eventually moved them all to Docker containers. It improved the development cycle and, despite the obscure bugs that some devs ran into, made for an overall happier engineering team. That was great! Then, as always, other issues came to the surface:

* As a developer, I would love to have all my applications running and connected to one another. I can do that with Docker! but OMG, running all these containers takes up a lot of memory. My fan is yelling at me. Halp.
* I really would like to anticipate production issues on my local machine... I'd rather not do a canary deploy and die a little when I press the deploy button and pray that my dedicated server has been setup properly and that I won't deploy to all the other servers by accident. Yikes. Why am I _still_ sweating.
* Is their a way for me to mimic production traffic on other environments??
* I'd love to be able to spin up a prototype for a new microservice easily and smoothly. I'm also not sure how much traffic it'll receive just yet, but I don't want to worry about load balancing and all that stuff... How does one go about that?

Enter Kubernetes. _The_ tool that will help out with all these issues, and more, _across environments_.

### Kubernetes, a shallow deep dive

Let's start at the beginning.

K8s runs on a cluster (essentially, a group of computers) comprised of a Master node, and puppy nodes\*. All nodes possess features that enable them to fix our dev lives better than Iyanla ever could.


#### The Master Node

The master node runs the ship. It provides the cluster's control plane, decides what to do on the cluster, detects various cluster events, and responds to said events when necessary.

Here are its components:

**kube-apiserver:**

The component that exposes the K8s API. It is the front end of everything that happens behind the scenes, aka the K8s control plane.
It is designed to scale horizontally, which means that it scales by deploying more instances of the master node. The opposite would be scaling vertically and adding more CPU or RAM to an exisiting machine.

**kube-scheduler:**

The component that assigns a node to a newly created pod when it doesn't have one pre-assigned.

**etcd:**

Key value store used as K8s backing store for all cluster data. It should always be backed up because shit _will_ happen.

**kube-controller-manager:**

The component that runs controllers. Each controller ensures that the cluster is always in the desired state. If a node goes down, the Node controller will do its best to bring it back up, for example. For more details on controllers, click here.

**cloud-controller-manager:**

This component is similar to the kube-controller-manager, except on the cloud. It will follow cloud provider specific requirements and ensure that the desired settings are stable. To re-use the NodeController example, it will check whether a node has gone down on the cloud.

#### The puppy nodes

These nodes all share the same components:

**kubelet**

An agent that ensures containers are running in a pod. It only worries about containers that were created by Kubernetes with K8s config files.

**kube-proxy**

Networking is my weakness, so this is a tough one. But as far as I understand, the kube-proxy enables the networking rules set in K8s services and performs all and any connection forwarding.

**container runtime**

This is a software that is responsible for running containers vs the kubelet agent which just checks that a container _is_ running.

## Setting up the cluster

There are a few ways for you to set up a cluster. You can:

- Set up a managed cluster on a cloud provider such as EKS on Amazon, GKE on Google, or AKS with Azure. [Check out this link](https://blog.hasura.io/gke-vs-aks-vs-eks-411f080640dc) to read the differences between each provider.
- Build your own multi-nodes cluster from [scratch](https://kubernetes.io/docs/setup/scratch/).
- Try things out locally with [Minikube](https://kubernetes.io/docs/setup/minikube/), the k8s tool that provides you with a local cluster made out of a single node.

## Kubectl

Kubectl, pronounced kyoob control, or kyoob cuddle for some, is the K8s command line tool. In order to use it, you must install it as explained in [this setup](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

With it you will be able to interact with your cluster, as well as deploy, rollback, and generally manage your applications with Kubernetes objects.

## Kubernetes Objects

A Kubernetes object is, at its essence, a config file written in `yaml`. With it, you tell Kubernetes what you want running on your cluster, and _how_ you want it to run. Using Kubectl and the Kubernetes API, you can create, modify, or delete objects as needed for your cluster's desired state.

Good to note that you can use other [client libraries](https://kubernetes.io/docs/reference/using-api/client-libraries/) outside of Kubectl to run commands with the K8s API.

When writing the configuration for a Kubernetes objects, remember to add a `spec` field that'll include basic info like a name, and a desired state indicating info on needed replicas, or ports. Below is an example, borrowed from the official docs.

{% highlight yaml %}

#deployment.yaml

apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

{% endhighlight %}

There is a long list of Kubernetes objects but deployments are, as far as I know, the most talked about. If, like me when I first got exposed to k8s, you are confused about what the hell a deployment is, stay tuned for more k8s posts where I'll dive deeper into objects and general k8s terminology!

<iframe src="https://giphy.com/embed/v5eQoZqI6sDyE" width="480" height="341" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>


That's all for now.
Until next time! <3

<br/>
<br/>
<br/>
<br/>
<br/>
<hr/>

\* this is NOT a technical term. I just like the sound of it better.
