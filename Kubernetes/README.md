## Kubernetes Knowledge

### What is Kubernetes(k8s) ?  
- Kubernetes is an open-source, self-healing, auto-scaling containerized workloads and micro-services orchestration solution. 
- It's mainly use for orchestrat `docker` and `rkt` containers.  
- Vertically, horrizontaly, infra-horrizontaly scalability.
- It's portable, extensible with large, rapidly growing ecosystem. Services, support and tools are widely awailable.  
- It orchestrates computing, networking, storage environment and this provides simplicity of IaaS with the fexibiliy of PaaS.  
- Enables portability accross infrastructure providers.  
- Kubernetes operates at container level and not a complete/traditional Paas system. 
- It provides some generally applicable features common to `Paas`, such as `deployment`, `scaling`, `load balancing`, `logging`, and `monitoring`.  
- Kubernetes facilitates by both `Declerative Configuration`, `Imperative Configuration` and `Automation`. 
- It has many installers and solutions for `bare-metal`, `gcp`, `aws`, `azzure` and many cloud providers.

### Kubernetes Concepts and Components
- We use `API objects` by `CLI` to describe desired state of our cluster like application workloads to run, container image, replicas, network, disk resources.  
- Both Masters and Nodes behind a loadbalancer.  
- Once we set our desired state, `Kubernetes Control Plane` matches/converts the current state to our desired state.  

##### kubectl, Kubernetes Master, Kubernetes Nodes(Minions, Non Masters) And Their Jobs
- In a simple installation, it consists of `kubectl`, `Master Node` and `Nodes`.  
- You can install this nodes on `bare-metal`, `aws` instances, `vms` etc.
###### kubectl
    - `kubectl`: command and cli interacts with master node by `RestAPI` to manage the cluster.
###### Master Components
    - `etcd` : highly available key=value storage for all cluster data.  
    - `kube-apiserver` : Frontend of k8s, validates and configures data for api objects.  
                         Horizontaly scalable by adding more instances.  
    
    - `kube-scheduler` : Take the pods that aren't bound to a node by `hardware/software/policy constrains.`  
    - `kube-control-manager` : 
       - `Node Controller` : Noticing and responding when nodes go down.   
       - `Replication Controller`: Maintaining correct number of pods for every object in the system.  
       - `Endpoint Controller` : Joins `Services` and `Pods`.  
       - Service account & Token Controllers: Create default accounts and API access tokes for new namespaces.  
     - `cloud-control-manager` : Following controllers have `cloud provider` dependencies 
                                 and interacts with underlying cloud provider.  
       - `Node Controller` : Check if node is deleted after it stops responding.  
       - `Route Controller` : Setting up routes in the underlying cloud infra.  
       - `Service Controller` : Creating, updating, deleting cloud provider load balancers.  
       - `Volume Controller` : Creating, attaching, mounting volumes and interacting with cloud provider 
                               to orchestrate volumes.  

###### Nodes(Minions)
    - `kubelet` : communicater with K8s Master.  
    - `kube-proxy` : A network proxy which reflects networking services on each node.  
```
# Component communication diagram
APIs(scheduling actuator >>> REST)
scheduler >>> (REST and scheduling) 
kubectl >>> authorication >>> REST
control manager >>> REST
REST >>> kubelet on Nodes >>> Pods >>> Docker
REST >>> etcd on Masters
```

### APPLICATION TYPES
- Stateful: Pods or containers are stateless.  
- Statefull: When you attach volume, it can be stateful.  

### Objects
- Kubernetes has `abstractions` that represents the state of your deployed containerized applications and workloads, their network, volumes and other resources.

###### Pod
- Pod describes the smallest unit, represents a running proccess in a container.  
- Pod covers/holds a container. It can hold more than one containers.  
- A pod in general is only for one process/container but you can add   
  additional containers example for monitoring, logging purposes.  
- `Pods` are `ephemeral` when crashes/stops, a static desired/described clean image will be brought up. Files created during runtime will be lost. `volumes` solves this problem.

