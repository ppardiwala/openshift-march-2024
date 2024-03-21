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


## Info - What is persistent volume in Kubernetes/OpenShift?
- Persistent Volume is an external storage
- This storage can be provisioned manually by System Administrators or automatically
- In case you prefer provisioning the external storage manually, then we need to choose where you are going to provision the external storage
  - NFS
  - AWS EBS
  - Azure Storage Service
  - Third party storage service
- In case you prefer provisioning the external storage automatically on demand, we need to create something called Storage Class in Kubernetes/OpenShift
- The Storage Class is a configuration/definition you perform that has credentials for AWS/Azure to create the persistent volume on demand
- Persistent Volume (PV) generally has the below options
  - Storage Class ( Optional )
  - Disk Size in MiB/GiB
  - Access Mode - ReadWriteOnce/ReadWriteMany
  
## Info - Persistent Volume Claim (PV)
- Any application(pod) that needs external storage will be requesting for external storage via Persistent Volume Claim
- Persistent Volume Claim(PVC)
  - Disk Size
  - Storage Class (Optional)
  - Access Mode
  - label selectors (optional)

 ## Info - Troubleshooting NFS mount issues

Reference - https://docs.openshift.com/container-platform/4.14/storage/persistent_storage/persistent-storage-nfs.html

On the NFS Server machine, open up nfs ports
```
firewall-cmd --permanent --add-service={nfs,rpc-bind,mountd}
firewall-cmd --permanent --add-port=2049/udp
firewall-cmd --permanent --add-port=2049/tcp
firewall-cmd --reload
```

Check the boolean status
```
[root@master-1 /]# getsebool -a | grep nfs
cobbler_use_nfs --> off
colord_use_nfs --> off
conman_use_nfs --> off
ftpd_use_nfs --> off
git_cgi_use_nfs --> off
git_system_use_nfs --> off
httpd_use_nfs --> off
ksmtuned_use_nfs --> off
logrotate_use_nfs --> off
mpd_use_nfs --> off
nagios_use_nfs --> off
nfs_export_all_ro --> on
nfs_export_all_rw --> on
nfsd_anon_write --> off
openshift_use_nfs --> off
polipo_use_nfs --> off
```

In case your Pods are unable to mount NFS shared folder, you need to enable nfs mounting within Openshift nodes as shown below

```
oc debug node/master-1.ocp4.training.tektutor
chroot /host /bin/bash
setsebool -P virt_use_nfs 1
setsebool -P openshift_use_nfs 1
setsebool -P mpd_use_nfs 1
setsebool -P nfsd_anon_write 1
exit

oc debug node/master-2.ocp4.training.tektutor
chroot /host /bin/bash
setsebool -P virt_use_nfs 1
setsebool -P openshift_use_nfs 1
setsebool -P mpd_use_nfs 1
setsebool -P nfsd_anon_write 1
exit

oc debug node/master-3.ocp4.training.tektutor
chroot /host /bin/bash
setsebool -P virt_use_nfs 1
setsebool -P openshift_use_nfs 1
setsebool -P mpd_use_nfs 1
setsebool -P nfsd_anon_write 1
exit

oc debug node/worker-1.ocp4.training.tektutor
chroot /host /bin/bash
setsebool -P virt_use_nfs 1
setsebool -P openshift_use_nfs 1
setsebool -P mpd_use_nfs 1
setsebool -P nfsd_anon_write 1
exit

oc debug node/worker-2.ocp4.training.tektutor
chroot /host /bin/bash
setsebool -P virt_use_nfs 1
setsebool -P openshift_use_nfs 1
setsebool -P mpd_use_nfs 1
setsebool -P nfsd_anon_write 1
exit
```

Expected output
<pre>
[jegan@tektutor.org wordpress-with-configmap-and-secrets]$ oc debug node/master-2.ocp4.training.tektutor
Temporary namespace openshift-debug-7zzkh is created for debugging node...
Starting pod/master-2ocp4trainingtektutor-debug-g8hj7 ...
To use host binaries, run `chroot /host`
chroot /host /bin/bash
Pod IP: 192.168.122.123
If you don't see a command prompt, try pressing enter.
sh-4.4# chroot /host /bin/bash
[root@master-2 /]# setsebool -P virt_use_nfs 1
[root@master-2 /]# exit
exit
sh-4.4# exit
exit

