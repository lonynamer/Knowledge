### Kubernetes Knowledge
#### Kubernetes General Documentation & References
---```
- Kubernetes Docs
https://kubernetes.io/docs/
- Kubernetes Course File
https://github.com/jleetutorial/kuburnetes-demo
---```
#### Clients
- "kubectl" is the client commands tool.
- "Kitematic" is a visual docker client.

#### Servers
- "Minikube" is a standalone single server type of Kubernetes. Can be used for studying purposes.
   The things runs the same ways like on a complex Kubernetes infrastructure.
- On windows and old MAC "docker toolbox" and it runs inside "virtual box". 

#### Starting Minikube and 
minikube start
kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080

kubectl expose deployment hello-minikube --type=NodePort

kubectl get pod

minikube service hello-minikube --url

# Get files
https://hub.com/jleetutorial/dockerapp.git/
  
# How to Install kubectl and minikube on  Linux: 
  
$curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.23.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
 
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.23.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
$ minikube start
kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080

kubectl expose deployment hello-minikube --type=NodePort

kubectl get pod

minikube service hello-minikube --url

# Get files
https://hub.com/jleetutorial/dockerapp.git/
  
# How to Install kubectl and minikube on  Linux: 
  
$curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.23.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
 
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.23.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
$ minikube start
$ kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
$ kubectl expose deployment hello-minikube  --type=NodePort
$ kubectl get pod
$ curl $(minikube service hello-minikube --url)
$ kubectl delete deployment hello-minikube
$ minikube stop
$ kubectl apply -f ./deployment.yaml


$ kubectl get pods
$ kubectl get pods [pod name] 
$ kubectl expose <type name> <identifier/name> [—port=external port] [—target-port=container-port [—type=service-type]
$ kubectl expose deployment tomcat-deployment --type=NodePort 
$ kubectl port-forward <pod name> [LOCAL_PORT:]REMOTE_PORT] 
$ kubectl attach <pod name> -c <container>  
$ kubectl exec  [-it] <pod name> [-c CONTAINER] — COMMAND [args…]
$ kubectl exec -it <pod name> bash  
$ kubectl label [—overwrite] <type> KEY_1=VAL_1 ….
$ kubectl label pods <pod name> healthy=fasle  
$ kubectl run <name> —image=image
$ kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
$ kubectl describe pod

#Yaml File Example

apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  selector:
    matchLabels:
      app: tomcat
  replicas: 1
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:9.0
        ports:
        - containerPort: 8080


# PORT FORWARDING FOR A SINGLE POD
$ kubectl port-forward tomcat-deployment-794dbf9966-7lmtq 5000:6000

# RUN A COMMAND, MAYBE BASH
 kubectl exec -it tomcat-deployment-794dbf9966-7lmtq bash

# GET THE CONTAINERS IN A POD

# KUBECTL ATTACH
 kubectl attach tomcat-deployment-794dbf9966-7lmtq -c <container>

# LABELING PODS
# Labeling means giving a key=value
 kubectl label pod tomcat-deployment-794dbf9966-7lmtq healthy=false

# Running an image
kubernetes-demo$ kubectl run hazelcast --image=hazelcast --port=5701

# CHEAT SHEAT REFERENCE
https://kubernetes.io/docs/reference/kubectl/userguide/
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

# Architecture:
Client > Master (API, Scheduler, Controller, ETCD) > Node (Kubelet(Runs pods)+Kube-Proxy(Glue for networking between pods), Pod, Docker) > Service Loadbalancer


# SCALING APPLICATIONS
# APPLICATION TYPES
# - Stateful and Stateless. It's important for scaling

# SCALING DEFINITIONS TO SCALE
# - replica
# - ReplicaSet
# - BarePods
# - Jobs
# - Daemon Sets

# CLUSTER INFO
 kubectl cluster-info

# REPLICA
kubectl scale --replicas=4 deployment/tomcat-deployment
kubectl get deployment
kubectl describe deployment
# or replicas: 2 branch under specs:

# LOAD BALANCER
kubectl expose deployment tomcat-deployment --type=LoadBalancer --port=8080 --target-port=8080 --name tomcat-load-balancer


