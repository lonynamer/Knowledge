#### Kubernetes Advanced Usage  
Learn Kubernetes before diving into this subjects.  
- Logging : ElasticSearch(as StatefulSet(PetSet)), fluentd, Kibana, Logtrail.  
- Authentication : OpenID Connect(OIDC), Auth0  
- Authorization : Kubernetes RBAC  
- Packaging : Helm  
- The Job Resource : Job 
- Scheduling : Cronjob   
- Deploying on Kubernetes : Spinnaker  
- Microservices on Kubernetes : Linkerd  
- Federation : kubefed  
- Monitoring : Prometheus  

###### Objectives of this knowledge
- Using advanced features  
- Usage in enterprise environment  
- Apply security features  
- Setup a HA cluster  

#### LOGGING WITH FLUENTD AND ELK
- The primitive and hard way of getting logs.  
```
kubectl log pod-name
kubectl logs container-names
```
- The modern and easy way of getting logs in a such dynamic environment, is log aggregation.  
- Log aggregation was implemented since 1980s by syslog.  
- The modern way is using an ELK stack (Elasticsearch, logstash and Kibana).  
- There are some hosted service like;  
    - http://loggly.com
    - http://papertrailapp.com

#### HOW TO SETUP CENTRALIZED LOGGING ON KUBERNETES USING;
- Fluentd : Log forwarding.  
- ElasticSearch : For log indexing.  
- Kibana : For virtualization.  
- LogTrail : An easy to use UI to show logs.  
- By this solution, logging can be customized by creating custom dashboards to show what is important for you.  

###### STARTING (BY PREPARATIONS AND INSTALLING KUBERNETES)  
- I have `lonynamer.com` domain in `AWS`, I don't want touch this domain but create a sub-hosted domain for using in training.  
  - Create a hosted zone in `AWS` as `kops.lonynamer.com`  
  - Copy the NS records and add to `lonynamer.com` as `kops.lonynamer.com. IN NS` paste what you copied. This way you will have a sub-hosted zone to play with kubernetes.  
- Create an `S3` bucket to keep the `kubernetes cluster state file`.  
- Create an `IAM` group and `IAM` user (with programatic acess), add the user to this group and apply `AdminAcess` policy to this group for testing purposes, get access and secret keys.  
- Create a `t2.micro` `Ubuntu 16.04 X64` server for kubernetes cluster builder `kops`.  
- Install `awscli` on this machine also configure (access_key, secret_key, region) by;  
  - https://docs.aws.amazon.com/cli/latest/userguide/installing.html  
  - Install awscli  
```
sudo apt-get update
sudo apt-get -y install python3-pip
sudo python3 -m pip install --upgrade pip
sudo python3 -m pip install awscli
```
  - Configure awscli  
```
aws configure
```
- Install `kubectl` command  
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
- Install `kops`. Kops will build our kubernetes cluster.  
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
- Use default `ubuntu` user for testing purposes and create a default ssh key. Instance created by the installer `kops` will use that public key, so you can do ssh to those instances.  
```
ssh-keygen
```
- Create a small cluster by `kops`, in this scenario cluster is created in eu-cental-1 region's eu-central-1b zone.  
```
kops create cluster \
 --name=mycluster.kops.lonynamer.com \
 --state=s3://mycluster-state \
 --dns-zone=kops.lonynamer.com \
 --zones=eu-central-1b \
 --master-zones=eu-central-1b \
 --networking=flannel \
 --node-count=2 \
 --master-count=1 \
 --master-size=t2.micro \
 --node-size=t2.medium \
 --master-volume-size=8 \
 --node-volume-size=8 \
 --ssh-public-key=~/.ssh/id_rsa.pub\
 --yes
#--authorization=RBAC \   # is very important for security, disabled just here, will be explained later.  
```
- Validate that the cluster is built, wait and do a few times till the cluster is built.  
```
export KOPS_STATE_STORE=s3://mycluster-state
kops validate cluster
```
- Your cluster is ready and `kubectl` is configured to talk with API.  
- Deleting and creating the cluster too many times will hit S3 push pull limitations for free-tier.  
- Stop the instances when you don't use by bringing down the cluster.  
- When ever you would like to bring down this cluster but not to delete it to continue later.  
```
export KOPS_STATE_STORE = s3://mycluster-state
kops get ig
kops edit ig master-eu-central-1b
kops edit ig nodes
# Reduce for master and nodes MinSize and MaxSize to 0 and `wq!` like `vi` .
kops update cluster
# To bring up back set the same numbers before 1 master, 2 nodes
kops update cluster
```

###### SETUP CENTRALIZED LOGGING
- You need at least 7.5 - 8 GB memory for all this configuration.  


###### NOT FINISHED, JUST STARTED AND WILL BE CONTINUED !!!