###### Service
- Service describes the connectivity of the `Pod`.
- Service covers the pod. 
- Types:  
  - ClusterIP: Exposes a deployment internally as a cluster to other deployments by `egress` loadbalancer.  
  - NodePort: Exposes a deployment to a port between 30000-32767 on each node. Example apache runs on 80 on container and exposed as 32323 on each node. You can access the service out of the cluster from each node.  
  - LoadBalancer: Exposes a deployment by an external load balancer of a cloud provider.  
  - ExternalName: Maps the service to a `CNAME`.  
  
###### Volume
- `Pods` are ephemeral and need to share files between `containers`
- Volumes are attached to pod and orchastrated by `Kubernetes`  
- awsEBS, azzureDisk, azzureFile, cephfs, configMap, csi, downwardAPI, emptyDir, FC(fiber), flocker, gcePersistentDisk, gitRepo(deprecated), glusterfs,hostPath, iscsi, local, nfs, persistentVolumeClaim, projected, portworxVolume, quobyte, rbd, scaleIO, secret, storageos, vsphereVolume are solutions supported.
###### Namespace
- Namespaces may be used for isolation of resourcess across users and teams, unique resource allocation.  
- You can put pods to desired namespaces.  
- In some cases, it is used for security, communication limitations.  
- It's like virtual clustering of deployments.
- Default Namespaces:
  - deault: Our default for our deployments.
  - kube-system: Namespace for kubernetes system.
  - kube-public: Is readable by all users without authentication. User for cluster usage information etc. Not a requirement.
- You can add your own namespaces and it's a good practice.

- High level abstractions called Controllers:
###### ReplicaSet
- Describes number of minimum, maximum numbers of Pods of a deployment.

###### Deployment
- Deployment covers the pods, replicasets and describes a complete deployment.  
- Describes pods, containers, containerport, replicas, attached volume, everything related aorund.  
- Provides declarative updates for Pods, ReplicaSets.  

###### StatefulSet
- Is like a deployment but provides sticky idendity of pods and containers, they are not interchanable accross any rescheduling.  
- This is required for deployments like `mongodb` cluster, `mysql` cluster where interoperability occurs between pods.  
- StatefullSets are required one or more of the followings.  
  - Stable, unique network identifiers.  
  - Stable, persistent storage.  
  - Ordered, graceful deployment and scaling.  
  - Ordered, graceful deletion and termination.  
  - Ordered, automated rolling updates.  
  - Deployment and Scaling Guarantees.  
     - Pods are created in a form of sequentially numbers and terminated reverse order.  
     - Before a scaling operation is running all of it's precedecessors must be running and be ready.  
     - Before a pod is terminated, all of it's successors must be completely shutdown.  

###### DaemonSet
- Ensures all (or some) nodes run a copy of a Pod. As new nodes are added and removed from the cluster.  
- Exammples: glusterd, ceph storage daemon on each node. `fluentd`, logstash log collection daemons. `Prometheus Node Exporter`, `collectd`, `Datadog` agent, New Relic agent, Ganglia `gmond` .  

###### Job
- Describes a specified number of jobs runs on specified numbers of pods. When specified number of jobs finishes on a specified number of pods, ensures that pods are terminated.  

- Mode objects

###### ConfigMap Volume
- A way to inject configuration data into pods.
- Stores alias mapping for the parameters to keep the conteineried applications portable.

###### Secret Volume
- Is used to pass sensitive information such application passwords to Pods. You can store secrets in Kubernetes API and mount them as files to use by Pods without coupling.
- tmpfs RAM backed filesystem and newer written to a non-volatile storage. 

###### Replication Controller
- Is described under deployments. Manages number of Pods that should run. Adds more if necessary more, remove it the number is too much.

- PersistentVolumeClaim, StorageClass, PersistentVolume works together.
###### StorageClass
- Describes a storage solution of a cloud provider.

###### PersistentVolumeClaim
- Describes the volume and provisions at cloud provider according to request.
- It can be defined as a request to create the necessary volume.
- In other words, it's an auto provisioner.  
- https://kubernetes.io/blog/2017/03/dynamic-provisioning-and-storage-classes-kubernetes/  

###### PersistentVolume
- Describes the volume to attach to the Pod/Pods

###### LABELS AND SELECTORS

###### READINESS PROBE

###### LIVENESS PROBE

