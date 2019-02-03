# Introduction

This is the documentation pages for my awesome docker/ceph cluster :) 
After being burnt way too many times from hardware failures, operating system 
botched updates (and some times just plain boredom of maintaining services) I 
was looking for an easy way of creating/maintaining/extending/deleting services.

Docker is an impressive new technology. It will allow me to :

* to abstract my services from the hardware (so in case of a hardware
failure, I can just restart them on another computer without too much effort)
* since the services are not dependent on the hardware, hopefully they will be
much more reliable and much more scalable. 
* document through code the requirements of each service and not having to write
way too many ugly README files
* play with a new technology
* to have a safe sandbox to test new tools, without influencing the existing deployed services


## Who is this for?

For the strong of heart that are looking for new ideas. It is not meant as guide,
but more as a starting step. I will not be providing any support, but any bugs/questions you are welcome to create an issue at github (and if I have time I 
will answer them, but no promises). Think of it as a collection of guides for 
creating your own cluster and running your self-hosted services. I post them
so if I ever have to start again from the begging, I will not have to do it 
completely from start from the beggining.

For the moment I am not using kubernetes. I wanted something simplier, so I could 
start learning docker. I might consider kubernets but not for the moment. So if
you are looking for kubernetes you are in the wrong place. Here we only have 
the traditional swarm services :)

Another place that has some similar instructions is the 
[cookbook](https://geek-cookbook.funkypenguin.co.nz/)
from Funky Penguin. His recipes were the inspiration for my own recipes.