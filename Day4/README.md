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
