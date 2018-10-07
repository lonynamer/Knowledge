
## INSTALLING KUBERNETES ON AWS BY KOPS/AWSCLI WITH/WITHOUT TERRAFORM

### Kops Instance Preparations (Install `awscli`, `kubectl`, `terraform` )  

##### Some good articles  
- https://github.com/kubernetes/kops/blob/master/docs/addons.md  
- https://medium.com/containermind/how-to-create-a-kubernetes-cluster-on-aws-in-few-minutes-89dda10354f4  
- https://kubernetes.io/docs/admin/authorization/rbac/  

##### Start

###### Security  
- Create an EC2 instance. It can be t2.micro very small  
- During the creation add an `IAM` role which includes `AdministratorAccess` `policy` for `EC2`  
- You can also dive deep and give only necessary rights for security hardenning.
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```
- The second way is creating user with the same IMA Roles and using. You can do this step after awscli installation.
```
aws configure
```
```
AWS Access Key ID [None]: AccessKeyValue
AWS Secret Access Key [None]: SecretAccessKeyValue
Default region name [None]: us-east-1
Default output format [None]:
```
- Also when you start to use kops, if you have authorization problems, export this variables.
```
export AWS_ACCESS_KEY=AccessKeyValue
export AWS_SECRET_KEY=SecretAccessKeyValue
```

###### ASW CLI
- Install AWS cli `aws`.  
```
sudo apt-get update
sudo apt-get -y install python3-pip
sudo python3 -m pip install --upgrade pip
sudo python3 -m pip install awscli
```

###### KUBECTL
- Install `kubectl` command binary.  
```
export KUBECTL_BIN=kubectl
 
function install_kubectl {
    if [ -z $(which $KUBECTL_BIN) ]
       then
           curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/$KUBECTL_BIN
           chmod +x ${KUBECTL_BIN}
           sudo mv ${KUBECTL_BIN} /usr/local/bin/${KUBECTL_BIN}
    else
       echo "Kubectl is most likely installed"
    fi
 
}
install_kubectl
```

###### KOPS
- Install `kops`  
```
function install_kops {
    if [ -z $(which kops) ]
       then
           curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
           chmod +x kops-linux-amd64
           sudo mv kops-linux-amd64 /usr/local/bin/kops
       else
           echo "kops is most likely installed"
       fi
}
 
install_kops
```

###### TERRAFORM
- Install `terraform`  
```
sudo apt-get update
sudo apt-get -y install unzip
export TERRAFORM_ZIP_FILE=terraform_0.11.7_linux_amd64.zip
export TERRAFORM=https://releases.hashicorp.com/terraform/0.11.7
export TERRAFORM_BIN=terraform
 
function install_terraform {
    if [ -z $(which $TERRAFORM_BIN) ]
       then
           wget ${TERRAFORM}/${TERRAFORM_ZIP_FILE}
           unzip ${TERRAFORM_ZIP_FILE}
           sudo mv ${TERRAFORM_BIN} /usr/local/bin/${TERRAFORM_BIN}
           ! [[ -z ${TERRAFORM_ZIP_FILE} ]] && rm -f ${TERRAFORM_ZIP_FILE}
    else
       echo "Terraform is most likely installed"
    fi
 
}
install_terraform
```

- Create a public,private rsa key.  
```
ssh keygen -t rsa
```


### Cluster Preparation Steps 
###### S3 Bucket
- Create and s3 bucket  
```
aws s3api create-bucket --bucket kops-lony --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1
```
Output:  
```
{
    "Location": "http://kops-lony.s3.amazonaws.com/"
}
```

### INSTALLING KUBERNETES BY KOPS ON AWS WITH/WITHOUT TERRAFORM
- A good tutorial. https://medium.com/containermind/how-to-create-a-kubernetes-cluster-on-aws-in-few-minutes-89dda10354f4
- In this cluster we will not use self `domainname` and `aws` loadbalancers are default.

### INSTALLING A CLUSTER BY ONLY KOPS
See `kops create cluster` flags and parameters.
```
kops create cluster --help
```

##### Basic
- Only Kops Create Cluster (Basic without too much selections and control)  
```
kops create cluster cluster.k8s.local --zones eu-central-1b --yes
```

##### With More Options And Control
- Variable that sets etcd state store database. 
- Necessary to set this variable always to work on a cluster.
```
export KOPS_STATE_STORE=s3://kops-lony
```
```
kops create cluster \
 --name=cluster.k8s.local \
 --state=s3://kops-lony \
 --authorization=RBAC \
 --zones=eu-central-1b \
 --master-zones=eu-central-1b \
 --networking=flannel \
 --master-count=2 \
 --master-count=1 \
 --master-size=t2.micro \
 --node-size=t2.micro \
 --master-volume-size=8 \
 --node-volume-size=8 \
 --ssh-public-key=~/.ssh/id_rsa.pub\
 --yes

