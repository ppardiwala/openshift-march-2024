# Day 1

## Docker Overview
- comes in 2 flavors
  - Community Edition - Docker CE
  - Enterpise Edition - Docker EE
- written in Go language
- Developed and maintained by an organization called Docker Inc
- Client/Server architecture
- Client tool - docker
- Server tool - runs a service in Linux/Windows/Mac
- Registries
  - this is where all docker images are cached
- Docker Image
  - is a blueprint of Docker container
  - 
- docker container
  - it is an application process that runs in a separate unix namespace
  - though it looks like a virtual machine or a OS, it is technically just an application process
  - containers gets its own network stack, it own file system
  - containers also support package manager ( apt, yum, rpm, linux tools )
  - containers will never be able to replace an Operating System or Virtual machines
  - containers don't have their own OS Kernel
  - container represent a single application
  - containers running on the same system/os shares the Hardwares resources available on the OS they are running
  - containers will depend on the OS Kernel on which they are running

## Linux Kernel Container Features
- Namespace
  - allows to isolate one container from rest of the other containers
- Control Groups (CGroups)
  - let's you apply some resource quota restricts like
    - restrict how many cpu cores a particular container can utilize
    - restrict how much RAM a particular container at the max can utilize
    - restric how much storage a particular container at the max can utilize

## What is a Container Engine?
- a high-level software that depends on Container Runtime to manage container images and containers
- it offers user-friendly commands to manage images and containers but it depends on Container Runtimes to do that under the hood
- examples
  - Docker depends on containerd which internally uses runC container Runtime
  - Podman depends on CRI-O Container Runtime to manage images and containers

## What is a Container Runtime?
- low-level software that know how to manage container images and containers
- it is not so user-friendly, hence normally end-users like us wouldn't use container runtime directly
- end-user will be using Container Engine not the Container runtime
- Exmaples
  - runC
  - CRI-O
  - rkt (pronounced as Rocket)

## What is Container Orchestration Platform?
- manages the containerized application workloads
- has one or more servers ( cluster )
- most popular features that are supported by any Container Orchestration Platform
  - high availability to your applications that runs with Container Orchestration Platforms
  - it also supports in-built load-balancing, monitoring features to heal/repair your non-responding applications, etc.,
  - it also supports scaling up/down ( increase/reduce the no of application instances - depending on user traffic )
  - rolling update
    - you can upgrade/downgrade your already alive application from one version to other without any downtime
    - rolling back to older version of an application is also possible
- Examples
  - Docker SWARM
  - Kubernetes
  - Red Hat OpenShift
 
## What is Docker SWARM?
- it is an Orchestration platfrom from Docker Inc organization
- it is open-source
- it is not production-grade
- it is very light-weight setup, you could install this easily on a normal laptop/desktop
- generally used for learning purpose, Dev/QA setup
- never seen this in production
- it only supports managing Docker containerized applications

## What is Kubernetes?
- it is an Orchestration platfrom from Google
- it is open-source
- it is production-grade
- it can be used in Dev/QA/Production pretty much everyone
- as it is open-source we don't get any official support
- AWS offers managed EKS - Elastic Kubernetes Service, you can get support from Amazon
- Azure offers managed AKS - Azure Kubernetes Service, you can get support from Microsoft
- Kuberetes supports Operators and Custom Resource Definitions to extend the functionality of Kubernetes

## What is OpenShift?
- it is an Orchestration platfrom from Red Hat
- it is Red Hat's distribution of Kubernetes
- Openshift is developed on top of opensource Google Kubernetes with many additional features
- OpenShift using Kubernetes Operators( ie Custom Resources and Customer Operators ) has added many additional features on top of Kubernetes
- You will get support from Red Hat
- It is a licensed product
