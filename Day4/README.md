# Day 4

## Reference
Red Hat Openshift also supports adding Windows Virtual Machines (nodes) into the Openshift cluster
https://docs.openshift.com/container-platform/4.8/windows_containers/understanding-windows-container-workloads.html#understanding-windows-container-workloads

## Lab - Creating an NFS Storage Class 

```
cd ~/openshift-march-2024
git pull
cd Day4/nfs-storage-class

./deploy.sh
```

## Lab - Scheduling pods matching a specific node criteria
There are 2 types of Node Affinity possible
1. Preferred During Scheduling Ignored During Execution
2. Required During Scheduling Ignored During Execution

Preferred 
- If the scheduler will try to find a Node that matches the preferred criteria, if it find then it would deploy the Pod onto the node that matches the preferred condition
- If the scheduler is not able to find a Node that maches the preferred condition, then it will still deploy onto some node even though that doesn't meet that criteria
- Use cases
  - if the Pod prefers to run on a node that has SSD disk
  - if a .Net application prefers to run on a Windows Compute Node in OpenShift cluster

First if you can if there are any nodes that has disk=ssd label
```
oc get nodes -l disk=ssd
```

Assuming, there are no nodes that has the disk=ssd label, let's proceed.

```
cd ~/openshift-march-2024
git pull

cd Day4/scheduling
oc apply -f preferred-affinity.yml
```

Let's check on which the pod is scheduled
```
oc get po -o wide
```

Let's now apply disk=ssd label to worker 2 node
```
oc label node worker-2.ocp.tektutor.org.labs disk=ssd
```

If you notice, once the pod is scheduled even if the scheduler find a node that maches the node affinity condition put forth by the pod is available, the Pod won't be moved to the node that matches the criteria once it is already scheduled to some node in the Openshift cluster.

```
oc get po -o wide
```

Now, let's delete the pod
```
cd ~/openshift-march-2024

cd Day4/scheduling
oc delete -f preferred-affinity.yml
```

Now, let's remove the label from worker-2.ocp.tektutor.org.labs node
```
oc label worker-2.ocp.tektutor.org.labs disk-
```
Expected output
<pre>
[jegan@tektutor.org scheduling]$ oc get nodes -l disk=ssd
No resources found
  
[jegan@tektutor.org scheduling]$ oc label node worker-2.ocp.tektutor.org.labs disk=ssd
node/worker-2.ocp.tektutor.org.labs labeled
  
[jegan@tektutor.org scheduling]$ oc get nodes -l disk=ssd
NAME                             STATUS   ROLES    AGE     VERSION
worker-2.ocp.tektutor.org.labs   Ready    worker   7h25m   v1.27.6+f67aeb3
  
[jegan@tektutor.org scheduling]$ oc label node worker-2.ocp.tektutor.org.labs disk-
node/worker-2.ocp.tektutor.org.labs unlabeled
  
[jegan@tektutor.org scheduling]$ oc get nodes -l disk=ssd
No resources found  
</pre>


Let's deploy the pod with required node affinity condition
```
cd ~/openshift-march-2024

cd Day4/scheduling
oc apply -f required-affinity.yml
```

Let's check if the pod status
```
oc get po -o wide
```

Since, the scheduler is not able to find a node that has disk=ssd label, the Pod will remain the Pending status.

Now, let's apply the disk=ssd label to worker2
```
oc label node worker-2.ocp.tektutor.org.labs disk=ssd
```

Now, let's list all the nodes that has label disk=ssd
```
oc get nodes -l disk=ssd
```

Now, we expect the pod to be deployed onto the worker2 as it meets the required criteria
```
oc get po -o wide
```

Finally, let's clean up the pod
```
oc delete -f required-affinity.yml
oc label node worker-2.ocp.tektutor.org.labs disk-
```

## Lab - Deploy custom spring-boot application from source code into OpenShift cluster
```
https://github.com/tektutor/spring-ms
```

