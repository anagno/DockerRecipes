# Introduction

As already mentioned, I choosed docker to create my own cluster for 
hosting my own services in my own hardware. 

Here I present my requirements and assumptions of my setup:

# Requirements

* Abstract the services from the hardware for high availability and 
portability (i.e. a particular service could be shutdown and moved to 
new hardware with minimal effort).
* Abstract the services from the Host Operating System. Any security issues,
in the operating system will not cause downtime in the services.
* Increase security by isolating the services from each other. 
A service that does not need to know the existinence of an another 
should not know it, and implementation details of a service should 
not leak to an another one.
* Minimum documentation of the deployment procedure. Everything should be
documented through code. 
* Consistent deployments. If our operating system changes completely, 
it should not influence our services (I have already been burned
from upgrades to libraries, that were irrelevant to my services).
* Encrypted access to the services using https
* Minimum coupling to IP address. The services should not rely 
on static ip, unless these implementation details are hidden internally
on the stack of each service.


# Assumptions

* My local network is safe. The services can communicate internally
without using encryption. 
* DNS, DHCP server will be always available. 
* We need an external storage service that will be always available 
to the docker nodes. In my case I am using [Ceph](https://ceph.com/).
So the first step will is to set up the pesistent storage for our docker
swarm. (For more details of the assumptions in the storage see 
[Mounting Ceph on docker nodes](persistent_storage/mounting_clients.md).
* We will create a virtual ip, from which our swarm will be available.
Only this IP has to be know to access the services. 