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
