## Key Concepts

### Monoliths, SOA, Microservices

From the perspective of service-oriented design, there are three ways of building a complex application. 

The first is to build a **monolith** application. A monolith application packs everything the application needs into one big file, which is then distributed to the end user. This is thew ay in which services were built in the "old days", and it did things like land man on the moon.

The second way is to build using services. Services are more loosely coupled pieces of software which perform specific pieces of user-given tasks. Services are great because they better fit the way that large software teams work, with each team becoming responsible for a "chunk" of the overall software. Services need to communicate between one another often, and use some kind of message bus to do so.

The final and most modern service architecture is with fully decoupled, containerized **microservices**. Microservices are runnable pieces of software which are individually responsible for very small slices of the overall application service. They use some sort of modern message protocols to communicate with one another very often.

Microservices originated from the UNIX command-line tool philosophy: do one thing, and do it well.

Microservices are the wave of the future because they make even the most complex projects much easier to divide and conquer: just split each task into a separate microservice, and work on it one piece (or multiple pieces) at a time. They're easier to handle, easier to test, easier to restart when a problem occurs, and due to their small size, intrinsically scalable.

### Containers, Docker

The most modern microservices are distributed as **containers**. Containers are lightweight virtual machines: chunks of compute which define an operating system (often a stripped down one, for efficiency's sake) and a sequence of steps performed by that operating system that, when completed, output a functioning microservice.

There are several competing containerization standards, but far and away the most successful and well-known one is Docker.

Small-enough applications may be built into a single container. It's also technically possible to build and serve applications based on multiple containers. However, in my opinion, the Docker native networking and volume management tooling leaves a lot to be desired.

I do not recommend doing anything complicated enough to require multiple containers using Docker alone. For larger-scale applications you should use some form of container orchestration.

If you're familiar with the packaging applications available to your programming language and environment of choice, containerization is easy. The hard part is figuring out how to scale your containerized applications. This problem is known as **container orchestration**.

### Container Orchestration, Kubernetes

There are several competing container orchestration systems. They all solve essentially the same problem, albeit in subtly different ways.

The first container orchestration challenge is resource management. For complex systems consisting of multiple containers, orchestrators are the software in charge of making sure that all of the containers are scheduled on hardware that can actually run them, have access to the disks and disk quotas that they require, and are otherwise healthy.

The second container orchestration challenge is being self-healing. Both the containers that the service is made up of and the hardware that these containers live on may fail, and a container orchestrator is in charge of rescheduling and restarting processes (potentially migrating off of bad hardware in the process) when they do.

The moment is Kubernetes. Kubernetes came about out of Google, based on their experience solving the isolated compute problem internally with Borg.

To achieve the self-healing property, Kubernetes maintains a highly available **master process**.

Every chunk of compute that is in the system runs overlapping chunks of the core Kubernetes processes (in a way, Kubernetes is itself microservice-based!). This means that if one or more nodes in Kubernetes fails, as long as at least one piece of compute in the system is still operating, Kubernetes will still be able to restart itself, then move on to restarting its containers.

### {Container, Resource} Administration

In container orchestration there exists a separation of concerns in resource management.

Kubernetes is designed to manage containers, and deploying projects using Kubernetes is all about shipping that stuff. This is **container administration**, and is the "fun part" of devops.

However, in order to have anything to ship containers on, Kubernetes needs to know about and have registered a variety of hardware resources known as **nodes**. Each node is a piece of compute; maybe a virtual machine, maybe a bare metal server, maybe even another container (yo dawg).

Provisioning these compute resources, and provisioning other specialized hardware like GPUs, TPUs, HDDs and SSDs, is not done by Kubernetes. All Kubernetes does is register these resources at start time. Provisioning these resources is a distinctly separate **resource administration** job.

This sort of administration was important in the days of eBay and other companies managing their own hardware. Now, with the cloud, you can get a megacorp to manage your hardware instances better than you ever could. 

While it's obviously possible to provision your own resources, Kubernetes is really, really meant to be used instead with these kinds of cloud services. And by and large, it is.

## Kubernetes

In the section above we covered two key Kubernetes concepts, nodes and the master service. In this section we'll dive deeper in and look at all the different core Kubernetes components from the perspective of our application.

Kubernetes ultimately manages containers, but it doesn't manage them directly. Instead of manages **pods**. A pod is a capsule containing one or more containers and a set of rules about what those containers should be provided with.
