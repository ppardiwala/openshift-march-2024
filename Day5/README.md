# Day 5

## References
About Parent Devfile
https://registry.devfile.io/viewer/devfiles/community/java-springboot?devfile-version=1.2.0

## Lab - Setup your own JFrog Artifactory Enterprise server 

On your Web browser, navigate to the below url
<pre>
https://jfrog.com/artifactory/enterprise-getting-started/
</pre>

Expected output
![JFrog](jfrog-1.png)
Click on "Start Free" button in the top right corner of the web page to get the below page
![JFrog](jfrog-3.png)
Click on Google to get the below page
![JFrog](jfrog-2.png)
Click on "Cloud Trial" and go with aws public cloud
![JFrog](jfrog-4.png)


## Lab - JFrog Private Docker Registry (Quick Test once you setup your JFrog Private Docker Registry)
```
docker login -ujegan@tektutor.org tektutor.jfrog.io
docker pull tektutor.jfrog.io/tektutor-docker/hello-world:latest
docker tag tektutor.jfrog.io/tektutor-docker/hello-world tektutor.jfrog.io/tektutor-docker/hello-world:1.0.0
docker push tektutor.jfrog.io/tektutor-docker/hello-world:1.0.0
```

Expected output
<pre>
[jegan@tektutor.org ~]$ docker login -ujegan@tektutor.org tektutor.jfrog.io
Password: 
WARNING! Your password will be stored unencrypted in /home/jegan/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
  
[jegan@tektutor.org ~]$ sudo su jegan
[sudo] password for jegan: 
[jegan@tektutor.org ~]$ docker pull tektutor.jfrog.io/tektutor-docker/hello-world:latest
latest: Pulling from tektutor-docker/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:53641cd209a4fecfc68e21a99871ce8c6920b2e7502df0a20671c6fccc73a7c6
Status: Downloaded newer image for tektutor.jfrog.io/tektutor-docker/hello-world:latest
tektutor.jfrog.io/tektutor-docker/hello-world:latest
  
[jegan@tektutor.org ~]$ docker tag tektutor.jfrog.io/tektutor-docker/hello-world tektutor.jfrog.io/tektutor-docker/hello-world:1.0.0
  
[jegan@tektutor.org ~]$ docker push tektutor.jfrog.io/tektutor-docker/hello-world:1.0.0
The push refers to repository [tektutor.jfrog.io/tektutor-docker/hello-world]
ac28800ec8bb: Layer already exists 
1.0.0: digest: sha256:d37ada95d47ad12224c205a938129df7a3e52345828b4fa27b03a98825d1e2e7 size: 524  
</pre>

## Lab - Building a Custom Docker Image and pushing to JFrog Private Docker Registry
```
cd ~/openshift-march-2024
git pull
cd Day5/spring-ms
docker build -t tektutor/hello-microservice:1.0 .
docker images
```

