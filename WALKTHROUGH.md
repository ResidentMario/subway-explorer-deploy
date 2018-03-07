## Part 1: Containerization

### Monoliths, SOA, Microservices

From the perspective of service-oriented design, there are three ways of building a complex application. 

The first is to build a monolith application. A monolith application packs everything the application needs into one big file, which is then distributed to the end user. This is thew ay in which services were built in the "old days", and it did things like land man on the moon.

The second way is to build using services. Services are more loosely coupled pieces of software which perform specific pieces of user-given tasks. Services are great because they better fit the way that large software teams work, with each team becoming responsible for a "chunk" of the overall software. Services need to communicate between one another often, and use some kind of message bus to do so.

The final and most modern service architecture is with fully decoupled, containerized microservices. Microservices are runnable pieces of software which are individually responsible for very small slices of the overall application service. They use some sort of modern message protocols to communicate with one another very often.

Microservices originated from the UNIX command-line tool philosophy: do one thing, and do it well.

Microservices are the wave of the future because they make even the most complex projects much easier to divide and conquer: just split each task into a separate microservice, and work on it one piece (or multiple pieces) at a time. They're easier to handle, easier to test, and due to their small size, intrinsically scalable.

### Containers, Docker

The most modern microservices are distributed as containers. Containers are lightweight virtual machines: chunks of compute which define an operating system (often a stripped down one, for efficiency's sake) and a sequence of steps performed by that operating system that, when completed, output a functioning microservice.

There are several competing containerization standards, but far and away the most successful and well-known one is Docker.

Small-enough applications may be built into a single container. It's also technically possible to build and serve applications based on multiple containers. However, in my opinion, the Docker native networking and volume management tooling leaves a lot to be desired.

I do not recommend doing anything complicated enough to require multiple containers using Docker alone. For larger-scale applications you should use some form of container orchestration.

If you're familiar with the packaging applications available to your programming language and environment of choice, containerization is easy. The hard part is figuring out how to scale your containerized applications. This problem is known as containerization orchestration.

Hence I omit saying anything else about containers because they are so easy to define. This write-up is practice explaining the concepts to the harder part, container orchestration...

TODO: Come back to this later.

### Container Orchestration, Kubernetes

There are several competing container orchestration schemes, but the runaway market leader at the moment is Kubernetes. Kubernetes came about out of Google, based on their experience solving the differenciable compute problem internally with Borg.

