# Docker, what the hell is that?

##### *18 Decembre 2016*

![Docker Banner](/2016/assets/images/2016-12-18-Docker-what-the-hell-is-that/Docker-Banner.png)

What is Docker, this not so old technology that everybody is talking about but that nobody seems to really understand?

First of all, Docker is 2 things:

- it is a software container platform (Docker Engine) which is open-source and is under Apache 2.0 licences, so free to use and redistribute
- it is also a company, Docker Inc, which is the creator of the Docker Engine open-source project

Here, when we will speak about Docker, we will only speak about the open-source project, the Docker Engine.

---

## So what exactly is Docker

Here, thanks to Docker website, we know that:

> Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries - anything that can be installed on a server.

At this point you may be thinking “So, what a big deal, Docker is a just another VM type” and you could not be more wrong for multiples reasons, the first one being that Docker is a container format and containers have some significant differences with VMs.

## So what are the differences between Containers and VMs

It is mostly the architectural approach which differentiate Containers from VMs.

The VMs are emulating a real machine (with virtual hardware) where the containers are directly sharing the kernel and ressources of the host.

## The Virtual Machines way of doing things

On top of the host operating system seats the hypervisor which emulates some virtual hardware, using this virtual hardware, you have to install a full guest OS, above which you will add the perquisite softwares for the app you are willing to run and then, finally there is the app.

![VM Architecture Schema](/2016/assets/images/2016-12-18-Docker-what-the-hell-is-that/VM-Schema.png)

For me, there are many issues when you proceed like this:

1. It is not optimized at all from a hardware perspective, the virtualized operating system is using CPU and RAM and usually you are over allocating the disk space allowed for each VMs.

2. When you have to spin up a new instance, it’s really slow, it usually includes a lot of manual steps and you have to manage VMs operating system lifecycle.

## The Docker way of doing things

If we have a look at the graph, we see the host operating system, on top of which we have the Docker Engine, above which seats directly the software prerequisites and the App.

![Docker Architecture Schema](/2016/assets/images/2016-12-18-Docker-what-the-hell-is-that/Docker-Schema.png)

It seems simpler, isn’t it?
No guest OS and the Docker Engine instead of the Hypervisor.

How is that possible ?

To make it short, the Docker Engine is using Cgroups, Kernel Namespaces and SE Linux policies to isolate your app from the rest of the operating system.

But what is important here is that Docker allows us to run our apps with less overhead than VMs does and that is what Docker is. Docker is a piece of software that allow us to run efficiently our app, that’s all.

## To conclude

Why should we care about Docker?

In this world of service oriented architecture, where more and more code has to be run, where more and more developers have to collaborate, it is more and more important to have a better usage of our ressources and, for me, Docker is a move in the right direction.