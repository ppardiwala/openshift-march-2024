# Day 3

## Info - Deployment vs DeploymentConfig

- In older version of Kubernetes to deploy applications we had to use ReplicationController
- ReplicationController supports
  - Scale up/down
  - Rolling update
  - it doesn't support declarative application deployment
  - during this time, OpenShift introducted a new type of Resource called DeploymentConfig to deploy applications
  - meanwhile, Kubernetes new versions introducted Deployment and Replicaset as an alternate to ReplicationController
  - Deployment & ReplicaSet
    - supports scale up/down
    - supports rolling update
    - its supports creating deployment in imperative and declarative style
    - in new version of K8s and OpenShift, it is recommended to use Deployment over the DeploymentConfig
  - ReplicationController and Deploymentconfig are deprecated but exists for backward compatibility

## Installing Helm Kubernetes/OpenShift Package Manager
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
