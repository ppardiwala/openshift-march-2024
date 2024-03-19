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

## Lab - Creating a ClusterIP Internal Service in declarative style
```
cd ~/openshift-march-2024
git pull
cd Day2/declarative-scripts
oc apply -f nginx-deploy.yml
oc expose deploy/nginx --type=ClusterIP --port=8080 --dry-run=client -o yaml
oc expose deploy/nginx --type=ClusterIP --port=8080 --dry-run=client -o yaml > nginx-clusterip-svc.yml
oc apply -f nginx-clusterip-svc.yml
oc get svc
oc delete -f nginx-svc.yml

oc apply -f nginx-nodeport-svc.yml
oc get svc
oc delete -f nginx-nodeport-svc.yml 

oc apply -f nginx-lb-svc.yml
oc get svc
oc delete -f nginx-lb-svc.yml 
```

Expected output
<pre>
[jegan@tektutor.org declarative-scripts]$ ls
nginx-deploy.yml  nginx-pod.yml  nginx-rs.yml
  
[jegan@tektutor.org declarative-scripts]$ oc apply -f nginx-deploy.yml 
deployment.apps/nginx created
  
[jegan@tektutor.org declarative-scripts]$ oc get po
NAME                    READY   STATUS              RESTARTS   AGE
nginx-58bfb7c6b-n9z4l   0/1     ContainerCreating   0          4s
nginx-58bfb7c6b-qn6dh   0/1     ContainerCreating   0          4s
nginx-58bfb7c6b-tnkth   0/1     ContainerCreating   0          4s
  
[jegan@tektutor.org declarative-scripts]$ oc expose deploy/nginx --type=ClusterIP --port=8080 --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: nginx
  type: ClusterIP
status:
  loadBalancer: {}
  
[jegan@tektutor.org declarative-scripts]$ oc expose deploy/nginx --type=ClusterIP --port=8080 --dry-run=client -o yaml > nginx-clusterip-svc.yml
  
[jegan@tektutor.org declarative-scripts]$ oc apply -f nginx-clusterip-svc.yml 
service/nginx created
  
[jegan@tektutor.org declarative-scripts]$ oc get svc
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
nginx   ClusterIP   172.30.79.19   <none>        8080/TCP   2s
  
[jegan@tektutor.org declarative-scripts]$ oc delete -f nginx-clusterip-svc.yml 
service "nginx" deleted
  
[jegan@tektutor.org declarative-scripts]$ oc get svc
No resources found in jegan namespace.
</pre>


## Lab - Creating NodePort and LoadBalancer services in declarative style
```
oc get all
oc expose deploy/nginx --type=NodePort --port=8080 --dry-run=client -o yaml
oc expose deploy/nginx --type=NodePort --port=8080 --dry-run=client -o yaml > nginx-nodeport-svc.yml
oc expose deploy/nginx --type=LoadBalancer --port=8080 --dry-run=client -o yaml > nginx-lb-svc.yml

```


Expected output
<pre>
[jegan@tektutor.org declarative-scripts]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-58bfb7c6b-n9z4l   1/1     Running   0          8m44s
pod/nginx-58bfb7c6b-qn6dh   1/1     Running   0          8m44s
pod/nginx-58bfb7c6b-tnkth   1/1     Running   0          8m44s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           8m44s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-58bfb7c6b   3         3         3       8m44s
  
[jegan@tektutor.org declarative-scripts]$ oc expose deploy/nginx --type=NodePort --port=8080 --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: nginx
  type: NodePort
status:
  loadBalancer: {}
  
[jegan@tektutor.org declarative-scripts]$ oc expose deploy/nginx --type=NodePort --port=8080 --dry-run=client -o yaml > nginx-nodeport-svc.yml
  
[jegan@tektutor.org declarative-scripts]$ oc expose deploy/nginx --type=LoadBalancer --port=8080 --dry-run=client -o yaml > nginx-lb-svc.yml
  
[jegan@tektutor.org declarative-scripts]$ ls
nginx-clusterip-svc.yml  nginx-deploy.yml  nginx-lb-svc.yml  nginx-nodeport-svc.yml  nginx-pod.yml  nginx-rs.yml
  
[jegan@tektutor.org declarative-scripts]$ oc apply -f nginx-nodeport-svc.yml 
service/nginx created
  