# CREATE, UPDATE, APPLY ROLLING UPDATES, ROLLBACK, PAUSE & RESUME ALL POSSIBLE
kubectl get deployments
kubectl rollout status deployment tomcat-deployment
kubectl set image deployment/tomcat-deployment tomcat=tomcat:9.0.1
kubectl roleout history deployment/tomcat-deployment
# AFTER HISTORY YOU GET REVISION NUMBERS AND CHECK THE CHANGES
kubectl rollout history deployment/tomcat-deployment --revision=2

# LABELS AND APPLY
# LABEL EXAMPLE ADD INSIDE YAML
template: > spec: > nodeSelector: > storageType: ssd
# AND RUN COMMAND
kubectl apply -f tomcat-deploy.yml
kubectl label node ip-172-31-42-57 storageType=ssd
# the deployment only work on storageType=ssd labeled machines.


# HEALT CHECK
# - READINESS(If it came up), LIVENESS(If it's working) 
# You can check by running command or tcp,http request.
# spec: > template: > spec: > container:
livenessProbe:
  httpGet:
    path: /
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 30
readinessProbe:
  httpGet:
    path: /
    port: 8080
  initialDelaySeconds: 30  # Period to fail
  periodSeconds: 30         # Period

# KUBE WEB UI > Runs on Master
# You can do the things in KUBE WEB UI as well
# Documentation
# https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
# ACCESSING WEB UI
https://<master-ip>:<apiserver-port>/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy

# EXCERCISE > INSTALL 4 REPLICA MONGODB
# MONGO RUNS ON PORT 27017 and supported by docker.
kubectl run mongo-exercise-1 --image=mongo --port 27017
kubectl scale --replicas=4 deployment/mongo-exercise-1
# You can also install using deployment and service yaml or helm package manager.

# DNS & SERVICE DISCOVERY
# Used for decoupling but working together coupling together.independent modules
# BUILD IN DNS
# Kubernetes configures kubelets to provide DNS IP to containers to solve DNS Names.
<my-srv-name>.<my-name-space>.svc.cluster.local
# NAMESPACES, by default everything runs under default cluster, but namespaces can create logical clusters. Used for large clusters or large organizations or departments.
# It should be configured inside /etc/resolv.conf



# VOLUMES
# Supports Azzue Disk & File, AWS EBS, Google Persistent Disk
# SAN/File System/Hardware
# CephFS, Fiber Channel, GlusterFS, NFS, iSCSI, Local(Not supported in production.)
spec.volume
spec.containers.volumeMounts
PersistentVolume defines & PersistentVolumeClaims the volume to attach to pods by master control. Claim means, pods looking for selected storage, else returns errors.

$ kubectl get persistentvolume

-----------------
# Volume Configuration Separate
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-1
  labels:
    type: local
spec: 
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes: 10Gi
    - ReadWriteOnce
  hostPath: "/mnt/data"
-------------------
# Volume Claim Section Inside Deployment
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec: 
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
-------------------
# deployment configuration

template:
  spec: 
    containers:
      volumeMounts:
      - name: wordpress-persistent-storage
        mountPath: /var/www/html
    volumes:
    - name: wordpress-persistent-storage
      persistentVolumeClaims:
        claimName: wp-pv-claim  

Section 5 Lecture 50
# Base64 encoded key value pairs to keep human readable data like password, keys to keep to use inside configuration files. Instead of human reabable, It will be a non-human readable hash.

kubectl create secret generic mysql-pass --from-literal=password=MY_PASSWORD
kubecat create secret generic mysql-pass --from-file=./username.txt --from-file=./password.txt

spec:
  containers:
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-pass
          key: password 

# Instead of using volume claim, using secret methodology to mound 
  volumeMounts: 
  - name: secret-storage
    mountPath: /etc/secretStore

volumes:
- name: secret-storage
  secret:
    secretName: mysql-pass

Section 5 Lecture 52 Usage and Resource Monitoring
Heap, InfluxDB, Grafana

- Volume Section 5 - 47 install wordpress with EBS.
- Connect AWS LoadBalancer.
- How to install multiple masters.
- Kops and Kubespray. Elad told to use to install.
- What is Base 64

Heapster: Container Cluster Monitoring maintained by CNCF

# Open in Git Copy Paste Code
# https://github.com/kubernetes/heapster/edit/master/deploy/kube-config/influxdb/heapster.yaml

# KubernetesDashboard Installation
# https://github.com/kubernetes/dashboard/wiki/Installation

There is info on our wiki pages that in order to be able to log in you need to access Dashboard over HTTPS. Frontend will not allow to log in if current domain does not use HTTPS.

