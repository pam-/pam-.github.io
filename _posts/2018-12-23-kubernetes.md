---
layout: post
title: "Kubernetes — why?"
type: code
image: "https://media.giphy.com/media/xT9DPiWlDIld3CLnqg/giphy.gif"
date: 2018-12-11 14:10 -0800
---

In the spirit of [KubeCon](https://events.linuxfoundation.org/events/kubecon-cloudnativecon-north-america-2018) and all the Kubernetes content I consumed, I thought I’d write a short explainer on, you guessed it, [Kubernetes](https://kubernetes.io/). This post aims to give a high-level overview of why and how Kubernetes is helpful. My hope is that it’ll give you enough knowledge to dive deeper into the [docs](https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational).

Without further ado, let’s dive into it.

![diving-gif](https://www.dropbox.com/s/1buhjqa62nt45la/diving.gif?raw=1)  
_lol_


## Why Kubernetes?

Kubernetes, pronounced _koo-ber-nay-tees_, is the container [HBIC](https://www.urbandictionary.com/define.php?term=HBIC). To be honest, I didn’t understand what the big deal was until I joined the infrastructure team. If, like me back then, you wonder why so many people rave about it, this post is for you.

**Vox engineering story time!**

![storytime-gif](https://www.dropbox.com/s/ttnuwuiskyh1e67/storytime.gif?raw=1)

For a bit, we had our applications running on a VM and eventually moved them all to Docker containers. It improved the development cycle and, despite the obscure bugs that some devs ran into, made for an overall happier engineering team. We were on our way to the developer’s utopia, but not quite there yet. In fact, a lot more things were needed. Let’s look at a few of them:

**_connecting multiple applications on local environments_**  
While developing, folks wished to have several applications running and talking to one another. While Docker made running many containers on one machine possible, it took a toll on memory and usually led to a really angry computer fan.

**_providing an (even) faster development cycle_**  
To prepare for our annual hackathon, we wished to create a world where new applications could be spun up quickly on staging and production and where all environments were very similar, if not identical.

**_preventing production bugs_**  
We wanted to anticipate production issues on reliable staging environments.

and more.

## Enter Kubernetes


![hbic-gif](https://www.dropbox.com/s/z6sqea6oj7qd1d8/ezgif.com-gif-maker.gif?raw=1)  
_the HBIC, as stated earlier._

Kubernetes (K8s) is a container orchestration tool that alleviates developers from doing a lot of hands-on management with their contained applications. Using the list of problems from earlier, let’s look at how it can solve for each:

**_connecting multiple applications on local environments_**  
No need with [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)! K8s namespaces are virtual clusters where an engineer can run multiple containers and remove the load from her own machine. She can run container A locally, work on it, and know that it can reliably connect to containers B and C running on a namespace.

**_providing an (even) faster development cycle_**  
With namespaces, K8s provides the ability to deploy applications' containers quickly on any sandbox, staging, or production environment. A developer must specify the necessary information such as containers info, dependencies, etc… in k8s [objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) and create them in a namespace. An [object](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) is essentially a `yaml` file with crucial information pertaining to a container or a cluster. In the case of an application, a [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) object can be created or applied into the namespace along with an [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) object. Once applied, k8s will run the container, provide a hostname for the application using the ingress object, and based on the deployment object’s specifications, it will monitor the app’s health.

**_predicting production bugs_**  
In order to work, K8s needs a running cluster. A cluster is essentially a group of computers divided between masters and workers nodes. If the staging and production environments are running on the same (or identical but separate) clusters, bugs that happen on staging are likely to happen on production.
With its auto-healing abilities, k8s is also a great tool for chaos engineering, a philosophy that tests varied disastrous scenarios on staging and production environments.

<br/>
<br/>

****

<br/>
<br/>

To recap, K8s makes it easier to bridge multiple applications regardless of the environment, it is a helpful tool for testing and preventing production calamities, and it ameliorates the development cycle so that applications can be shipped faster. That’s not all! For example, K8s also alleviates application management with its ability to load-balance, auto-scale, and generally monitor containers health. It does a lot for developers, hence its popularity.

I hope this was somewhat helpful. I’m off to building my own cluster on [EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)! I’ll document the process and blog about it when it is done. Stay tuned and…wish me luck!

Until then 👋

![bye](https://www.dropbox.com/s/ersmbed7rwtu7xh/bye.gif?raw=1)