Expected output
<pre>
[jegan@tektutor.org spring-ms]$ ls -l
total 8
-rw-r--r--. 1 jegan jegan  217 Mar 22 10:31 Dockerfile
-rw-r--r--. 1 jegan jegan 1715 Mar 21 19:44 pom.xml
drwxr-xr-x. 3 jegan jegan   18 Mar 21 19:44 src

    
[jegan@tektutor.org spring-ms]$ # Build custom docker image with sample spring boot microservice
[jegan@tektutor.org spring-ms]$ <b>docker build -t tektutor/hello-microservice:1.0 .</b>
[+] Building 2.3s (11/11) FINISHED                                                               docker:default
 => [internal] load build definition from Dockerfile                                                       0.0s
 => => transferring dockerfile: 315B                                                                       0.0s
 => [internal] load metadata for registry.access.redhat.com/ubi8/openjdk-11:latest                         0.8s
 => [internal] load metadata for docker.io/library/maven:3.6.3-jdk-11                                      2.2s
 => [internal] load .dockerignore                                                                          0.0s
 => => transferring context: 2B                                                                            0.0s
 => [stage-1 1/2] FROM registry.access.redhat.com/ubi8/openjdk-11:latest@sha256:285b35387bd04f93bb2ad8a2d  0.0s
 => [internal] load build context                                                                          0.0s
 => => transferring context: 1.25kB                                                                        0.0s
 => [stage1 1/3] FROM docker.io/library/maven:3.6.3-jdk-11@sha256:1d29ccf46ef2a5e64f7de3d79a63f9bcffb4dc5  0.0s
 => CACHED [stage1 2/3] COPY . .                                                                           0.0s
 => CACHED [stage1 3/3] RUN mvn clean package                                                              0.0s
 => CACHED [stage-1 2/2] COPY --from=stage1 target/*.jar app.jar                                           0.0s
 => exporting to image                                                                                     0.0s
 => => exporting layers                                                                                    0.0s
 => => writing image sha256:ec169bcd065873433a41201a6141821e8e155dfea95e9c291592fefb2732c2af               0.0s
 => => naming to docker.io/tektutor/hello-microservice:1.0     
  0.0s
  
[jegan@tektutor.org spring-ms]$ <b>docker images</b>
REPOSITORY                                      TAG       IMAGE ID       CREATED         SIZE
tektutor/hello-microservice                     1.0       ec169bcd0658   15 hours ago    410MB
tektutor.jfrog.io/tektutor-docker/hello-world   1.0.0     d2c94e258dcb   10 months ago   13.3kB
tektutor.jfrog.io/tektutor-docker/hello-world   latest    d2c94e258dcb   10 months ago   13.3kB
  
[jegan@tektutor.org spring-ms]$ <b>docker run -d --name hello --hostname hello -p 80:8080 tektutor/hello-microservice:1.0</b>
bc4bea9f0f746b490650d444842fde6d69c4f84132cbd2c8ed4d3171ff8ced69
  
[jegan@tektutor.org spring-ms]$ <b>docker ps</b>
CONTAINER ID   IMAGE                             COMMAND               CREATED        STATUS        PORTS                                                       NAMES
bc4bea9f0f74   tektutor/hello-microservice:1.0   "java -jar app.jar"   1 second ago   Up 1 second   8443/tcp, 8778/tcp, 0.0.0.0:80->8080/tcp, :::80->8080/tcp   hello
[jegan@tektutor.org spring-ms]$ curl localhost
</pre>

We need to tag the hello-microservice docker image as per your JFrog portal url
```
cd ~/openshift-march-2024
git pull
cd Day5/spring-ms
pwd

docker images
docker tag tektutor/hello-microservice:1.0 tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0
docker images
```

Expected output
<pre>
[jegan@tektutor.org spring-ms]$ pwd
/home/jegan/openshift-march-2024/Day5/spring-ms
  
[jegan@tektutor.org spring-ms]$ <b>docker images</b>
REPOSITORY                                      TAG       IMAGE ID       CREATED         SIZE
tektutor/hello-microservice                     1.0       ec169bcd0658   16 hours ago    410MB
tektutor.jfrog.io/tektutor-docker/hello-world   1.0.0     d2c94e258dcb   10 months ago   13.3kB
tektutor.jfrog.io/tektutor-docker/hello-world   latest    d2c94e258dcb   10 months ago   13.3kB
  
[jegan@tektutor.org spring-ms]$ <b>docker tag tektutor/hello-microservice:1.0 tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0</b>
  
[jegan@tektutor.org spring-ms]$ <b>docker images</b>
REPOSITORY                                             TAG       IMAGE ID       CREATED         SIZE
tektutor/hello-microservice                            1.0       ec169bcd0658   16 hours ago    410MB
tektutor.jfrog.io/tektutor-docker/hello-microservice   1.0       ec169bcd0658   16 hours ago    410MB
tektutor.jfrog.io/tektutor-docker/hello-world          1.0.0     d2c94e258dcb   10 months ago   13.3kB
tektutor.jfrog.io/tektutor-docker/hello-world          latest    d2c94e258dcb   10 months ago   13.3kB  
</pre>

Let's push this image to our JFrog Private Docker Registry as shown below
```
docker image
docker push tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0
```

Expected output
<pre>
[jegan@tektutor.org spring-ms]$ <b>docker push tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0</b>
The push refers to repository [tektutor.jfrog.io/tektutor-docker/hello-microservice]
191a955daa2f: Pushed 
00a9cea1d198: Pushed 
f61c43e793f6: Pushed 
1.0: digest: sha256:04d0d1042191c75c68261d5017d1d10e0ef3f25f003f02916781b20ef6fb5704 size: 955  
</pre>

Let's try deploying our hello-microservice applicaton into Openshift using the image we pushed into your JFrog Artifactory
```
oc new-app --image=tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0
oc get po 
```

Expected output
<pre>
[jegan@tektutor.org spring-ms]$ <b>oc new-app --image=tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0</b>
W0322 12:09:36.522032 3222928 newapp.go:523] Could not find an image stream match for "tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0". Make sure that a container image with that tag is available on the node for the deployment to succeed.
--> Found container image ec169bc (16 hours old) from tektutor.jfrog.io for "tektutor.jfrog.io/tektutor-docker/hello-microservice:1.0"

    Java Applications 
    ----------------- 
    Platform for building and running plain Java applications (fat-jar and flat classpath)

    Tags: builder, java

--> Creating resources ...
    deployment.apps "hello-microservice" created
    service "hello-microservice" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/hello-microservice' 
    Run 'oc status' to view your app.
  
[jegan@tektutor.org spring-ms]$ <b>oc get po -w</b>
NAME                                  READY   STATUS         RESTARTS   AGE
hello-microservice-7dd969c847-6gvsg   0/1     ErrImagePull   0          12s
hello-microservice-7dd969c847-6gvsg   0/1     ImagePullBackOff   0          17s
hello-microservice-7dd969c847-6gvsg   0/1     ErrImagePull       0          31s  
</pre>

In order to download container image from our private JFrog Container Image registry, we need provide login credentials.  These login credentials we can save in a Openshift secret as shown below
```
oc create secret docker-registry private-jfrog-image-registry --docker-server=tektutor.jfrog.io --docker-username=<replace-this-with-your-jfrog-user-login> --docker-password=<replace-this-with-your-jfrog-access-token>

oc get secrets
```

Expected output
<pre>
[jegan@tektutor.org spring-ms]<b>oc get secrets</b>
NAME                       TYPE                                  DATA   AGE
builder-dockercfg-rsp4w    kubernetes.io/dockercfg               1      23m
builder-token-vz2lv        kubernetes.io/service-account-token   4      23m
default-dockercfg-fnztn    kubernetes.io/dockercfg               1      23m
default-token-g2jbg        kubernetes.io/service-account-token   4      23m
deployer-dockercfg-8tn5r   kubernetes.io/dockercfg               1      23m
deployer-token-fkcfz       kubernetes.io/service-account-token   4      23m
private-jfrog-image-registry    kubernetes.io/dockerconfigjson        1      8m5s  
</pre>

Now you may deploy the hello-microservice in declarative style as shown below
```
cd ~/openshift-march-2024
git pull
cd Day5/spring-ms
oc apply -f hello-microservice-deploy.yml

oc get po -w
```

Expected output
<pre>
jegan@tektutor.org spring-ms]$ ls
Dockerfile  hello-microservice-deploy.yml  pom.xml  src
[jegan@tektutor.org spring-ms]$ <b>oc apply -f hello-microservice-deploy.yml</b>
deployment.apps/hello created
  
[jegan@tektutor.org spring-ms]$ <b>oc get po -w</b>
NAME                     READY   STATUS              RESTARTS   AGE
hello-7b548bfc5f-chzcn   0/1     ContainerCreating   0          3s
hello-7b548bfc5f-ps872   0/1     ContainerCreating   0          3s
hello-7b548bfc5f-twlnc   0/1     ContainerCreating   0          3s
hello-7b548bfc5f-twlnc   1/1     Running             0          27s
hello-7b548bfc5f-ps872   1/1     Running             0          28s

[jegan@tektutor.org spring-ms]<b>oc get secrets</b>
NAME                       TYPE                                  DATA   AGE
builder-dockercfg-rsp4w    kubernetes.io/dockercfg               1      23m
builder-token-vz2lv        kubernetes.io/service-account-token   4      23m
default-dockercfg-fnztn    kubernetes.io/dockercfg               1      23m
default-token-g2jbg        kubernetes.io/service-account-token   4      23m
deployer-dockercfg-8tn5r   kubernetes.io/dockercfg               1      23m
deployer-token-fkcfz       kubernetes.io/service-account-token   4      23m
private-jfrog-image-registry    kubernetes.io/dockerconfigjson        1      8m5s  
</pre>


# Knative Serverless applications

## Info - What is OpenShiftServerless?
- Based on opensource knative project
- Deployment model for containers and functions that does resource allocation on demand
- Developers can focus on application development without needing to worry about infrastructe and deployment logistics
- Supports evern-driven architecture
- Can use any programming language or runtime
- Can run your application anywhere OpenShift runs
- Manage serverless applications natively within OpenShift

## Lab - Creating a knative service imperatively
```
kn service create --image=bitnami/nginx:latest
```

## Lab - Creating a knative service declaratively
```
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
 name: frontend
spec:
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - image: mgrimald/guestbook
        resources:
          requests:
             cpu: 100m
             memory: 100Mi
          env:
          - name: GET_HOSTS_FROM
            value: dns
          ports:
          - containerPort: 80
```

## Lab - BuildConfig - build container image cloning source code from GitHub repository
```
cd ~/openshift-march-2024
git pull

cd Day5/BuildConfig
oc apply -f imagestream.yml
oc get imagestreams
oc get imagestream
oc get is

oc apply -f buildconfig.yml
oc get buildconfigs
oc get buildconfig
oc get bc

oc get builds
oc get build

oc logs -f bc/spring-hello
```

Expected output
<pre>
jegan@tektutor.org BuildConfig]$ oc new-project jegan
Already on project "jegan" on server "https://api.ocp.tektutor.org.labs:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[jegan@tektutor.org BuildConfig]$ pwd
/home/jegan/openshift-march-2024/Day5/BuildConfig
  
[jegan@tektutor.org BuildConfig]$ ls -l
total 16
-rw-r--r--. 1 jegan jegan  408 Mar 22 15:40 buildconfig.yml
-rw-r--r--. 1 jegan jegan  217 Mar 22 15:46 Dockerfile
-rw-r--r--. 1 jegan jegan  132 Mar 22 15:42 imagestream.yml
-rw-r--r--. 1 jegan jegan 1715 Mar 22 15:46 pom.xml
drwxr-xr-x. 3 jegan jegan   18 Mar 22 15:46 src
  
[jegan@tektutor.org BuildConfig]$ oc apply -f imagestream.yml 
imagestream.image.openshift.io/tektutor-spring-hello created

    
[jegan@tektutor.org BuildConfig]$ oc get imagestreams
NAME                    IMAGE REPOSITORY                                                               TAGS   UPDATED
tektutor-spring-hello   image-registry.openshift-image-registry.svc:5000/jegan/tektutor-spring-hello          
[jegan@tektutor.org BuildConfig]$ oc get imagestream
NAME                    IMAGE REPOSITORY                                                               TAGS   UPDATED
tektutor-spring-hello   image-registry.openshift-image-registry.svc:5000/jegan/tektutor-spring-hello          
[jegan@tektutor.org BuildConfig]$ oc get is
NAME                    IMAGE REPOSITORY                                                               TAGS   UPDATED
tektutor-spring-hello   image-registry.openshift-image-registry.svc:5000/jegan/tektutor-spring-hello   
</pre>

Let's check the buildconfig output
<pre>
[jegan@tektutor.org BuildConfig]$ ls -l
total 16
-rw-r--r--. 1 jegan jegan  408 Mar 22 15:40 buildconfig.yml
-rw-r--r--. 1 jegan jegan  217 Mar 22 15:46 Dockerfile
-rw-r--r--. 1 jegan jegan  132 Mar 22 15:42 imagestream.yml
-rw-r--r--. 1 jegan jegan 1715 Mar 22 15:46 pom.xml
drwxr-xr-x. 3 jegan jegan   18 Mar 22 15:46 src
[jegan@tektutor.org BuildConfig]$ oc apply -f buildconfig.yml 
buildconfig.build.openshift.io/spring-hello created
[jegan@tektutor.org BuildConfig]$ 
[jegan@tektutor.org BuildConfig]$ 
[jegan@tektutor.org BuildConfig]$ oc get buildconfigs
NAME           TYPE     FROM   LATEST
spring-hello   Docker   Git    1
[jegan@tektutor.org BuildConfig]$ oc get buildconfig
NAME           TYPE     FROM   LATEST
spring-hello   Docker   Git    1
[jegan@tektutor.org BuildConfig]$ oc get bc
NAME           TYPE     FROM   LATEST
spring-hello   Docker   Git    1
[jegan@tektutor.org BuildConfig]$ oc get builds
NAME             TYPE     FROM          STATUS    STARTED          DURATION
spring-hello-1   Docker   Git@a94193c   Running   16 seconds ago   
[jegan@tektutor.org BuildConfig]$ oc get build
NAME             TYPE     FROM          STATUS    STARTED          DURATION
spring-hello-1   Docker   Git@a94193c   Running   18 seconds ago   
[jegan@tektutor.org BuildConfig]$ oc logs -f bc/spring-hello
Cloning "https://github.com/tektutor/openshift-march-2024.git" ...
	Commit:	a94193c53acf297b2a9848787ac258524f0b8e70 (Update README.md)
	Author:	Jeganathan Swaminathan <mail2jegan@gmail.com>
	Date:	Fri Mar 22 15:54:43 2024 +0530
time="2024-03-22T10:28:21Z" level=info msg="Not using native diff for overlay, this may cause degraded performance for building images: kernel has CONFIG_OVERLAY_FS_REDIRECT_DIR enabled"
I0322 10:28:21.340760       1 defaults.go:112] Defaulting to storage driver "overlay" with options [mountopt=metacopy=on].
Caching blobs under "/var/cache/blobs".

Pulling image docker.io/maven:3.6.3-jdk-11 ...
Trying to pull docker.io/library/maven:3.6.3-jdk-11...
Getting image source signatures
Copying blob sha256:6c215442f70bd949a6f2e8092549943905e2d4f9c87a4f532d7740ae8647d33a
Copying blob sha256:5d6f1e8117dbb1c6a57603cb4f321a861a08105a81bcc6b01b0ec2b78c8523a5
Copying blob sha256:48c2faf66abec3dce9f54d6722ff592fce6dd4fb58a0d0b72282936c6598a3b3
Copying blob sha256:234b70d0479d7f16d7ee8d04e4ffdacc57d7d14313faf59d332f18b2e9418743
Copying blob sha256:004f1eed87df3f75f5e2a1a649fa7edd7f713d1300532fd0909bb39cd48437d7
Copying blob sha256:d7eb6c022a4e6128219b32a8e07c8c22c89624ff440ebac1506121794bc15ccc
Copying blob sha256:355e8215390faee903502a9fddfc65cd823f1606f053376ba2575adce66974a1
Copying blob sha256:cf5eb43522f68d7e2347e19ad70dadcf1594d25b792ede0464c2936ff902c4c6
Copying blob sha256:4fee0489a65b64056f81358639bfe85fd87776630830fd02ce8c15e34928bf9c
Copying blob sha256:413646e6fa5d7bcd9722d3e400fc080a77deb505baed79afa5fedae23583af25
Copying config sha256:e23b595c92ada5c9f20a27d547ed980a445f644eb1cbde7cfb27478fa38c4691
Writing manifest to image destination

Pulling image registry.access.redhat.com/ubi8/openjdk-11 ...
Trying to pull registry.access.redhat.com/ubi8/openjdk-11:latest...
Getting image source signatures
Copying blob sha256:0bb48edf8994fcf133c612f92171d68f572091fb0b1113715eab5f3e5e7f54e5
Copying blob sha256:74e0c06e5eac269967e6970582b9b25168177df26dffed37ccde09369302a87a

Copying blob sha256:48c2faf66abec3dce9f54d6722ff592fce6dd4fb58a0d0b72282936c6598a3b3
Copying blob sha256:234b70d0479d7f16d7ee8d04e4ffdacc57d7d14313faf59d332f18b2e9418743
Copying blob sha256:004f1eed87df3f75f5e2a1a649fa7edd7f713d1300532fd0909bb39cd48437d7
Copying blob sha256:d7eb6c022a4e6128219b32a8e07c8c22c89624ff440ebac1506121794bc15ccc
Copying blob sha256:355e8215390faee903502a9fddfc65cd823f1606f053376ba2575adce66974a1
Copying blob sha256:cf5eb43522f68d7e2347e19ad70dadcf1594d25b792ede0464c2936ff902c4c6
Copying blob sha256:4fee0489a65b64056f81358639bfe85fd87776630830fd02ce8c15e34928bf9c
Copying blob sha256:413646e6fa5d7bcd9722d3e400fc080a77deb505baed79afa5fedae23583af25
Copying config sha256:e23b595c92ada5c9f20a27d547ed980a445f644eb1cbde7cfb27478fa38c4691
Writing manifest to image destination

Pulling image registry.access.redhat.com/ubi8/openjdk-11 ...
Trying to pull registry.access.redhat.com/ubi8/openjdk-11:latest...
Getting image source signatures
Copying blob sha256:0bb48edf8994fcf133c612f92171d68f572091fb0b1113715eab5f3e5e7f54e5
Copying blob sha256:74e0c06e5eac269967e6970582b9b25168177df26dffed37ccde09369302a87a
Copying config sha256:a6b53e10c7678edc1d2e8090ed0a0b40d147f8e110ac2277931828ef11276f96
Writing manifest to image destination
Adding transient rw bind mount for /run/secrets/rhsm
[1/2] STEP 1/3: FROM docker.io/maven:3.6.3-jdk-11 AS stage1
[1/2] STEP 2/3: COPY . .
--> c5264029e5f0
[1/2] STEP 3/3: RUN mvn clean package
[INFO] Scanning for projects...
Downloading from central: https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-starter-parent/2.4.2/spring-boot-starter-parent-2.4.2.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-starter-parent/2.4.2/spring-boot-starter-parent-2.4.2.pom (8.6 kB at 24 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-dependencies/2.4.2/spring-boot-dependencies-2.4.2.pom
[2/2] COMMIT temp.builder.openshift.io/jegan/spring-hello-1:7d550e00
--> 0e87f95533d4
Successfully tagged temp.builder.openshift.io/jegan/spring-hello-1:7d550e00
0e87f95533d464ca205e5e8b3e0a6a1ca34be096cd0f43df0302acb433cf4e0a

Pushing image image-registry.openshift-image-registry.svc:5000/jegan/tektutor-spring-hello:latest ...
Getting image source signatures
Copying blob sha256:8ffbb4d0fd9e01bfa5a94d0f565e8c1f4d95cd76df979fb01768e168e006d2ef
Copying blob sha256:74e0c06e5eac269967e6970582b9b25168177df26dffed37ccde09369302a87a
Copying blob sha256:0bb48edf8994fcf133c612f92171d68f572091fb0b1113715eab5f3e5e7f54e5
Copying config sha256:0e87f95533d464ca205e5e8b3e0a6a1ca34be096cd0f43df0302acb433cf4e0a
Writing manifest to image destination
Successfully pushed image-registry.openshift-image-registry.svc:5000/jegan/tektutor-spring-hello@sha256:69cfb72162c69b3adc19a78ae63c793f8db2c7ded33b012b5e8c129d33f1b544
Push successful  
</pre>


## Lab - Securing our application with edge route (https)
Upgrading the openssl in CentOS
https://webhostinggeeks.com/howto/install-update-openssl-centos/

Find your Openshift cluster domain
```
oc get ingresses.config/cluster -o jsonpath={.spec.domain}
```
Expected similar output
<pre>
[jegan@tektutor.org openshift-march-2024]$ oc get ingresses.config/cluster -o jsonpath={.spec.domain}
apps.ocp4.training.tektutor
</pre>


Installing build tools to compile openssl from source code
```
sudo yum groupinstall 'Development Tools'
sudo yum install perl-IPC-Cmd perl-Test-Simple
cd /usr/src
wget https://www.openssl.org/source/openssl-3.0.0.tar.gz
tar -zxf openssl-3.0.0.tar.gz
rm openssl-3.0.0.tar.gz
dnf install perl
cd /usr/src/openssl-3.0.0
./config
make
make test
make install
ln -s /usr/local/lib64/libssl.so.3 /usr/lib64/libssl.so.3
ln -s /usr/local/lib64/libcrypto.so.3 /usr/lib64/libcrypto.so.3
openssl version
```

Let's deploy a microservice and create an edge route as shown below
```
cd ~/openshift-march-2024
git pull
cd Day5/edge-route
oc new-app nginx --image=bitnmai/nginx:latest 
oc expose deploy/nginx

openssl genrsa -out key.key
openssl req -new -key key.key -out csr.csr -subj="/CN=nginx-jegan.apps.ocp.tektutor.org.labs"
openssl x509 -req -in csr.csr -signkey key.key -out crt.crt
oc create route edge --service hello --hostname nginx-jegan.apps.ocp.tektutor.org.labs --key key.key --cert crt.crt
```

Expected output
<pre>
[jegan@tektutor.org edge-route]$ oc get svc
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
nginx   ClusterIP   172.30.208.33   <none>        8080/TCP   87m
	
[jegan@tektutor.org edge-route]$ oc expose deploy/nginx --port=8080
service/nginx exposed

[jegan@tektutor.org edge-route]$ oc get svc
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
nginx       ClusterIP   172.30.16.165   <none>        8080/TCP   4s

[jegan@tektutor.org edge-route]$ oc get ingresses.config/cluster -o jsonpath={.spec.domain}
apps.ocp.tektutor.org.labs

[jegan@tektutor.org edge-route]$ oc project
Using project "jegan-devops" on server "https://api.ocp4.training.tektutor:6443".

[jegan@tektutor.org edge-route]$ openssl req -new -key key.key -out csr.csr -subj="/CN=nginx-jegan.apps.ocp.tektutor.org.labs"

[jegan@tektutor.org edge-route]$ openssl x509 -req -in csr.csr -signkey key.key -out crt.crt

[jegan@tektutor.org edge-route]$ oc create route edge --service nginx --hostname nginx-jegan.apps.ocp.tektutor.org.labs --key key.key --cert crt.crt
route.route.openshift.io/nginx created

[jegan@tektutor.org edge-route]$ oc get route
NAME    HOST/PORT                                        PATH   SERVICES   PORT    TERMINATION   WILDCARD
nginx   nginx-jegan.apps.ocp.tektutor.org.labs          nginx      <all>   edge          None
</pre>                                                                                                                                 

## Post test link for Openshift training
https://app.mymapit.in/code4/tiny/hpLlQw

## Feedback link
https://survey.zohopublic.com/zs/ZpD42A