Removing debug pod ...
Temporary namespace openshift-debug-7zzkh was removed.
</pre>

Try to mount the NFS shared folder manually from one of the OpenShift nodes
```
mount -vvv -t nfs 192.168.1.127:/var/nfs/jegan/mariadb /tmp
umount -vvv -t nfs 192.168.1.127:/var/nfs/jegan/mariadb /tmp
```

## Info - In case you are curious to see the containers running inside openshift nodes
```
oc debug node/worker-1.ocp4.training.tektutor
chroot /host /bin/bash
podman version
podman info
crictl ps -a
```

Expected output
<pre>
[jegan@tektutor.org wordpress-with-configmap-and-secrets]$ oc debug node/worker-1.ocp4.training.tektutor
Temporary namespace openshift-debug-cpxmp is created for debugging node...
Starting pod/worker-1ocp4trainingtektutor-debug-gr4q9 ...
To use host binaries, run `chroot /host`
Pod IP: 192.168.122.206
If you don't see a command prompt, try pressing enter.
sh-4.4# chroot /host /bin/bash
  
[root@worker-1 /]# podman version
Client:       Podman Engine
Version:      4.4.1
API Version:  4.4.1
Go Version:   go1.20.12
Built:        Thu Feb  8 05:04:04 2024
OS/Arch:      linux/amd64
[root@worker-1 /]# podman info
host:
  arch: amd64
  buildahVersion: 1.29.0
  cgroupControllers:
  - cpuset
  - cpu
  - io
  - memory
  - hugetlb
  - pids
  - rdma
  - misc
  cgroupManager: systemd
  cgroupVersion: v2
  conmon:
    package: conmon-2.1.7-3.2.rhaos4.14.el9.x86_64
    path: /usr/bin/conmon
    version: 'conmon version 2.1.7, commit: 0cfc49682f4e9723c654c26ad6be1a3aacd7dcea'
  cpuUtilization:
    idlePercent: 99.12
    systemPercent: 0.29
    userPercent: 0.59
  cpus: 8
  distribution:
    distribution: '"rhcos"'
    variant: coreos
    version: "4.14"
  eventLogger: journald
  hostname: worker-1.ocp4.training.tektutor
  idMappings:
    gidmap: null
    uidmap: null
  kernel: 5.14.0-284.52.1.el9_2.x86_64
  linkmode: dynamic
  logDriver: journald
  memFree: 8187338752
  memTotal: 16375459840
  networkBackend: cni
  ociRuntime:
    name: crun
    package: crun-1.14-1.rhaos4.14.el9.x86_64
    path: /usr/bin/crun
    version: |-
      crun version 1.14
      commit: 667e6ebd4e2442d39512e63215e79d693d0780aa
      rundir: /run/crun
      spec: 1.0.0
      +SYSTEMD +SELINUX +APPARMOR +CAP +SECCOMP +EBPF +CRIU +YAJL
  os: linux
  remoteSocket:
    path: /run/podman/podman.sock
  security:
    apparmorEnabled: false
    capabilities: CAP_CHOWN,CAP_DAC_OVERRIDE,CAP_FOWNER,CAP_FSETID,CAP_KILL,CAP_NET_BIND_SERVICE,CAP_SETFCAP,CAP_SETGID,CAP_SETPCAP,CAP_SETUID
    rootless: false
    seccompEnabled: true
    seccompProfilePath: /usr/share/containers/seccomp.json
    selinuxEnabled: true
  serviceIsRemote: false
  slirp4netns:
    executable: /usr/bin/slirp4netns
    package: slirp4netns-1.2.0-3.el9.x86_64
    version: |-
      slirp4netns version 1.2.0
      commit: 656041d45cfca7a4176f6b7eed9e4fe6c11e8383
      libslirp: 4.4.0
      SLIRP_CONFIG_VERSION_MAX: 3
      libseccomp: 2.5.2
  swapFree: 0
  swapTotal: 0
  uptime: 2h 27m 29.00s (Approximately 0.08 days)
plugins:
  authorization: null
  log:
  - k8s-file
  - none
  - passthrough
  - journald
  network:
  - bridge
  - macvlan
  - ipvlan
  volume:
  - local
registries:
  search:
  - registry.access.redhat.com
  - docker.io