As for the alternate version everything works as intended. Starting from K8S 1.7 (if I remember correctly) default permissions are very limited. You would need to grant Dashboard manually more permissions, i.e. by creating ClusterRoleBinding with cluster-admin ClusterRole for kubernetes-dashboard SA.
# Dashboard.

# Section 5 Lecrute 54 Namespace & Resource Quotas
# Name object of any type is possible for teams not to fight so namespaces are used.
# default is fine but example you can create namespace per team, organisations, staging, production.
# You can limit Compute, Storage, Memory, Number of Objects on NameSpace by ResourceQuotas

apiVersion: v1
metada:
  name: compute-resources
spec:
  hard:
    pods: "4"
    requests.cpu: "1"    # max to request per name space
    requests.memory: "1Gi" # max to request per name space
    limits.cpu: "2"  # cannot exceed somehow per name 
    limits.memory: 2Gi  # cannot exceed somehow

$ kubectl create namespace test_namespace
$ kubectl get namespace

# Create a NameSpace
$ kubectl create namespace cpu-limited-tomcat

$ vi cpu-limited-tomcat
#
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    limit.cpu: "400m"  # 400 %

# Apply ResourceQuota to Namespace
# kubectl apply -f ./tomcat-deployment --namespace=cpu-limited-tomcat

# Get NameSpaces
$ kubectl get namespace

# Get ResourceQuotas
$ kubectl get resourcequota --namespace=kube-system

# Create deployment on Namespace --namespace=
$
kubectl apply -f ./tomcat-deployment.yaml --namespace=cpu-limited-tomcat

# Describe deployment
kubectl describe deployment tomcat-deployment
spec.template.spec.containers:
  resources:
    requests:
      cpu: "200m"

# Kubernetes will not let to deploy or will apply the minimum available if you limit workspace with less quota than resource requested by deployment.

# In trouble shouting, it's important if 3 replicas cannot be created by limits, how many replicaes will create.


Section 5 - Lecture 56 - Auto Scaling
# HPA - Horizontal Pod Autoscaler will create or remove pods according to maintain avarage CPU utilization accross the pods.

# Example % 50 CPU utilization per pod avarage of % 50 per Pod.

$ kubectl autoscake deployment wordpress --cpu-percent=50 --min=1 --max=10

# Create a load generator for testing
$ kubectl run -i --tty load-generator --image=busybox /bin/sh
# Creating load
$ while true; do wget -q -O-http://wordpress.default.svc.cluster.local; done
# Get HPA
$ kubectl get hpa

# Autoscaling Commands
# since the latest minikube doesn't enable metrics-server by default
minikube addons enable metrics-server  
 
$ kubectl apply -f ./wordpress-deployment.yaml
$ kubectl autoscale deployment wordpress --cpu-percent=50 --min=1 --max=5

(In the other terminal)
$ kubectl run -i --tty load-generator --image=busybox /bin/sh
$ while true; do wget -q -O- http://wordpress.default.svc.cluster.local; done
(In the first terminal)
$ kubectl get hpa 

# Section 5 Lecture 58 Auditing.
# https://kubernetes.io/docs/tasks/debug-application-cluster/audit/
# Auditing means recording who does what, when and why provides security-relevant chronological set of records. ?
# 2 type of auditing Legacy and Advances
# Consider Advanced Auditing.
# kube-apiserver performs auditing according to Audit Policy and send records to the Audit Backend.
# It's defined on kube-apiserver by yaml file.
# Logs are in json format.
# Audit policy defines what should be logged and nearly anything can be logged.

Backend Types:
- Logs - Save to external storage.
- Webhooks - sends events to an external API/system.

$ vi audit-policy.yaml
apiVersion: audit.k8s.io/v1beta1
kind: Policy
rules:
- level: Metadata

# Enabling Auditing in Minicube
minikube start  --extra-config=apiserver.Authorization.Mode=RBAC --extra-config=apiserver.Audit.LogOptions.Path=/var/logs/audit.log   --extra-config=apiserver.Audit.PolicyFile=/etc/kubernetes/addons/audit-policy.yam

# Metadata Better View
# jq command to manipulate json data for better formatting.
$ cat audit.log | jq . 

# Section 5 Lecture Auto-Scaling and Resources
# Example install highly scalable ghost, create a persistent volume. 
 
