# Day 2

## Lab - Deploying nginx in declarative style using yaml file
```
oc create deploy nginx --image=bitnami/nginx --dry-run=client -o yaml
oc create deploy nginx --image=bitnami/nginx --dry-run=client -o yaml > nginx-deploy.yml
oc apply -f nginx-deploy.yml
oc get deploy,rs,po
```

Expected output
<pre>
[jegan@tektutor.org declarative-scripts]$ oc create deploy nginx --image=bitnami/nginx --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: bitnami/nginx
        name: nginx
        resources: {}
status: {}
[jegan@tektutor.org declarative-scripts]$ oc create deploy nginx --image=bitnami/nginx --dry-run=client -o yaml > nginx-deploy.yml

[jegan@tektutor.org declarative-scripts]$ oc apply -f nginx-deploy.yml 
deployment.apps/nginx created
  
[jegan@tektutor.org declarative-scripts]$ oc get deploy,rs,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   0/1     1            0           7s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-58bfb7c6b   1         1         0       7s

NAME                        READY   STATUS              RESTARTS   AGE
pod/nginx-58bfb7c6b-bmbc2   0/1     ContainerCreating   0          7s
[jegan@tektutor.org declarative-scripts]$ oc get po -w
NAME                    READY   STATUS    RESTARTS   AGE
nginx-58bfb7c6b-bmbc2   1/1     Running   0          13s  
</pre>

## Lab - Performing scale up in nginx deployment using declarative style
You need to edit the nginx-deploy.yml and update the replicas value from 1 to 3, save it and apply as shown below
```
oc apply -f nginx-deploy.yml
oc get po
```

Expected output
<pre>
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  <b>replicas: 3</b>
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: bitnami/nginx
        name: nginx
        resources: {}
status: {}
[jegan@tektutor.org declarative-scripts]$ vim nginx-deploy.yml 
  
[jegan@tektutor.org declarative-scripts]$ oc apply -f nginx-deploy.yml 
deployment.apps/nginx configured
  
[jegan@tektutor.org declarative-scripts]$ oc get po -w
NAME                    READY   STATUS              RESTARTS   AGE
nginx-58bfb7c6b-7nvgh   0/1     ContainerCreating   0          3s
nginx-58bfb7c6b-bmbc2   1/1     Running             0          5m40s
nginx-58bfb7c6b-l6zlp   0/1     ContainerCreating   0          3s
nginx-58bfb7c6b-l6zlp   1/1     Running             0          5s
nginx-58bfb7c6b-7nvgh   1/1     Running             0          6s  
</pre>

## Lab - Deleting a deployment in declarative style
```
cd ~/openshift-march-2024
git pull
cd Day3/declarative-scripts
oc delete -f nginx-deploy.yml
```

Expected output
<pre>
[jegan@tektutor.org declarative-scripts]$ ls
nginx-deploy.yml
  
[jegan@tektutor.org declarative-scripts]$ oc delete -f nginx-deploy.yml 
deployment.apps "nginx" deleted
  
[jegan@tektutor.org declarative-scripts]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
No resources found in jegan namespace.  
</pre>

## Lab - Creating a pod in declarative style
```
oc run nginx-pod --image=bitnami/nginx:latest --dry-run=client -o yaml
oc run nginx-pod --image=bitnami/nginx:latest --dry-run=client -o yaml > nginx-pod.yml
oc apply -f nginx-pod.yml
oc get po
oc delete -f nginx-pod.yml
```

Expected output
<pre>
[jegan@tektutor.org declarative-scripts]$ ls
nginx-deploy.yml  nginx-rs.yml
[jegan@tektutor.org declarative-scripts]$ oc run nginx-pod --image=bitnami/nginx:latest --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
spec:
  containers:
  - image: bitnami/nginx:latest
    name: nginx-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
[jegan@tektutor.org declarative-scripts]$ oc run nginx-pod --image=bitnami/nginx:latest --dry-run=client -o yaml > nginx-pod.yml
  
[jegan@tektutor.org declarative-scripts]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
No resources found in jegan namespace.
  
[jegan@tektutor.org declarative-scripts]$ oc apply -f nginx-pod.yml 
Warning: would violate PodSecurity "restricted:v1.24": allowPrivilegeEscalation != false (container "nginx-pod" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx-pod" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx-pod" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx-pod" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
pod/nginx-pod created
  
[jegan@tektutor.org declarative-scripts]$ oc get po
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          6s
  
[jegan@tektutor.org declarative-scripts]$ oc delete -f nginx-pod.yml 
pod "nginx-pod" deleted  
</pre>