# More Options
# --authorization=RBAC \                # Role Based Access Control (RBAC)
# --dns=private
# --dns-zone=k8s.local
# --topology=private
# --api-loadbalancer-type=internal \
# --image=ami-086a09d5b9fa35dc7 \       # If you choose different OS image, enable `net.ipv4.ip_forward=1` in /etc/sysctl.conf and check `firewall and security tools are now blocking.`
# --kubernetes-version=1.9.3 \
# --networking=calico \
# --networking=flannel \
# --network-cidr=10.253.0.0/16 \
# --out=kops_cluster_create_terraform \
# --target=terraform
```
- If you dont't use --yes flag. It will only create cluster `etcd` database in `s3` bucket for a stateful cluster.  
- Then you need to run `kops update` command to apply and create the cluster
```
kops update cluster cluster.k8s.local --yes
```
- Can edit the cluster later like this `vi` style or check the config file and update.  
```
kops edit cluster
kops update cluster cluster.k8s.local state s3://kops-lony --yes
```

##### POST INSTALLATION
- Suggestions after installation  
```
kops validate cluster                               # run validate command till you see the cluster and no errors and the next step.(`Your cluster cluster.k8s.local is ready.`)  
kubectl get nodes --show-labels                     # show labels.  
ssh -i ~/.ssh/id_rsa admin@api.cluster.k8s.local    # do ssh to api, if not working.  
kubectl get pods --all-namespaces                   # Check all pods in `kube-system` namespace is running.   
```

###### ADDONS
- Install addons https://github.com/kubernetes/kops/blob/master/docs/addons.md.  
- As an example install `Kubernetes Dashboard`.  
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
kops get secrets kube --type secret -oplaintext
kops get secrets admin --type secret -oplaintext
# Accessing dashboard has a bug that causes a problem, is not reachable from remote.
```

- Test your cluster with a simple `Deployment` and `Service` of `tomcat`.
```
# Deployment
kubectl run tomcat --image=tomcat --port=80
# OR
kubectl create deployment tomcat --image=tomcat 
# Service
kubectl expose deployment tomcat --port=80 --target-port=8080 --type=LoadBalancer
```
Output:
```
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP                                                                  PORT(S)          AGE
kubernetes   ClusterIP      100.64.0.1       <none>                                                                       443/TCP          51m
tomcat       LoadBalancer   100.66.108.111   a09693a68c87c11e8b935060e708c3e8-2002356580.eu-central-1.elb.amazonaws.com   80:31049/TCP   7m
```
- Browse to http://a09693a68c87c11e8b935060e708c3e8-2002356580.eu-central-1.elb.amazonaws.com  .  
- For each `Service` in type of `LoadBalancer`, a new loadbalancer will be created as, we built our cluster using `aws` loadbalancers.

###### DELETING THE CLUSTER
```
export KOPS_STATE_STORE=s3://kops-lony
kops delete cluster cluster.k8s.local
```
###### STOPPING THE CLUSTER
- `Kubernetes Cluster` created by `Kops`, is not designed to be stopped. Even you stop or terminate the instances, new ones will be created according to `Desired State` configured at `Launch Configurations` and `Auto Scaling Gropus` in `AWS` created by `Kops`.  
- So, We will do a trick and change the `Instance Groups` configurations of desired min and max number of nodes and masters.
```
kops get ig
```
Output:
```
Using cluster from kubectl context: cluster.k8s.local

NAME                    ROLE    MACHINETYPE     MIN     MAX     ZONES
master-eu-central-1b    Master  t2.medium       0       0       eu-central-1b
nodes                   Node    t2.medium       0       0       eu-central-1b
```
```
kops edit ig master-eu-central-1b
kops edit ig nodes
```
- Edit `vi` style and change `maxSize:` and `maxSize:` values to 0 and `:wq!` .  
- Update the configuration change. Instances will terminate.
- You will not use data, when you reverts back the settings the same way, cluster will come back.
```
kops update cluster cluster.k8s.local --yes
```

##### CLUSTER CREATION BY TERRAFORM AND KOPS
- Variable that sets etcd state store database. 
- Necessary to set this variable always to work on a cluster.
```
export KOPS_STATE_STORE=s3://kops-lony
```
- Kops command below will now create the cluster.  
- It will create `etcd` state store on aws `s3`. 
- Additionally a `terraform` directory named `kops_cluster_create_terraform` which will be our `IaC` source to install cluster.
- Additionally, if you delete the cluster, need to keep backup of `etcd` s3://kops-lony and keep with `terraform` file.
```
kops create cluster \
 --name=cluster.k8s.local \
 --state=s3://kops-lony \
 --authorization=RBAC \
 --zones=eu-central-1b \
 --master-zones=eu-central-1b \
 --networking=flannel \
 --master-count=2 \
 --master-count=1 \
 --master-size=t2.micro \
 --node-size=t2.micro \
 --master-volume-size=8 \
 --node-volume-size=8 \
 --ssh-public-key=~/.ssh/id_rsa.pub\
 --out=kops_cluster_create_terraform \
 --target=terraform

```
Terraform Apply  
```
cd kops_cluster_create_terraform
terraform init
terraform apply
```
###### Validate  
- Couldn't validate there is a DNS issue, need to convert hostname of `api` to FDQN for `kubectl` or `kops` as they cannot reach hostname becuase we didn't use our own domain name. Need to figure out the FDQN of `api` and set as default.


###### DELETE THE CLUSTER BY TERRAFORM
- Inside the `kops_cluster_create_terraform` directory.  
```
terraform destroy
```

### ENSURE LATER
```
Important settings after.
DNS Resolving
Fully qualified DNS of master ? In domainless installation.
How to set smaller etcd ebs volume on master less than 20 GB.
```