###### UPDATES, ROLLOUT UPDATES

###### HORIZONTAL POD AUTOSCALER (hpa)

- Some Other Stuff
###### Auditing.
- https://kubernetes.io/docs/tasks/debug-application-cluster/audit/  
- Auditing means recording who does what, when and why provides security-relevant chronological set of records.  
- 2 type of auditing Legacy and Advanced  
- Consider Advanced Auditing.  
- kube-apiserver performs auditing according to Audit Policy and send records to the Audit Backend.  
- It's defined on kube-apiserver by yaml file.  
- Logs are in json format.  
- Audit policy defines what should be logged and nearly anything can be logged.  
- Backend Types:  
  - Logs - Save to external storage.  
  - Webhooks - sends events to an external API/system.  

###### Monitoring
You can install tools like Heap, InfluxDB, Grafana, Prometeus for monitoring. 
- Heapster: Container Cluster Monitoring maintained by CNCF. 
- Prometeus is a good solution for monitoring. 

### Kubernetes General Documentation & References
---
- Kubernetes Docs
https://kubernetes.io/docs/
---
### CHEAT SHEAT REFERENCE
- https://kubernetes.io/docs/reference/kubectl/userguide/  
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/  
- https://kubernetes.io/docs/concepts/services-networking/service/  
- https://kubernetes.io/docs/concepts/architecture/master-node-communication/  
- https://stackoverflow.com/questions/39293441/needed-ports-for-kubernetes-cluster

#### INSTALLATION OPTIONS
- https://kubernetes.io/docs/setup/pick-right-solution/  
- INSTALLATION BY KOPS  
  https://github.com/lonynamer/Knowledge/tree/master/Kops-Terraform-Awscli-Kubernetes-Installation  
  https://github.com/kubernetes/kops/
- INSTALLATION BY RANCHER  
  https://rancher.com/docs/rancher/v2.x/en/  
- INSTALLATION BY KUBESPRAY  
  https://github.com/kubernetes-incubator/kubespray  
- INSTALLATION BY CONJURE AND JUJU  
  https://kubernetes.io/docs/getting-started-guides/ubuntu/  
- INSTALLATION BY KUBEADM  
  https://kubernetes.io/docs/setup/independent/install-kubeadm/  
- INSTALLATION MANUALLY ON BAREMETAL  
  https://severalnines.com/blog/installing-kubernetes-cluster-minions-centos7-manage-pods-services  
  
### High Availability For Deployments Best Practices
OnMaster > kube-apiserver : kube-controller-manager : kube-scheduler + etcd (key value pair configuration)
###### On production to provide HA:  
- put etcd on redundant storage.
- kube-api replicated load balanced.
- master-elected kube-control-manager kube scheduler daemons.
- Multiple worker nodes
- Separate independent Linux machines and run kubelet and monit
###### Steps for HA
- Reliable Data Storage Layer and running etcd on each masted node.
- Replicated api servers behind load balancers
- Ensure only one actor works on the data at a given time, each scheduler and controller will be started with a --leader-elect flag that will use lease-lock API between themselves to ensure one instance of each is running at a given time. 
- If one node fails, request will go to another.
- You can create the cluster by Kops easily.
  
### HIGH AVAILABILITY FOR MASTERS AND INSTALLATION BY KOPS  
- https://medium.com/appvia/extending-your-kubernetes-cluster-into-a-new-aws-availability-zone-with-kops-127ead8b31bc  
- If master fails, you cannot manage your cluster.  
- Best Practice:  
  - Creating odd number of masters, minimum 3.  
  - Creating the masters in different zones.  
```
aws s3api create-bucket --bucket kube-lony-test --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1
export KOPS_STATE_STORE=s3://kube-lony-test
kops create cluster kube-lony-cluster.k8s.local --zone eu-central-1b,eu-central-1a,eu-central-1c --node-count 3 --master-zones eu-central-1b,eu-central-1a,eu-central-1c --yes
# Connect to master
ssh -i ~/.ssh/id_rsa admin@ec2-35-157-217-201.eu-central-1.compute.amazonaws.com
sudo ps ax
cat /var/logs/...
docker ps
kubectl get deployments --all-namespaces
kubectl get deployments --namespace=kube-system
```

