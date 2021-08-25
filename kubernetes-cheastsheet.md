Kubernetes
==========

## Install

### Prerequisites
1. Bash v5+ checkout [Upgrading Bash on macOS](https://itnext.io/upgrading-bash-on-macos-7138bd1066ba)
2. bash-completion@2

### Install Docker and Kubernetes(k8s)
> Installing *Docker* and *Kubernetes* on **MacOS** is eazy. 

Download and install `Docker for Mac` **Edge** Version. [Download Link](https://hub.docker.com/editions/community/docker-ce-desktop-mac)

After installation, you get `Docker` engine with option to enable `Kubernetes` and `kubectl` cli tool on your `MacOS`.

### Install bash-completion for MacOS (Bash v5+)
```bash
brew install bash-completion@2
```
Paste this into your ~/.extra or ~/.bash_profile  file:
```bash
# bash-completion used with Bash v5+
export BASH_COMPLETION_COMPAT_DIR="/usr/local/etc/bash_completion.d"
[[ -r "/usr/local/etc/profile.d/bash_completion.sh" ]] && . "/usr/local/etc/profile.d/bash_completion.sh"
```

### Enable kubectl auto-completion for MacOS (Bash v5+)
```bash
kubectl completion bash > $(brew --prefix)/etc/bash_completion.d/kubectl
alias k=kubectl
complete -F __start_kubectl k
```

### Creating a Kubernetes cluster
1. After Docker for Mac is installed, configure it with sufficient resources. You can do that via the [Advanced menu](https://docs.docker.com/docker-for-mac/#advanced) in Docker for Mac's preferences. Set **CPUs** to at least **4** and Memory to at least **8.0 GiB**.
2. Now enable Docker for Mac's [Kubernetes capabilities](https://docs.docker.com/docker-for-mac/#kubernetes) and wait for the cluster to start up.
3. Install [kubernetic](https://kubernetic.com/) app. This works as replacement for `kubernetes-dashboard`
4. Follow instructions [here](https://github.com/knative/docs/blob/master/install/Knative-with-Docker-for-Mac.md) and [here](https://polarsquad.github.io/istio-workshop/) to setup **Istio** and **Knative**. 

---

## Example

### Microboot
  [Microboot](https://hub.docker.com/r/dontrebootme/microbot) The microbot image is an image I use mostly for cluster scheduler demonstrations. It's simply an alpine linux base with nginx and a wrapper script to make the static webpage show the container hostname. Tags :v1 and :v2 have different images they display making it simple to demonstrate rolling upgrades via a load balancer etc.
```bash
kubectl create deployment microbot --image=dontrebootme/microbot:v1
microk8s kubectl expose deployment microbot --type=NodePort --port=80 --name=microbot-service
```


## Install Tools (Optional)


### Skaffold
  [Skaffold](https://skaffold.dev/docs/) is a command line tool (from Google) that facilitates continuous development for Kubernetes applications.
  It also provides building blocks and describe customizations for a CI/CD pipeline.
```bash
brew install skaffold
skaffold version
```

### Helm
  [helm][1] has client-side cli and server-side `tiller` components
  
  Install Helm via `brew`. More info [Here](https://collabnix.com/kubernetes-application-deployment-made-easy-using-helm-on-docker-for-mac-18-05/)
  
```bash
# install helm cli on mac with brew
brew install kubernetes-helm
```
#### To begin working with Helm 
  install tiller into the kube-system
  This will install Tiller to your running Kubernetes cluster.
  It will also set up any necessary local configuration.
```bash
helm init
```

#### Check if it is working 
```
# check version
helm version
# show if tiller is installed
kubectl get pods --namespace kube-system
# upgrade helm version
helm init --upgrade
```

#### Using Helm
```
# update charts repo
helm repo update

# install postgre chart
# helm install --name nginx stable/nginx-ingress
helm install --name pg --namespace default --set postgresPassword=postgres,persistence.size=1Gi stable/postgresql
kubectl get pods -n default

# list installed charts
helm ls

# delete postgre
$ helm delete my-postgre

# delete postgre and purge
$ helm delete --purge my-postgre
```

#### You can also create your own Chart by using the scaffolding command 
```bash
helm create mychart
```
  This will create a folder which includes all the files necessary to create your own package :
```
├── Chart.yaml
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   └── service.yaml
└── values.yaml
```

#### optionally add `helm-secrets` [plugin](https://developer.epages.com/blog/tech-stories/kubernetes-deployments-with-helm-secrets/)

```bash
helm plugin install https://github.com/futuresimple/helm-secrets 
```

### Ingress Controller with Traefik
> based on [Docker for Mac with Kubernetes — Ingress Controller with Traefik](https://medium.com/@thms.hmm/docker-for-mac-with-kubernetes-ingress-controller-with-traefik-e194919591bb)

`cd .deploy/traefik`
    
1. Create a file called `traefik-values.yaml`.
    ```yaml
    dashboard:
      enabled: true
      domain: traefik.k8s
    ssl:
      enabled: true
      insecureSkipVerify: true
    kubernetes:
      namespaces:
        - default
        - kube-system
    ```

2. Install the Traefik Chart and check if the pod is up and running.
    ```bash
    helm install stable/traefik --name=traefik --namespace=kube-system -f traefik-values.yaml
    kubectl get pods --namespace=kube-system
    kubectl get ingress traefik-dashboard --namespace=kube-system -o yaml
    # to see traefik logs
    kubectl logs $(kubectl get pods --namespace=kube-system -lapp=traefik -o jsonpath='{.items[0].metadata.name}') -f --namespace=kube-system
    # To update, if you change `traefik-values.yaml` later
    helm upgrade --namespace=kube-system  -f traefik-values.yaml traefik stable/traefik
    ```

3. Add your domains to MacOS `/etc/hosts` as needed. Other options:  `wildcard DNS in localhost development` [1](https://gist.github.com/eloypnd/5efc3b590e7c738630fdcf0c10b68072), [2](https://medium.com/localz-engineering/kubernetes-traefik-locally-with-a-wildcard-certificate-e15219e5255d)

    ```
    127.0.0.1       localhost traefik.k8s web.traefik.k8s keycloak.traefik.k8s 
    ```

4. Deploying the K8s dashboard and check if the pod is up and running.
    ```
    cd .deploy/traefik
    git clone https://github.com/thmshmm/chart-k8s-dashboard.git k8s-dshbrd/
    helm install k8s-dshbrd --name kubernetes-dashboard --namespace=kube-system
    kubectl get ingress kubernetes-dashboard --namespace=kube-system -o yaml
    ```


### kompose
> cli tool to conver Docker Compose files to Kubernetes
```bash
# install
brew install kompose
# to use
kompose convert -f docker-compose.yaml
```

### kube-ps1
optionally add Kubernetes prompt info for bash
```bash
brew install kube-ps1
```

### Kubefwd
> [kubefwd](https://github.com/txn2/kubefwd) is a command line utility built to port forward some or all pods within a Kubernetes namespace
#### Install
```bash
# If you are running MacOS and use homebrew you can install kubefwd directly from the txn2 tap:
brew install txn2/tap/kubefwd
# To upgrade
brew upgrade kubefwd
```
#### Usage
```bash
# Forward all services for the namespace the-project:
sudo kubefwd services -n the-project
# Forward all services for the namespace the-project where labeled system: wx:
sudo kubefwd services -l system=wx -n the-project
```

---

## Usage 

### kubectl Cheat Sheets
> To read more on kubectl, check out the [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).


### Kubectl commands
> commonly used Kubectl commands

> you can pratice kubectl commands at [katacoda](https://www.katacoda.com/courses/kubernetes/playground) playground

```
kubectl version
kubectl cluster-info
kubectl get storageclass
kubectl get nodes
kubectl get ep kube-dns --namespace=kube-system
kubectl get persistentvolume
kubectl get  PersistentVolumeClaim --namespace default
kubectl get pods --namespace kube-system
kubectl get ep
kubectl get sa
kubectl get serviceaccount
kubectl get clusterroles
kubectl get roles
kubectl get ClusterRoleBinding
# Show Merged kubeconfig settings.
kubectl config view
kubectl config get-contexts
# Display the current-context
kubectl config current-context           
kubectl config use-context docker-desktop
kubectl port-forward service/ok 8080:8080 8081:80 -n the-project
# Delete evicted pods
kubectl get po --all-namespaces | awk '{if ($4 ~ /Evicted/) system ("kubectl -n " $1 " delete pods " $2)}'
```

### Namespaces and Context

> Execute the kubectl Command for Creating Namespaces
```bash
# Namespace for Developers
kubectl create -f namespace-dev.json
# Namespace for Testers
kubectl create -f namespace-qa.json
# Namespace for Production
kubectl create -f namespace-prod.json
```

> Assign a Context to Each Namespace
```
# Assign dev context to development namespace
kubectl config set-context dev --namespace=dev --cluster=minikube --user=minikube
# Assign qa context to QA namespace
kubectl config set-context qa --namespace=qa --cluster=minikube --user=minikube
# Assign prod context to production namespace
kubectl config set-context prod --namespace=prod --cluster=minikube --user=minikube
```

> Switch to the Appropriate Context
```
# List contexts
kubectl config get-contexts
# Switch to Dev context
kubectl config use-context dev
# Switch to QA context
kubectl config use-context qa
# Switch to Prod context
kubectl config use-context prod

kubectl config current-context
```

> see cluster-info
```bash
kubectl cluster-info
```
> nested kubectl commands

```bash
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}') 8082:8088
```

> kubectl proxy creates proxy server between your machine and Kubernetes API server.
By default it is only accessible locally (from the machine that started it).

```
kubectl proxy --port=8080
curl http://localhost:8080/api/
curl http://localhost:8080/api/v1/namespaces/default/pods
```

### Accessing logs
```bash
# get all the logs for a given pod:
kubectl logs my-pod-name
# keep monitoring the logs
kubectl -f logs my-pod-name
# Or if you have multiple containers in the same pod, you can do:
kubectl -f logs my-pod-name internal-container-name
# This allows users to view the diff between a locally declared object configuration and the current state of a live object.
kubectl alpha diff -f mything.yml
```

### Execute commands in running Pods
```bash
kubectl exec -it my-pod-name -- /bin/sh
```

### CI/CD
> Redeploy newly build image to existing k8s deployment
```
BUILD_NUMBER = 1.5.0-SNAPSHOT // GIT_SHORT_SHA
kubectl diff -f sample-app-deployment.yaml
kubectl -n=staging set image -f sample-app-deployment.yaml sample-app=xmlking/ngxapp:$BUILD_NUMBER
```

### Rolling back deployments
> Once you run `kubectl apply -f manifest.yml`
```bash
# To get all the deploys of a deployment, you can do:
kubectl rollout history deployment/DEPLOYMENT-NAME
# Once you know which deploy you’d like to roll back to, you can run the following command (given you’d like to roll back to the 100th deploy):
kubectl rollout undo deployment/DEPLOYMENT_NAME --to-revision=100
# If you’d like to roll back the last deploy, you can simply do:
kubectl rollout undo deployment/DEPLOYMENT_NAME
```

### Tips and Tricks
```bash
# Show resource utilization per node:
kubectl top node
# Show resource utilization per pod:
kubectl top pod
# if you want to have a terminal show the output of these commands every 2 seconds without having to run the command over and over you can use the watch command such as
watch kubectl top node
# --v=8 for debuging 
kubectl get po --v=8
```

####  troubleshoot headless services  
```bash
k get ep
# ssh to one of the container and run dns check:
host <httpd-discovery>
```

#### Alias

```bash
alias k="kubectl"
alias watch="watch "
alias kg="kubectl get"
alias kgdep="kubectl get deployment"
alias ksys="kubectl --namespace=kube-system"
alias kd="kubectl describe"
alias bb="kubectl run busybox --image=busybox:1.30.1 --rm -it --restart=Never --command --"
```
### Microk8s Alias example
```bash
echo "alias kubectl='microk8s kubectl'" >> .bashrc
```

> you can use `busybox` for debuging inside cluster

```bash
bb nslookup demo
bb wget -qO- http://demo:8888
bb sh
```
 
#### Container Security
> for better security add following securityContext settings to manifest
```yaml
securityContext:
  # Blocking Root Containers
  runAsNonRoot: true
  # Setting a Read-Only Filesystem
  readOnlyRootFilesystem: true
  # Disabling Privilege Escalation
  allowPrivilegeEscalation: false
  # For maximum security, you should drop all capabilities, and only add specific capabilities if they’re needed:
    capabilities:
      drop: ["all"]
      add: ["NET_BIND_SERVICE"]
```


#### Debug k8s

For many steps here you will want to see what a `Pod` running in the k8s cluster sees. The simplest way to do this is to run an interactive busybox `Pod`:
```bash
kubectl run -it --rm --restart=Never busybox --image=busybox sh
```

#### Debugging with an ephemeral debug container

Ephemeral containers are useful for interactive troubleshooting when `kubectl exec` is insufficient because a container has crashed or a container image doesn't include debugging utilities, such as with `distroless` images. 

This allows a user to inspect a running pod without restarting it and without having to enter the container itself to, for example, check the filesystem, execute additional debugging utilities, or initial network requests from the pod network namespace. Part of the motivation for this enhancement is to also eliminate most uses of SSH for node debugging and maintenance

```bash
# First, create a pod for the example: 
kubectl run ephemeral-demo --image=k8s.gcr.io/pause:3.1 --restart=Never
# add a debugging container 
kubectl alpha debug -it ephemeral-demo --image=busybox --target=ephemeral-demo
```

#### Generateing k8s YAML from local files using `--dry-run`
```bash
# generate a kubernetes tls file
kubectl create secret tls keycloak-secrets-tls \
--key tls.key --cert tls.crt \
-o yaml --dry-run > 02-keycloak-secrets-tls.yml
```

#### iTerm2 tips
> in iTerm2
1. split screen horizontally
2. go to the bottom screen and split it vertically

I was using top screen for the work with yaml files and kubectl.

Left bottom screen was running:

    watch kubectl get pods

Right bottom screen was running:

    watch "kubectl get events --sort-by='{.lastTimestamp}' | tail -6"

With such setup it was easy to observe in real time how my pods are being created.



---


## Reference 

[1]: https://docs.helm.sh/using_helm/#installing-helm
1. [Debug Services](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
1. [debug-running-pod](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/)
1. [Docker for Mac with Kubernetes — Enable Ingress and K8S Dashboard](https://medium.com/@thms.hmm/docker-for-mac-with-kubernetes-ingress-controller-with-traefik-e194919591bb)
1. [Example recipes for Kubernetes Network Policies](https://github.com/ahmetb/kubernetes-network-policy-recipes)
1. [How To Use GPG on the Command Line](http://blog.ghostinthemachines.com/2015/03/01/how-to-use-gpg-command-line/)
5. [Using Your YubiKey with OpenPGP](https://support.yubico.com/support/solutions/articles/15000006420-using-your-yubikey-with-openpgp)
1. [Kubernetes Deployments with Helm - Secrets](https://developer.epages.com/blog/tech-stories/kubernetes-deployments-with-helm-secrets/)


### Apuntes

```bash
#Consultar puertos:
kubectl get svc postgres



#Delete container examples: 
kubectl delete service postgres 
kubectl delete deployment postgres
kubectl delete configmap postgres-config
kubectl delete persistentvolumeclaim postgres-pv-claim
kubectl delete persistentvolume postgres-pv-volume

#describe
kubectl describe pod pgadmin-68c84bbb57-vqrlw

#logs
kubectl logs pgadmin

#Connect container pod 

kubectl exec -it pod-name -- bash

#Login user DB
psql -U postgres
```