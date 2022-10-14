# GitOps PoC with ArgoCD
PoC is to have small clusters to emulate the workload lifecycle, staging and prod. Each with two additional namespaces, e.g., "apple" and "orange".
Point ArgoCD workload to the gitRepo for installation of some arbitrary workloads, then exercise the upgrade process first for staging, then for prod

## Prerequisites
* Kubernetes Cluster. To create local Kubernetes cluster, use [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
* kubectl Cli

## Install ArgoCD on your Cluster
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## Clone Repository
Clone repository to your local device.
```
git clone https://github.com/upendrasahu/gitops-argocd-poc.git
```

### Git Repository Hierarchy
Folder structure below is used in this project. You are free to change it.
```
/
├── stage                    # App workload for yaml for staging environment
│ ├── apple                  # Workload for apple namespace
│ └── orange                 # workload for orange namespace
├── prod                     # App workload for yaml for prod environment
│ ├── apple                  # Workload for apple namespace
│ └── orange                 # workload for orange namespace
├── apps                     # stores Argo CD app of app pattern installation which will point to github repo
│ ├── prod                   # App workload for prod cluster
│ ├── stage                  # App workload for stage cluster
│ └── argo-git-sercret.yaml  # Secret yaml for the github repo
```

## Setup ArgoCD workload
Below steps are for stage cluster, follow similar steps for prod by changing stage to prod in file paths
1. Create namespaces
```
kubectl create namespace apple
kubectl create namespace orange
```
2. Create Secret
```
# Update GitHub token in apps/argo-git-sercret.yaml file
kubectl apply -f apps/argo-git-sercret.yaml
```
3. Create app workload for apple namespace
```
kubectl apply -f apps/stage/apple.yaml
```

3. Create app workload for orange namespace
```
kubectl apply -f apps/stage/orange.yaml
```

4. Check ArgoCD applications created
```
kubectl get applications -n argocd
NAME             SYNC STATUS   HEALTH STATUS
apple            Synced        Healthy
orange           Synced        Healthy
```
## Validate
1. Once apple and orange applications are synced, it will create additional ArgoCD application
```
kubectl get applications -n argocd
NAME             SYNC STATUS   HEALTH STATUS
apple            Synced        Healthy
orange           Synced        Healthy
kube-metrics     Synced        Healthy
prometheus-app   Synced        Healthy
```

2. Workloads created in apple and orange namespaces
```
kubectl get pods -n apple
NAME                                              READY   STATUS    RESTARTS   AGE
kube-metrics-kube-state-metrics-b4858c858-k9zh7   1/1     Running   0          30s

kubectl get pods -n orange
NAME                                              READY   STATUS    RESTARTS        AGE
kube-prometheus-stack-operator-5cf7947445-lndn2   1/1     Running   0               40s
prometheus-kube-prometheus-stack-prometheus-0     2/2     Running   0               40s
```