## Lab - Finding the OpenShift Private Image Registry 
```
oc get po --all-namespaces | grep registry

oc get svc -n openshift-image-registry
```

Expected output
<pre>
[jegan@tektutor.org spring-ms]$ oc get po --all-namespaces | grep registry
openshift-image-registry                           cluster-image-registry-operator-7b4979cf68-v7kqv                1/1     Running     0             11h
openshift-image-registry                           image-pruner-28516320-8t9mt                                     0/1     Completed   0             10h
openshift-image-registry                           image-registry-75d695567f-ghvfc                                 1/1     Running     0             11h
openshift-image-registry                           node-ca-6s95s                                                   1/1     Running     0             11h
openshift-image-registry                           node-ca-gn9lt                                                   1/1     Running     0             11h
openshift-image-registry                           node-ca-gw454                                                   1/1     Running     0             11h
openshift-image-registry                           node-ca-jtxp5                                                   1/1     Running     0             11h
openshift-image-registry                           node-ca-tg4bx                                                   1/1     Running     0             11h
  
[jegan@tektutor.org spring-ms]$ oc get svc -n openshift-image-registry
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
image-registry            ClusterIP   172.30.89.109   <none>        5000/TCP    11h
image-registry-operator   ClusterIP   None            <none>        60000/TCP   11h
[jegan@tektutor.org spring-ms]$ 
[jegan@tektutor.org spring-ms]$ oc describe svc/image-registry -n openshift-image-registry
Name:              image-registry
Namespace:         openshift-image-registry
Labels:            docker-registry=default
Annotations:       imageregistry.operator.openshift.io/checksum: sha256:1c19715a76014ae1d56140d6390a08f14f453c1a59dc36c15718f40c638ef63d
                   service.alpha.openshift.io/serving-cert-secret-name: image-registry-tls
                   service.alpha.openshift.io/serving-cert-signed-by: openshift-service-serving-signer@1710977180
                   service.beta.openshift.io/serving-cert-signed-by: openshift-service-serving-signer@1710977180
Selector:          docker-registry=default
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                172.30.89.109
IPs:               172.30.89.109
Port:              5000-tcp  5000/TCP
TargetPort:        5000/TCP
Endpoints:         10.128.0.68:5000
Session Affinity:  None
Events:            <none>
</pre>

## Lab - Deploying an application from CLI using source strategy
```
oc new-app registry.access.redhat.com/ubi8/openjdk-11~https://github.com/tektutor/spring-ms.git --strategy=source
```
Expected output
<pre>
[jegan@tektutor.org spring-ms]$ oc new-app registry.access.redhat.com/ubi8/openjdk-11~https://github.com/tektutor/spring-ms.git --strategy=source
--> Found container image a6b53e1 (4 weeks old) from registry.access.redhat.com for "registry.access.redhat.com/ubi8/openjdk-11"

    Java Applications 
    ----------------- 
    Platform for building and running plain Java applications (fat-jar and flat classpath)

    Tags: builder, java

    * An image stream tag will be created as "openjdk-11:latest" that will track the source image
    * A source build using source code from https://github.com/tektutor/spring-ms.git will be created
      * The resulting image will be pushed to image stream tag "spring-ms:latest"
      * Every time "openjdk-11:latest" changes a new build will be triggered

--> Creating resources ...
    imagestream.image.openshift.io "openjdk-11" created
    imagestream.image.openshift.io "spring-ms" created
    buildconfig.build.openshift.io "spring-ms" created
    deployment.apps "spring-ms" created
    service "spring-ms" created
--> Success
    Build scheduled, use 'oc logs -f buildconfig/spring-ms' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/spring-ms' 
    Run 'oc status' to view your app.  
</pre>


You may check the logs as shown below
```
oc logs -f bc/spring-ms
```

You need to create a route
```
oc get svc
oc expose svc/spring-ms
oc get route
```

Now from the webconsole --> Developer context -> topology you can click on the route link(arrow point upward) to access the route public url to see the web page.