# Section 6 Lecture 60 High Availability
OnMaster > kube-apiserver : kube-controller-manager : kube-scheduler + etcd (key value pair configuration)

# On production to provide HA:  
# - put etcd on redundant storage.
# - kube-api replicated load balanced.
# - master-elected kube-control-manager kube scheduler daemons.
# - Multiple worker nodes

# Tools: Kops to do this.
kubectl(clients)>Load Balancer> Master Nodes(etcd,apiserver,scheduler,controller manager, podmaster, kubelet, monit)

# Separate independent Linux machines and run kubelet and monit

# Steps for HA
# - Reliable Data Storage Layer and running etcd on each masted node.
# - replicated api servers behind load balancers
# - ensure only one actor works on the data at a given time, ech scheduler and controller will be started with a --leader-elect flag that will use lease-lock API between themselves to ensure one instance of each is running at a given time. 
# if one node fails, request will go to another.

# Section 6, Lecture 62 - Masters - Build a cluster on AWS
# https://github.com/kubernetes/kops/
kube-apiserver, kube-controller-manager, kube-scheduler relies on etcd. 
# Installation will be done by Kops
# Steps
# Get AWS key.
Access key ID	AKIAI7WAFOXS4FHVJMYQ
Secret access key	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
# - Install Kops on Linux
$ curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
$ chmod +x kops-linux-amd64
$ sudo mv kops-linux-amd64 /usr/local/bin/kops
# Install kubectl
$ wget -O kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
# - Install awscli 
# - Search documentation for aws command line tool suite if installing on AWS and ubuntu. 
# Python is dependency.
$ curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
$ sudo python3 ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
# - Configure for access, access keys and region
$ aws configure
# Create a bucket by awscli
# aws s3api create-bucket --bucket kube-lony-test --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1
# "Location": "http://kube-lony-test.s3.amazonaws.com/"
# Set Environment Variables for installation 
$ export KOPS_STATE_STORE=s3://kube-lony-test
# Create public private keys, cluster creation requires
$ ssh-keygen
# Create the cluster
$ kops create cluster kube-lony-cluster.k8s.local --zone eu-central-1b --yes
# Deleting Cluster
$ kops delete cluster kube-lony-cluster.k8s.local --yes
# Sugestions after installation, do all of them
# Suggestions:
# * Do valide till you see that cluster is ready.
# * validate cluster: kops validate cluster
# * list nodes: kubectl get nodes --show-labels
# * ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.kube-lony-cluster.k8s.local
# * This will not work, try as below.
# * ssh ssh -i ~/.ssh/id_rsa admin@ec2-35-157-217-201.eu-central-1.compute.amazonaws.com
$ sudo ps ax
$ cat /var/logs/...
$ docker ps
$ kubectl get deployments --all-namespaces
$ kubectl get deployments --namespace=kube-system
# * the admin user is specific to Debian. If not using Debian please use the appropriate user based on your OS.
# * read about installing addons at: https://github.com/kubernetes/kops/blob/master/docs/addons.md.

Section 6, Lecture 64 - HIGH AVAILABILITY
# https://medium.com/appvia/extending-your-kubernetes-cluster-into-a-new-aws-availability-zone-with-kops-127ead8b31bc

Best Practice:
- Creating odd number of masters, minimum 3 
- Creating the masters in different zones.
$ aws s3api create-bucket --bucket kube-lony-test --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1

$ export KOPS_STATE_STORE=s3://kube-lony-test

$ kops create cluster kube-lony-cluster.k8s.local --zone eu-central-1b,eu-central-1a,eu-central-1c --node-count 3 --master-zones eu-central-1b,eu-central-1a,eu-central-1c --yes

$ kubectl apply -f ./wordpress-deployment.yaml
$ kubectl apply -f ./mysql-deployment.yaml
$ kubectl get deployments
$ kubectl get pods
$ kubectl describe pods


# Section 6 Lecture 65 - Volumes on AWS
# - Auto provisioner for dynamicly provisioned volumes. You don't need to create volumes they are auto provisioned, you only create volume claim and it's handled automatically. 
# - Auto provisioning supported platforms. You can change default storage classes.
# https://kubernetes.io/blog/2017/03/dynamic-provisioning-and-storage-classes-kubernetes/

-----

- What is kubectl create --save-config -f tomcat-deploymeny.yaml  ?
- If Kubedns is not configured under /etc/resolv.conf, what we do ?
- Kops cluster with yaml and more details.