### KUBERNETES CODE GENERATOR
https://github.com/kubernetes/code-generator
###

###
### Clients
---
- `kubectl` is the client commands tool.
- `Kitematic` is a visual docker client.
---
### Servers
---
- `Minikube` is a standalone single server type of Kubernetes. Can be used for studying purposes.
   The things runs the same ways like on a complex Kubernetes infrastructure.  
- On windows and old MAC `docker toolbox` and it runs inside `virtual box`.  
- `Kops` is a good installer and orchastrator for installing and managing a `Kubernetes` cluster on `AWS` and other platforms/cloud-providers.  Which is adviced way to install and learn.

### ADDONS
   https://github.com/kubernetes/kops/blob/master/docs/addons.md.

### TECHNICAL, COMMANDS AND YAML DESCRIPTION FOR IaC
Some Important Commands To Know After Kubectl
- `get` shows 
- `describe` describes in json format  
- `expose` publishes outside.  
- `exec` runs a command in a pod, like you can run bash/sh and connect to console. -it (interactive and tty)  
- `attach` attachs a container like, you have a backgrounded bash, attach is used to attach.  
- `label` assings key:value pairs, some key value pairs like app, tier are used to attach services/components together.  
   Like Deployment, Service, Pod, Container, Storage. As labels and selectors.  
- `run` runs.  
- `create` creates.  

###### SOME COMMANDS
```
kubectl --version
kubectl cluster-info
kubectl get nodes
kubectl get pods
kubectl get services
kubectl get rs
kubectl describe rs
kubectl describe services
kubectl get deployments
kubectl describe deployments
kubectl logs pod `podname`
kubectl describe pod `podname`
kubectl delete pod `podname`
kubectl --namespace=<insert-namespace-name-here> run nginx --image=nginx
kubectl --namespace=<insert-namespace-name-here> get pods
kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
# Validate namespace
kubectl config view | grep namespace:
kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
kubectl get pod
kubectl describe pod
kubectl get deployment
kubectl delete deployment hello-minikube
kubectl run `deployment name` --image=`image-to-install` --port=`port-to-open-internally`
kubectl expose deployment hello-minikube --type=NodePort
kubectl expose deployment hello-minikube --type=LoadBalancer
kubectl get service
kubectl get pods
kubectl get pods [pod name] 
kubectl expose <type name> <identifier/name> [—port=external port] [—target-port=container-port [—type=service-type]
kubectl expose deployment tomcat-deployment --type=NodePort 
kubectl port-forward <pod name> [LOCAL_PORT:]REMOTE_PORT] 
kubectl attach <pod name> -c <container>  
kubectl exec  [-it] <pod name> [-c CONTAINER] — COMMAND [args…]
kubectl exec -it <pod name> bash  
kubectl label [—overwrite] <type> KEY_1=VAL_1 ….
#Apply a label
kubectl label node ip-172-31-42-57 storageType=ssd
kubectl label pods <pod name> healthy=fasle  
kubectl run <name> —image=image
kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
kubectl describe pod
#Port Forwarding for a single pod
kubectl port-forward tomcat-deployment-794dbf9966-7lmtq 5000:6000

#Run a command on a Pod
kubectl exec -it tomcat-deployment-794dbf9966-7lmtq bash

#To list the containers of a pod
kubectl get --all-namespaces --selector k8s-app=kube-dns --output json pods | jq --raw-output '.items[].spec.containers[].name'

#To see all details regarding one specific container try something like this:
kubectl get --all-namespaces --selector k8s-app=kube-dns --output json pods | jq '.items[].spec.containers[] | select(.name=="etcd")'

#Attach console of a container
kubectl attach tomcat-deployment-794dbf9966-7lmtq -c <container>

#Label a Pod
kubectl label pod tomcat-deployment-794dbf9966-7lmtq healthy=false

#Running an image
kubernetes-demo$ kubectl run hazelcast --image=hazelcast --port=5701

#SCALING
kubectl scale --replicas=4 deployment/tomcat-deployment
#Create Service with Loadbalancer
kubectl expose deployment tomcat-deployment --type=LoadBalancer --port=8080 --target-port=8080 --name tomcat-load-balancer
kubectl run mongo-exercise-1 --image=mongo --port 27017
kubectl scale --replicas=4 deployment/mongo-exercise-1

kubectl get persistentvolume

#Update deployment
## CREATE, UPDATE, APPLY ROLLING UPDATES, ROLLBACK, PAUSE & RESUME ALL POSSIBLE
kubectl get deployments
kubectl rollout status deployment tomcat-deployment
kubectl set image deployment/tomcat-deployment tomcat=tomcat:9.0.1
kubectl roleout history deployment/tomcat-deployment

##Get rollout history after update
kubectl rollout history deployment/tomcat-deployment --revision=2
```