[jegan@tektutor.org declarative-scripts]$ oc get svc
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx   NodePort   172.30.232.98   <none>        8080:31812/TCP   3s
  
[jegan@tektutor.org declarative-scripts]$ oc delete -f nginx-nodeport-svc.yml 
service "nginx" deleted
  
[jegan@tektutor.org declarative-scripts]$ oc apply -f nginx-lb-svc.yml 
service/nginx created
  
[jegan@tektutor.org declarative-scripts]$ oc get svc
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)          AGE
nginx   LoadBalancer   172.30.217.133   192.168.122.90   8080:32673/TCP   4s
  
[jegan@tektutor.org declarative-scripts]$ oc delete -f nginx-lb-svc.yml 
service "nginx" deleted  
</pre>

## Lab - Rolling update
```

```

Expected output
<pre>

</pre>

## Lab - Creating a mysql deployment in declarative style
Create a file named mysql-deploy.yml with the below content
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - env:
        - name: MARIADB_ROOT_PASSWORD
          value: root@123
        image: bitnami/mariadb:latest
        imagePullPolicy: Always
        name: mysql
        ports:
        - containerPort: 3306
          protocol: TCP
```

Create the mysql deployment as shown below
```
oc apply -f mysql-deploy.yml
oc get deploy,rs,po
```

Expected output
<pre>
[jegan@tektutor.org declarative-scripts]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
No resources found in jegan namespace.
[jegan@tektutor.org declarative-scripts]$ oc apply -f mysql-deploy.yml 
deployment.apps/mysql created
[jegan@tektutor.org declarative-scripts]$ oc get deploy,rs,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql   0/1     1            0           6s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-8689bc77f7   1         1         0       6s

NAME                         READY   STATUS              RESTARTS   AGE
pod/mysql-8689bc77f7-mt85r   0/1     ContainerCreating   0          6s
[jegan@tektutor.org declarative-scripts]$ oc get po -w
NAME                     READY   STATUS    RESTARTS   AGE
mysql-8689bc77f7-mt85r   1/1     Running   0          12s
^C[jegan@tektutor.org declarative-scripts]$ oc rsh pod/mysql-8689bc77f7-mt85r
$ mysql -u root -p
mysql: Deprecated program name. It will be removed in a future release, use '/opt/bitnami/mariadb/bin/mariadb' instead
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 11.2.3-MariaDB Source distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
5 rows in set (0.001 sec)

MariaDB [(none)]> CREATE DATABASE tektutor;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> 

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| tektutor           |
| test               |
+--------------------+
6 rows in set (0.000 sec)

MariaDB [(none)]> USE tektutor;
Database changed
MariaDB [tektutor]> SHOW TABLES;
Empty set (0.000 sec)

MariaDB [tektutor]> CREATE TABLE training ( id INT NOT NULL, name VARCHAR(250) NOT NULL, duration VARCHAR(250) NOT NULL, PRIMARY KEY(id) );
Query OK, 0 rows affected (0.002 sec)

MariaDB [tektutor]> DESCRIBE training;
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| id       | int(11)      | NO   | PRI | NULL    |       |
| name     | varchar(250) | NO   |     | NULL    |       |
| duration | varchar(250) | NO   |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
3 rows in set (0.001 sec)

MariaDB [tektutor]> INSERT INTO training VALUES ( 1, "Advanced OpenShift", "5 Days" );
Query OK, 1 row affected (0.001 sec)

MariaDB [tektutor]> INSERT INTO training VALUES ( 2, "Microservices with Go lang", "5 Days" );
Query OK, 1 row affected (0.001 sec)

MariaDB [tektutor]> INSERT INTO training VALUES ( 3, "Apache Kafka", "5 Days" );
Query OK, 1 row affected (0.000 sec)

MariaDB [tektutor]> SELECT * FROM training;
+----+----------------------------+----------+
| id | name                       | duration |
+----+----------------------------+----------+
|  1 | Advanced OpenShift         | 5 Days   |
|  2 | Microservices with Go lang | 5 Days   |
|  3 | Apache Kafka               | 5 Days   |
+----+----------------------------+----------+
3 rows in set (0.000 sec)

MariaDB [tektutor]> exit
Bye
$ exit
[jegan@tektutor.org declarative-scripts]$   
</pre>