store:
  configFile: /etc/containers/storage.conf
  containerStore:
    number: 1
    paused: 0
    running: 0
    stopped: 1
  graphDriverName: overlay
  graphOptions:
    overlay.skip_mount_home: "true"
  graphRoot: /var/lib/containers/storage
  graphRootAllocated: 53082042368
  graphRootUsed: 10045403136
  graphStatus:
    Backing Filesystem: xfs
    Native Overlay Diff: "true"
    Supports d_type: "true"
    Using metacopy: "false"
  imageCopyTmpDir: /var/tmp
  imageStore:
    number: 23
  runRoot: /run/containers/storage
  transientStore: false
  volumePath: /var/lib/containers/storage/volumes
version:
  APIVersion: 4.4.1
  Built: 1707368644
  BuiltTime: Thu Feb  8 05:04:04 2024
  GitCommit: ""
  GoVersion: go1.20.12
  Os: linux
  OsArch: linux/amd64
  Version: 4.4.1

[root@worker-1 /]# crictl ps -a
CONTAINER           IMAGE                                                                                                                    CREATED             STATE               NAME                                 ATTEMPT             POD ID              POD
ecc1a7baeadee       0e06c0f2c42bd0c734b0f48d16df04e7b97128f4426abe2d7089b5bac0d578a4                                                         11 minutes ago      Running             container-00                         0                   8bf530cdf3a93       worker-1ocp4trainingtektutor-debug-gr4q9
cbdf81deaa10b       cd08464f8e6e5145736152e423985a40b51dad242b9f407192fe76ff64d737a6                                                         23 minutes ago      Running             wordpress                            0                   62e243d3ab13d       wordpress-658fc8c5b9-2mqwr
9442b68ada46e       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:8b27c7da20a626e7ff25028beeb1022b7def370f67b7a8fcd384a7645a94706e   2 hours ago         Running             network-check-target-container       0                   b72264e779c39       network-check-target-tljqs
deba86b5d8145       b80ae5477b314111e459b66db252527ca8fc090d98b99fe1465c3a3f439c942d                                                         2 hours ago         Running             kube-rbac-proxy                      0                   d3c28cff27228       network-metrics-daemon-2g8dp
8af14733dc02a       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:693fef5f95dd1facfdcb6df3514a001a425a6939676a53406734c3e49ebdbecb   2 hours ago         Running             network-metrics-daemon               0                   d3c28cff27228       network-metrics-daemon-2g8dp
dd0aed94ef61b       8cd3b7da296077275c861caf6b36866bfbc99a55c6f4b8bbac8ac46585a6582e                                                         2 hours ago         Running             kube-multus-additional-cni-plugins   0                   dcf97b2e6093f       multus-additional-cni-plugins-dkkrd
cdd218c096a82       73045715f6c5d4a8e20a069621417bcb14cb6487579e2d9017b724037a3a33cd                                                         2 hours ago         Exited              whereabouts-cni                      0                   dcf97b2e6093f       multus-additional-cni-plugins-dkkrd
7cee0a34831b3       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:0ca8943e16c7d52f1be259061170c7f666c431d1d34ccbd64aea38d4735de45c   2 hours ago         Exited              whereabouts-cni-bincopy              0                   dcf97b2e6093f       multus-additional-cni-plugins-dkkrd
407fcd19d8bfc       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:930b5bd27584df4fb521c5ec0377603e51f1d633a3f99dc3d3a833fd600fc35a   2 hours ago         Running             serve-healthcheck-canary             0                   03477eb55f75f       ingress-canary-qmlkc
e91fc52fa7512       b80ae5477b314111e459b66db252527ca8fc090d98b99fe1465c3a3f439c942d                                                         2 hours ago         Running             kube-rbac-proxy                      0                   fd0cce7c593ea       dns-default-9b54c
db8e84f475674       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:6b38d75b297fa52d1ba29af0715cec2430cd5fda1a608ed0841a09c55c292fb3   2 hours ago         Running             dns                                  0                   fd0cce7c593ea       dns-default-9b54c
f6f95c09fc18b       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:198f5a31c424606b1ea67581b132a51efe71c416c56ec7652f8fd445af44773f   2 hours ago         Exited              routeoverride-cni                    0                   dcf97b2e6093f       multus-additional-cni-plugins-dkkrd
d0ce549b8929e       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:7afae9702bc149efe67fb2090704b0d57998b7f656c11d93a45f9a11a235c4b3   2 hours ago         Exited              bond-cni-plugin                      0                   dcf97b2e6093f       multus-additional-cni-plugins-dkkrd
f02cfa9127ae4       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:38ff2b79106b412567008136569e1fe9eccab3b6eae7b19d8c47c9bc2cf1f99a   2 hours ago         Exited              cni-plugins                          0                   dcf97b2e6093f       multus-additional-cni-plugins-dkkrd
51797c9b5c19e       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:27fb71a6cbae202f99d429f625a10a1bb30ebd3d79e7379297bd01d4bda25e75   2 hours ago         Running             kube-multus                          0                   a23ee6af46612       multus-9kpbs
8a039f96cd110       b80ae5477b314111e459b66db252527ca8fc090d98b99fe1465c3a3f439c942d                                                         2 hours ago         Running             kube-rbac-proxy                      0                   4ec7ee120c29e       sdn-df86w
0cf91799745bb       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:28a764758233131ef71261efc1d9aa1050e5aa28d348a0f421e013d1ffb3fdc5   2 hours ago         Running             tuned                                0                   6f62d4d97093b       tuned-h4b8v
e1b4843e90128       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:ac59d66362f1cebe713714e3ffff83a1d13b63997a6f29c921d02153e3318dc3   2 hours ago         Running             sdn                                  0                   4ec7ee120c29e       sdn-df86w
625727614d826       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:d7a26cf7bc38061639ee3b8515bc2a8cb8c64f5ebbb70250dd43e633222cb19c   2 hours ago         Running             dns-node-resolver                    0                   ee4158e4ee23b       node-resolver-7kdbr
7421367a30a27       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:0437b6e70a2364cca080fe64e5b481a1f397b433f6e188423c9e47512120c178   2 hours ago         Running             node-ca                              0                   6a6fe64808783       node-ca-5btz8
6ec9a9a4205a3       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:bae1c43ad895bf85ea535042f71842b60148a25a0e356a081913bf67c83f5855   2 hours ago         Exited              egress-router-binary-copy            0                   dcf97b2e6093f       multus-additional-cni-plugins-dkkrd
35d30a270c1c5       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:b3cc4a7529a6c70b983803c59ee744c5a0a989ca6f8c6eab15d041d96399df88   2 hours ago         Running             kube-rbac-proxy                      0                   3ad9366847070       machine-config-daemon-nv8cs
737329b3e0875       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:b3cc4a7529a6c70b983803c59ee744c5a0a989ca6f8c6eab15d041d96399df88   2 hours ago         Running             kube-rbac-proxy                      0                   0e644968e91ec       node-exporter-bs6bl
028b82500e031       65d35907a90182d763291f5acdb1fa709b61082dba129dcd73b409a94f9d1df5                                                         2 hours ago         Running             node-exporter                        0                   0e644968e91ec       node-exporter-bs6bl
017a754b11670       quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:33835833fc124d5220af83ef4b85338c2bee24daee48eb0f775dfb39a742acd6   2 hours ago         Exited              init-textfile                        0                   0e644968e91ec       node-exporter-bs6bl
1d649dbb6828c       51b5d7bfff504a6a1eb57616ae304482352cce4c105b4234b4816215534cd4ef                                                         2 hours ago         Running             machine-config-daemon                0                   3ad9366847070       machine-config-daemon-nv8cs
[root@worker-1 /]# crictl ps -a | grep wordpress
cbdf81deaa10b       cd08464f8e6e5145736152e423985a40b51dad242b9f407192fe76ff64d737a6                                                         23 minutes ago      Running             wordpress                            0                   62e243d3ab13d       wordpress-658fc8c5b9-2mqwr  
</pre>


## Lab - Deploying wordpress and mysql multi-pod application that uses Persistent volumes
```
cd ~/openshift-march-2024
git pull
cd Day2/wordpress

oc apply -f mariadb-pv.yml
oc apply -f mariadb-pvc.yml
oc apply -f mariadb-deploy.yml
oc apply -f mariadb-svc.yml

oc apply -f wordpress-pv.yml
oc apply -f wordpress-pvc.yml
oc apply -f wordpress-deploy.yml
oc apply -f wordpress-route.yml
```
