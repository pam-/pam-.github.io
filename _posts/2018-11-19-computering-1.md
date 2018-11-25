---
layout: post
title: "computering #00"
subtitle: "an exposing series about TIL. small and big."
type: code
date: 2018-11-19 13:21 -0500
---

## #00: k8s debugging

#### The deets

I made a basic [Rails][rails] test application that worked as expected locally but wouldn't load once its [Helm chart][helm] got deployed to a [Kubernetes][kubernetes] [namespace][namespace].
The application's pod kept crashing and getting stuck on `crashLoopBackOff`, an error I hadn't personally dealt with in my short [Kubernetes][kubernetes] experience.

After hours of confused looks, mumbled profanities, some furious googling, and a walk, here's what worked:

**1. Getting the Logs**

I ran `kubectl -n <namespace-name> describe pod <problematic-pod-name>` and saw that the `initContainer` was to blame for the failing pod. For context, my pod was depending on the `initContainer` to finish running its job, before it could run as expected. I asked my girl Google about `crashLoopBackOff and initContainer` and she kindly suggested I check the failing `initContainer`'s logs with `kubectl -n <namespace-name> logs <problematic-pod-name> -c <problematic-init-container>`. Lo and behold, the logs revealed that my charts were missing the job for the `initContainer` because I had somehow forgotten to add it........... :woman_facepalming: but hooray for quick fixes!

**2. Readiness Probe Endpoint**

I added the job. Redeployed my chart, watched the `initContainer` complete its job, ran `kubectl -n <namespace-name> describe pod <problematic-pod-name>`, and womp! Still broken. This time, although showing a `Running` status, the pod came out as `Unhealthy` and consequently didn't have any replicas working. It was unhealthy because the [readiness probe][readiness-probe] kept failing.

By definition, the probe hits an endpoint to check that the container is up and running. In my case, you guessed it, the route endpoint was missing. :woman_facepalming: but yay to another easy fix! Since I was using Rails, all it took was adding the route in `config/routes.rb`, and returning a `json` object in the route's controller action in order to avoid creating an obsolete view for it.


#### Commands used
{% highlight bash %}
# get status of all the pods in a given namespace
kubectl -n <namespace-name> get pod

# get clearly formatted details about a specific pod in a namespace
kubectl -n <namespace-name> describe pod <problematic-pod-name>

# get pod logs
kubectl -n <namespace-name> logs <problematic-pod-name>

# get logs of a pod's initContainer
kubectl -n <namespace-name> logs <problematic-pod-name> -c <problematic-init-container>
{% endhighlight %}

#### Lessons
- you _can_ get the logs of an initContainer. Hadn't occured to me until this issue because I don't think that much about k8s except when I use it.
- human error will forever keep me humble. _Forever_.

[rails]: https://rubyonrails.org/
[helm]: https://docs.helm.sh/developing_charts/
[kubernetes]: https://cloud.google.com/kubernetes-engine/kubernetes-comic/
[namespace]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[readiness-probe]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/