###### DEPLOYMENT/DESCRIPTION BY YAML FILE
- Pods, Services, Deployments, Networks, Storages, Kubernetes Addons, Namespace etc., all can be described, installed and changed by yaml file like this.  
```
kubectl apply -f ./deployment.yaml
```
###Yaml File Example
```
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
```

### HEALT CHECK
- READINESS(If it came up)
- LIVENESS(If it's working) 
#### spec: > template: > spec: > container:
```
spec:
  template:
    spec:
      container:
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
```
### KUBE WEB UI > Runs on Master
#### You can do the things in KUBE WEB UI as well
#### Documentation
---
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
---
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
#### ACCESSING WEB UI
---
https://<master-ip>:<apiserver-port>/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
---

You can also install using deployment and service yaml or helm package manager.

### DNS & SERVICE DISCOVERY
Used for decoupling but working together together as independent modules
#### BUILD IN DNS
#### Kubernetes configures kubelets to provide DNS IP to containers to solve DNS Names of each other.
`<my-srv-name>.<my-name-space>.svc.cluster.local`








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
```
```
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
```

### Base64 encoded key value pairs
To keep human readable data like password, keys to keep to use inside configuration files. Instead of human reabable, It will be a non-human readable hash.
```
kubectl create secret generic mysql-pass --from-literal=password=MY_PASSWORD
kubecat create secret generic mysql-pass --from-file=./username.txt --from-file=./password.txt
```
```
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
```


### Namespace & Resource Quotas
Name object of any type is possible for teams not to fight so namespaces are used.
Default is fine but example you can create namespace per team, groups under organisations, staging, production.
 You can limit Compute, Storage, Memory, Number of Objects on NameSpace by ResourceQuotas
```
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
```
```
$ kubectl create namespace test_namespace
$ kubectl get namespace
```
#### Create a NameSpace
```
kubectl create namespace cpu-limited-tomcat
```
```
vi cpu-limited-tomcat
```
```
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
```
- Kubernetes will not let to deploy or will apply the minimum available if you limit workspace with less quota than resource requested by deployment.
- In trouble shouting, it's important if 3 replicas cannot be created by limits, how many replicaes will create.


### Auto Scaling
- HPA - Horizontal Pod Autoscaler will create or remove pods according to maintain avarage CPU utilization accross the pods.
- Example % 50 CPU utilization per pod avarage of % 50 per Pod.
```
$ kubectl autoscake deployment wordpress --cpu-percent=50 --min=1 --max=10
```
Create a load generator for testing
```
$ kubectl run -i --tty load-generator --image=busybox /bin/sh
```
Creating load
```
while true; do wget -q -O-http://wordpress.default.svc.cluster.local; done
```
Get HPA
```
$ kubectl get hpa
```
#### Autoscaling Commands
Since the latest minikube doesn't enable metrics-server by default
minikube addons enable metrics-server  
```
$ kubectl apply -f ./wordpress-deployment.yaml
```
```
$ kubectl autoscale deployment wordpress --cpu-percent=50 --min=1 --max=5
```
(In the other terminal)
```
kubectl run -i --tty load-generator --image=busybox /bin/sh
while true; do wget -q -O- http://wordpress.default.svc.cluster.local; done
```
(In the first terminal)
```
$ kubectl get hpa 
```



### Auditing
```
$ vi audit-policy.yaml
apiVersion: audit.k8s.io/v1beta1
kind: Policy
rules:
- level: Metadata
```

 




   








