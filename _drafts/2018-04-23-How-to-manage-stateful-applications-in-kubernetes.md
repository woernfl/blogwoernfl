---
layout: post
title: How to manage stateful applications in Kubernetes.
feature-img: "assets/img/posts/2018-04-23-How-to-manage-stateful-applications-in-kubernetes/Storage-Banner.jpg"
thumbnail: "assets/img/posts/2018-04-23-How-to-manage-stateful-applications-in-kubernetes/Storage-Banner.jpg"
excerpt_separator: <!--more-->
tags: [Docker, Container, Kubernetes]
---

I recently got the chance to speak at the DevoxxFR. I loved the event, and even more the chat I had in the hallway track.
<!--more-->

I went there to talk about the different ways of managing stateful applications with Kubernetes.

Basically there are 2 ways of doing it:
- Manually creating the persitant volumes, which is leading to you creating all the persistant volume on your own (the last thing you want to do) (Slides 18 to 22 of the attached presentation)
- Automatically creating the persitant volumes by letting Kubernetes talk with your storage backend so that he can provision himself what he needs (Slides 24 to 29 of the attached presentation)

If you do not know what is a persitant volume, that's ok <i class="fa fa-smile-o"></i>. Let say that you can see a persistant volume as a network disk or a network drive, whatever rings a bell in your mind.

Let see what is happening in each cases. 

# Before we begin

To make it easier to understand, let put some context here. Let's say that we are in a company called BigCo. BigCo has already rolled out Kubernetes to manage all the services it is providing. John and Bob are 2 employees of BigCo. John is a Sysadmin and Bob is a developer. One day Bob came to John with an ask, he would need to run a MySql database in the Kubernetes cluster hosting his application. After some research John is ready to share a small POC with Bob.

# Manually creating the persitant volumes


# Automatically creating the persitant volumes


![Storage Banner]({{ site.baseurl }}/assets/img/posts/2018-04-23-How-to-manage-stateful-applications-in-kubernetes/Storage-Banner.jpg)
<script async class="speakerdeck-embed" data-id="4c9e38395a8d45dcbb08dda95242c7fd" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>