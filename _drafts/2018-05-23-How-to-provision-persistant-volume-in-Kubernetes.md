---
layout: post
title: How to provision persistant volume in Kubernetes.
feature-img: "assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/Storage-Banner.jpg"
thumbnail: "assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/Storage-Banner.jpg"
excerpt_separator: <!--more-->
tags: [Docker, Container, Kubernetes]
---

I recently got the chance to speak at DevoxxFR. I loved the event, and even more the chat I had in the hallway track. I went there to talk about the different ways of `managing stateful applications via Kubernetes`, this talk raised a lot of questions which inspired me to write this blog post.
<!--more-->

Most of this questions were about the `persistant volume` provisionning, so let's dive into this subject.

---

Basically there are 2 ways of managing it:

- `Manually creating the persistant volume`, which is leading to you creating all the persistant volume on your own (the last thing you want to do)
- `Automatically creating the persistant volume` by letting Kubernetes talk with your storage backend so that it can provision itself what he needs

If you do not know what is a `persistant volume`, that's ok <i class="fa fa-smile-o"></i>. Let say that you can see it as a network disk or a network drive, whatever rings a bell in your mind.

Some would argue that there is also the `statefulset` object and I would answer that from a state perspective the `statefulset` is using the automatic creation of the `persistant volume`, nothing more. Hoever `statefulset` are providing a number of feature usually useful when dealing with stateful applications. The most useful one being the ordered deployment and deletion of pods.

---

# Let set some context

To make it easier to understand, let put some context here. Let's say that we are in a company called `BigCo`. `BigCo` has already rolled out `Kubernetes` to manage all services it provides. `John` and `Bob` are 2 employees of `BigCo`. `John` is a sysadmin and `Bob` is a developer. One day `Bob` come to `John` with an ask, he would need to run a MySql database in the `Kubernetes` cluster hosting his application.

# Manually creating the persistant volumes

After some research `John` is ready to share a small POC with `Bob`. `John` just want to validate the concept, therefore he choose to create the persistant manually himself and don't get into the automatic persistant volume creation.

## The flow

So here is how the provisionning process work in this situation:

1. `BigCo` has a `Kubernetes` cluster and a storage backend up and running. They can both communicate together.

![PV+PVC-18]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/PV+PVC-18.jpg)
2. `John` create a `persistant volume`.

![PV+PVC-19]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/PV+PVC-19.jpg)
3. `Bob` create a `persistant volume claim` referencing the `persistant volume` previously created

![PV+PVC-20]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/PV+PVC-20.jpg)
4. `Bob` create a `pod` and reference the `persistant volume claim` in this pod

![PV+PVC-21]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/PV+PVC-21.jpg)
5. `Kubernetes` mount the `persistant volume` in the pod

![PV+PVC-22]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/PV+PVC-22.jpg)

## The result

Now if the pod dies and is rescheduled somewhere else, the data will not be lost.

## Limitation

`John` had to create a persistant volume before `Bob` could do his job, in `BigCo`, that is usually meaning the `Bob` had to submit a ticket to `John`. That is usually source of slowdow in the process, which we want to avoid.

# Automatically creating the persistant volumes

## Which Kubernetes component will be used in this scenario

## The flow

So here is how the provisionning process work in this situation:

1. Again, `BigCo` has a `Kubernetes` cluster and a storage backend up and running. They can both communicate together

![DP-24]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/DP-24.jpg)
2. `John` create one or many `storage class`

![DP-25]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/DP-25.jpg)
3. `Bob` create a `persistant volume claim` refering this time to one of the `storage class`

![DP-26]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/DP-26.jpg)
4. `Bob` create a `pod` and reference the `persistant volume claim` in the `pod`

![DP-27]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/DP-27.jpg)
5. `Kubernetes` create the `persistant volume` based on the `persistant volume claim` and the `storage class` choosen

![DP-28]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/DP-28.jpg)
6. `Kubernetes` mount the `persistant volume` in the `pod`

![DP-29]({{ site.baseurl }}/assets/img/posts/2018-05-23-How-to-provision-persistant-volume-in-Kubernetes/DP-29.jpg)

## The result

Same thing as previously, if the pod dies and is rescheduled somewhere else, the data will not be lost.

Once the many `storage class` have been created by `John`, `Bob` can create whithout delay the `persistant volume` he need.

## Limitation

As `persistant volume` are dynamically created, and some people tend to forget to manage the all lifecycle, which can led to `persistant volume` not deleted even if they are not used anymore. Hoever, this can be easily worked around by reviewing from time to time the list of `persistant volume` instentiated and not linked to a `pod`.

# The talk material

1. [the github link](https://github.com/woernfl/k8s-stateful-demo)
2. the full slidedeck

<script async class="speakerdeck-embed" data-id="4c9e38395a8d45dcbb08dda95242c7fd" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>