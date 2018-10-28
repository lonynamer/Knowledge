#### Kubernetes Advanced Usage  
Learn Kubernetes before diving into this 

subjects.  
- Logging : ElasticSearch(as StatefulSet

(PetSet)), fluentd, Kibana, Logtrail.  
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
- The modern and easy way of getting logs in a 

such dynamic environment, is log aggregation.  
- Log aggregation was implemented since 1980s by 

syslog.  
- The modern way is using an ELK stack 

(Elasticsearch, logstash and Kibana).  
- There are some hosted service like;  
    - http://loggly.com
    - http://papertrailapp.com

#### HOW TO SETUP CENTRALIZED LOGGING ON 

KUBERNETES USING;
- Fluentd : Log forwarding.  
- ElasticSearch : For log indexing.  
- Kibana : For virtualization.  
- LogTrail : An easy to use UI to show logs.  
- By this solution, logging can be customized by 

creating custom dashboards to show what is 

important for you.  

###### STARTING (BY PREPARATIONS AND INSTALLING 

KUBERNETES)  
- I have `lonynamer.com` domain in `AWS`, I don't 

want touch this domain but create a sub-hosted 

domain for using in training.  
  - Create a hosted zone in `AWS` as 

`kops.lonynamer.com`  
  - Copy the NS records and add to 

`lonynamer.com` as `kops.lonynamer.com. IN NS` 

paste what you copied. This way you will have a 

sub-hosted zone to play with kubernetes.  
- Create an `S3` bucket to keep the `kubernetes 

cluster state file`.  
- Create an `IAM` group and `IAM` user (with 

programatic acess), add the user to this group 

and apply `AdminAcess` policy to this group for 

testing purposes, get access and secret keys.  
- Create a `t2.micro` `Ubuntu 16.04 X64` server 

for kubernetes cluster builder `kops`.  
- Install `awscli` on this machine also configure 

(access_key, secret_key, region) by;  
  - 

https://docs.aws.amazon.com/cli/latest/userguide/

installing.html  
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
           curl -LO 

https://storage.googleapis.com/kubernetes-

release/release/$(curl -s 

https://storage.googleapis.com/kubernetes-

release/release/stable.txt)/bin/linux/amd64/

$KUBECTL_BIN
           chmod +x ${KUBECTL_BIN}
           sudo mv ${KUBECTL_BIN} 

/usr/local/bin/${KUBECTL_BIN}
    else
       echo "Kubectl is most likely installed"
    fi
 
}
install_kubectl
```
- Install `kops`. Kops will build our kubernetes 

cluster.  
```
function install_kops {
    if [ -z $(which kops) ]
       then
           curl -LO 

https://github.com/kubernetes/kops/releases/downl

oad/$(curl -s 

https://api.github.com/repos/kubernetes/kops/rele

ases/latest | grep tag_name | cut -d '"' -f 

4)/kops-linux-amd64
           chmod +x kops-linux-amd64
           sudo mv kops-linux-amd64 

/usr/local/bin/kops
       else
           echo "kops is most likely installed"
       fi
}
 
install_kops
```
- Use default `ubuntu` user for testing purposes 

and create a default ssh key. Instance created by 

the installer `kops` will use that public key, so 

you can do ssh to those instances.  
```
ssh-keygen
```
- Create a small cluster by `kops`, in this 

scenario cluster is created in eu-cental-1 

region's eu-central-1b zone.  
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
#--authorization=RBAC \   # is very important for 

security, disabled just here, will be explained 

later.  
```
- Validate that the cluster is built, wait and do 

a few times till the cluster is built.  
```
export KOPS_STATE_STORE=s3://mycluster-state
kops validate cluster
```
- Your cluster is ready and `kubectl` is 

configured to talk with API.  

###### SETUP CENTRALIZED LOGGING
- In this section we will install `Elastic 

Search` as backend Nosql db, `Fluentd` as log 

collector/parser, `Kibana` as dashboard. All for 

centralized logging.  
- You need at least 7.5 - 8 GB memory for all 

this configuration.   

- First create a directory for configurations. 

All configuration files in this directory.  
```
mkdir logging
cd logging
```

- Elastic Search : `RBAC authentication and 

authorization`, `ClusterRole`, 

`ClusterRoleBinding`, `StateFulSet of 2 

replicas`, also `VolumeReclaimation` configured 

within the statefull set.  
Config: es-statefulset.yaml
```
# RBAC authn and authz
apiVersion: v1
kind: ServiceAccount
metadata:
  name: elasticsearch-logging
  namespace: kube-system
  labels:
    k8s-app: elasticsearch-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: elasticsearch-logging
  labels:
    k8s-app: elasticsearch-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
rules:
- apiGroups:
  - ""
  resources:
  - "services"
  - "namespaces"
  - "endpoints"
  verbs:
  - "get"
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: kube-system
  name: elasticsearch-logging
  labels:
    k8s-app: elasticsearch-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
subjects:
- kind: ServiceAccount
  name: elasticsearch-logging
  namespace: kube-system
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: elasticsearch-logging
  apiGroup: ""
---
# Elasticsearch deployment itself
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: elasticsearch-logging
  namespace: kube-system
  labels:
    k8s-app: elasticsearch-logging
    version: v5.5.1
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  serviceName: elasticsearch-logging
  replicas: 2
  selector:
    matchLabels:
      k8s-app: elasticsearch-logging
      version: v5.5.1
  template:
    metadata:
      labels:
        k8s-app: elasticsearch-logging
        version: v5.5.1
        kubernetes.io/cluster-service: "true"
    spec:
      serviceAccountName: elasticsearch-logging
      containers:
      - image: gcr.io/google-

containers/elasticsearch:v5.5.1-1
        name: elasticsearch-logging
        resources:
          # need more cpu upon initialization, 

therefore burstable class
          limits:
            cpu: 1000m
            memory: 2.5Gi
          requests:
            memory: 2.5Gi
            cpu: 100m
        ports:
        - containerPort: 9200
          name: db
          protocol: TCP
        - containerPort: 9300
          name: transport
          protocol: TCP
        volumeMounts:
        - name: es-storage
          mountPath: /data
        env:
        - name: "NAMESPACE"
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: "ES_JAVA_OPTS"
          value: "-XX:-AssumeMP"
      # Elasticsearch requires vm.max_map_count 

to be at least 262144.
      # If your OS already sets up this number to 

a higher value, feel free
      # to remove this init container.
      initContainers:
      - image: alpine:3.6
        command: ["/sbin/sysctl", "-w", 

"vm.max_map_count=262144"]
        name: elasticsearch-logging-init
        securityContext:
          privileged: true
  volumeClaimTemplates:
  - metadata:
      name: es-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 8Gi
```

- StorageClass configuration to be used for 

`Elastic Search Pods` in `StateFulSet`. It should 

be in same AZ where the pods run. Pods will 

reclaim and get block device from this class.  
Config: storage.yaml
```
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  zone: eu-central-1b
```


- Elastic Search service configuration  
Config: es-service.yml  
```
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-logging
  namespace: kube-system
  labels:
    k8s-app: elasticsearch-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/name: "Elasticsearch"
spec:
  ports:
  - port: 9200
    protocol: TCP
    targetPort: db
  selector:
    k8s-app: elasticsearch-logging
```

- Fluentd `ServiceAccount`, `ClusterRole`, 

`ClusterRoleBinding`, `DaemonSet` for fluentd to 

run on each node.   
Config: fluentd-es-ds.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd-es
  namespace: kube-system
  labels:
    k8s-app: fluentd-es
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd-es
  labels:
    k8s-app: fluentd-es
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
rules:
- apiGroups:
  - ""
  resources:
  - "namespaces"
  - "pods"
  verbs:
  - "get"
  - "watch"
  - "list"
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd-es
  labels:
    k8s-app: fluentd-es
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
subjects:
- kind: ServiceAccount
  name: fluentd-es
  namespace: kube-system
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: fluentd-es
  apiGroup: ""
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd-es-v2.0.1
  namespace: kube-system
  labels:
    k8s-app: fluentd-es
    version: v2.0.1
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  template:
    metadata:
      labels:
        k8s-app: fluentd-es
        kubernetes.io/cluster-service: "true"
        version: v2.0.1
      # This annotation ensures that fluentd does 

not get evicted if the node
      # supports critical pod annotation based 

priority scheme.
      # Note that this does not guarantee 

admission on the nodes (#40573).
      annotations:
        scheduler.alpha.kubernetes.io/critical-

pod: ''
    spec:
      serviceAccountName: fluentd-es
      containers:
      - name: fluentd-es
        image: gcr.io/google-containers/fluentd-

elasticsearch:v2.0.1
        env:
        - name: FLUENTD_ARGS
          value: --no-supervisor -q
        resources:
          limits:
            memory: 500Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: libsystemddir
          mountPath: /host/lib
          readOnly: true
        - name: config-volume
          mountPath: /etc/fluent/config.d
      nodeSelector:
        beta.kubernetes.io/fluentd-ds-ready: 

"true"
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      # It is needed to copy systemd library to 

decompress journals
      - name: libsystemddir
        hostPath:
          path: /usr/lib64
      - name: config-volume
        configMap:
          name: fluentd-es-config-v0.1.0
```

- `ConfigMaP` for fluentd to parse logs
Config:  fluentd-es-configmap.yaml
```
kind: ConfigMap
apiVersion: v1
data:
  containers.input.conf: |-
    # This configuration file for Fluentd / td-

agent is used
    # to watch changes to Docker log files. The 

kubelet creates symlinks that
    # capture the pod name, namespace, container 

name & Docker container ID
    # to the docker logs for pods in the 

/var/log/containers directory on the host.
    # If running this fluentd configuration in a 

Docker container, the /var/log
    # directory should be mounted in the 

container.
    #
    # These logs are then submitted to 

Elasticsearch which assumes the
    # installation of the fluent-plugin-

elasticsearch & the
    # fluent-plugin-kubernetes_metadata_filter 

plugins.
    # See https://github.com/uken/fluent-plugin-

elasticsearch &
    # https://github.com/fabric8io/fluent-

plugin-kubernetes_metadata_filter for
    # more information about the plugins.
    #
    # Example
    # =======
    # A line in the Docker log file might look 

like this JSON:
    #
    # {"log":"2014/09/25 21:15:03 Got request 

with path wombat\n",
    #  "stream":"stderr",
    #   "time":"2014-09-25T21:15:03.499185026Z"}
    #
    # The time_format specification below makes 

sure we properly
    # parse the time format produced by Docker. 

This will be
    # submitted to Elasticsearch and should 

appear like:
    # $ curl 'http://elasticsearch-

logging:9200/_search?pretty'
    # ...
    # {
    #      "_index" : "logstash-2014.09.25",
    #      "_type" : "fluentd",
    #      "_id" : "VBrbor2QTuGpsQyTCdfzqA",
    #      "_score" : 1.0,
    #      "_source":{"log":"2014/09/25 22:45:50 

Got request with path wombat\n",
    #                 

"stream":"stderr","tag":"docker.container.all",
    #                 "@timestamp":"2014-09-

25T22:45:50+00:00"}
    #    },
    # ...
    #
    # The Kubernetes fluentd plugin is used to 

write the Kubernetes metadata to the log
    # record & add labels to the log record if 

properly configured. This enables users
    # to filter & search logs on any metadata.
    # For example a Docker container's logs might 

be in the directory:
    #
    #  

/var/lib/docker/containers/997599971ee6366d4a5920

d25b79286ad45ff37a74494f262e3bc98d909d0a7b
    #
    # and in the file:
    #
    #  

997599971ee6366d4a5920d25b79286ad45ff37a74494f262

e3bc98d909d0a7b-json.log
    #
    # where 997599971ee6... is the Docker ID of 

the running container.
    # The Kubernetes kubelet makes a symbolic 

link to this file on the host machine
    # in the /var/log/containers directory which 

includes the pod name and the Kubernetes
    # container name:
    #
    #    synthetic-logger-0.25lps-

pod_default_synth-lgr-

997599971ee6366d4a5920d25b79286ad45ff37a74494f262

e3bc98d909d0a7b.log
    #    ->
    #    

/var/lib/docker/containers/997599971ee6366d4a5920

d25b79286ad45ff37a74494f262e3bc98d909d0a7b/997599

971ee6366d4a5920d25b79286ad45ff37a74494f262e3bc98

d909d0a7b-json.log
    #
    # The /var/log directory on the host is 

mapped to the /var/log directory in the container
    # running this instance of Fluentd and we end 

up collecting the file:
    #
    #   /var/log/containers/synthetic-logger-

0.25lps-pod_default_synth-lgr-

997599971ee6366d4a5920d25b79286ad45ff37a74494f262

e3bc98d909d0a7b.log
    #
    # This results in the tag:
    #
    #  var.log.containers.synthetic-logger-

0.25lps-pod_default_synth-lgr-

997599971ee6366d4a5920d25b79286ad45ff37a74494f262

e3bc98d909d0a7b.log
    #
    # The Kubernetes fluentd plugin is used to 

extract the namespace, pod name & container name
    # which are added to the log message as a 

kubernetes field object & the Docker container ID
    # is also added under the docker field 

object.
    # The final tag is:
    #
    #   kubernetes.var.log.containers.synthetic-

logger-0.25lps-pod_default_synth-lgr-

997599971ee6366d4a5920d25b79286ad45ff37a74494f262

e3bc98d909d0a7b.log
    #
    # And the final log record look like:
    #
    # {
    #   "log":"2014/09/25 21:15:03 Got request 

with path wombat\n",
    #   "stream":"stderr",
    #   "time":"2014-09-25T21:15:03.499185026Z",
    #   "kubernetes": {
    #     "namespace": "default",
    #     "pod_name": "synthetic-logger-0.25lps-

pod",
    #     "container_name": "synth-lgr"
    #   },
    #   "docker": {
    #     "container_id": 

"997599971ee6366d4a5920d25b79286ad45ff37a74494f26

2e3bc98d909d0a7b"
    #   }
    # }
    #
    # This makes it easier for users to search 

for logs by pod name or by
    # the name of the Kubernetes container 

regardless of how many times the
    # Kubernetes pod has been restarted 

(resulting in a several Docker container IDs).
    # Example:
    # {"log":"[info:2016-02-16T16:04:05.930-

08:00] Some log text here

\n","stream":"stdout","time":"2016-02-

17T00:04:05.931087621Z"}
    <source>
      type tail
      path /var/log/containers/*.log
      pos_file /var/log/es-containers.log.pos
      time_format %Y-%m-%dT%H:%M:%S.%NZ
      tag kubernetes.*
      format json
      read_from_head true
    </source>
  system.input.conf: |-
    # Example:
    # 2015-12-21 23:17:22,066 [salt.state       

][INFO    ] Completed state [net.ipv4.ip_forward] 

at time 23:17:22.066081
    <source>
      type tail
      format /^(?<time>[^ ]* [^ ,]*)[^\[]*\[[^

\]]*\]\[(?<severity>[^ \]]*) *\] (?<message>.*)$/
      time_format %Y-%m-%d %H:%M:%S
      path /var/log/salt/minion
      pos_file /var/log/es-salt.pos
      tag salt
    </source>
    # Example:
    # Dec 21 23:17:22 gke-foo-1-1-4b5cbd14-node-

4eoj startupscript: Finished running startup 

script /var/run/google.startup.script
    <source>
      type tail
      format syslog
      path /var/log/startupscript.log
      pos_file /var/log/es-startupscript.log.pos
      tag startupscript
    </source>
    # Examples:
    # time="2016-02-04T06:51:03.053580605Z" 

level=info msg="GET /containers/json"
    # time="2016-02-04T07:53:57.505612354Z" 

level=error msg="HTTP Error" err="No such image: 

-f" statusCode=404
    <source>
      type tail
      format /^time="(?<time>[^)]*)" level=(?

<severity>[^ ]*) msg="(?<message>[^"]*)"( 

err="(?<error>[^"]*)")?( statusCode=

($<status_code>\d+))?/
      path /var/log/docker.log
      pos_file /var/log/es-docker.log.pos
      tag docker
    </source>
    # Example:
    # 2016/02/04 06:52:38 filePurge: successfully 

removed file 

/var/etcd/data/member/wal/00000000000006d0-

00000000010a23d1.wal
    <source>
      type tail
      # Not parsing this, because it doesn't have 

anything particularly useful to
      # parse out of it (like severities).
      format none
      path /var/log/etcd.log
      pos_file /var/log/es-etcd.log.pos
      tag etcd
    </source>
    # Multi-line parsing is required for all the 

kube logs because very large log
    # statements, such as those that include 

entire object bodies, get split into
    # multiple lines by glog.
    # Example:
    # I0204 07:32:30.020537    3368 

server.go:1048] POST /stats/container/: 

(13.972191ms) 200 [[Go-http-client/1.1] 

10.244.1.3:40537]
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/kubelet.log
      pos_file /var/log/es-kubelet.log.pos
      tag kubelet
    </source>
    # Example:
    # I1118 21:26:53.975789       6 

proxier.go:1096] Port "nodePort for kube-

system/default-http-backend:http" (:31429/tcp) 

was open before and is still needed
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/kube-proxy.log
      pos_file /var/log/es-kube-proxy.log.pos
      tag kube-proxy
    </source>
    # Example:
    # I0204 07:00:19.604280       5 

handlers.go:131] GET /api/v1/nodes: (1.624207ms) 

200 [[kube-controller-manager/v1.1.3 

(linux/amd64) kubernetes/6a81b50] 

127.0.0.1:38266]
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/kube-apiserver.log
      pos_file /var/log/es-kube-apiserver.log.pos
      tag kube-apiserver
    </source>
    # Example:
    # I0204 06:55:31.872680       5 

servicecontroller.go:277] LB already exists and 

doesn't need update for service kube-

system/kube-ui
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/kube-controller-manager.log
      pos_file /var/log/es-kube-controller-

manager.log.pos
      tag kube-controller-manager
    </source>
    # Example:
    # W0204 06:49:18.239674       7 

reflector.go:245] 

pkg/scheduler/factory/factory.go:193: watch of 

*api.Service ended with: 401: The event in 

requested index is outdated and cleared (the 

requested history has been cleared 

[2578313/2577886]) [2579312]
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/kube-scheduler.log
      pos_file /var/log/es-kube-scheduler.log.pos
      tag kube-scheduler
    </source>
    # Example:
    # I1104 10:36:20.242766       5 

rescheduler.go:73] Running Rescheduler
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/rescheduler.log
      pos_file /var/log/es-rescheduler.log.pos
      tag rescheduler
    </source>
    # Example:
    # I0603 15:31:05.793605       6 

cluster_manager.go:230] Reading config from path 

/etc/gce.conf
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/glbc.log
      pos_file /var/log/es-glbc.log.pos
      tag glbc
    </source>
    # Example:
    # I0603 15:31:05.793605       6 

cluster_manager.go:230] Reading config from path 

/etc/gce.conf
    <source>
      type tail
      format multiline
      multiline_flush_interval 5s
      format_firstline /^\w\d{4}/
      format1 /^(?<severity>\w)(?<time>\d{4} [^

\s]*)\s+(?<pid>\d+)\s+(?<source>[^ \]]+)\] (?

<message>.*)/
      time_format %m%d %H:%M:%S.%N
      path /var/log/cluster-autoscaler.log
      pos_file /var/log/es-cluster-

autoscaler.log.pos
      tag cluster-autoscaler
    </source>
    # Logs from systemd-journal for interesting 

services.
    <source>
      type systemd
      filters [{ "_SYSTEMD_UNIT": 

"docker.service" }]
      pos_file /var/log/gcp-journald-docker.pos
      read_from_head true
      tag docker
    </source>
    <source>
      type systemd
      filters [{ "_SYSTEMD_UNIT": 

"kubelet.service" }]
      pos_file /var/log/gcp-journald-kubelet.pos
      read_from_head true
      tag kubelet
    </source>
    <source>
      type systemd
      filters [{ "_SYSTEMD_UNIT": "node-problem-

detector.service" }]
      pos_file /var/log/gcp-journald-node-

problem-detector.pos
      read_from_head true
      tag node-problem-detector
    </source>
  forward.input.conf: |-
    # Takes the messages sent over TCP
    <source>
      type forward
    </source>
  monitoring.conf: |-
    # Prometheus Exporter Plugin
    # input plugin that exports metrics
    <source>
      @type prometheus
    </source>
    <source>
      @type monitor_agent
    </source>
    # input plugin that collects metrics from 

MonitorAgent
    <source>
      @type prometheus_monitor
      <labels>
        host ${hostname}
      </labels>
    </source>
    # input plugin that collects metrics for 

output plugin
    <source>
      @type prometheus_output_monitor
      <labels>
        host ${hostname}
      </labels>
    </source>
    # input plugin that collects metrics for 

in_tail plugin
    <source>
      @type prometheus_tail_monitor
      <labels>
        host ${hostname}
      </labels>
    </source>
  output.conf: |-
    # Enriches records with Kubernetes metadata
    <filter kubernetes.**>
      type kubernetes_metadata
    </filter>
    <match **>
       type elasticsearch
       log_level info
       include_tag_key true
       host elasticsearch-logging
       port 9200
       logstash_format true
       # Set the chunk limits.
       buffer_chunk_limit 2M
       buffer_queue_limit 8
       flush_interval 5s
       # Never wait longer than 5 minutes between 

retries.
       max_retry_wait 30
       # Disable the limit on the number of 

retries (retry forever).
       disable_retry_limit
       # Use multiple threads for processing.
       num_threads 2
    </match>
metadata:
  name: fluentd-es-config-v0.1.0
  namespace: kube-system
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
```

Config: `Kibana` Deployment
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: kibana-logging
  template:
    metadata:
      labels:
        k8s-app: kibana-logging
    spec:
      containers:
      - name: kibana-logging

        # official image without logtrail:
        # image: 

docker.elastic.co/kibana/kibana:5.5.1

        # image with logtrail
        image: wardviaene/kibana-logtrail:5.5.1
        resources:
          # need more cpu upon initialization, 

therefore burstable class
          limits:
            cpu: 1000m
            memory: 2.5Gi
          requests:
            cpu: 100m
            memory: 2.5Gi
        env:
          - name: ELASTICSEARCH_URL
            value: http://elasticsearch-

logging:9200
          # use this if you want to use proxy
          #- name: SERVER_BASEPATH
          #  value: 

/api/v1/proxy/namespaces/kube-

system/services/kibana-logging
          - name: XPACK_MONITORING_ENABLED
            value: "false"
          - name: XPACK_SECURITY_ENABLED
            value: "false"
        ports:
        - containerPort: 5601
          name: ui
          protocol: TCP
```

- `Kibana` service to access `Kibana` dashboard.  
Config:  kibana-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: kibana-logging
  namespace: kube-system
  labels:
    k8s-app: kibana-logging
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/name: "Kibana"
spec:
  ports:
  - port: 5601
    protocol: TCP
    targetPort: ui
  selector:
    k8s-app: kibana-logging
  type: LoadBalancer
```

- Label the nodes as fluentd-ds-ready=true so the 

fluentd pods will run on this labeled nodes.  
```
kubectl get nodes
for i in `kubectl get node |cut -d ' ' -f 1 |grep 

internal` ; do kubectl label nodes ${i} 

beta.kubernetes.io/fluentd-ds-ready=true ; done
```

- All our pods and objects in this system are 

located under `kube-system` namespace, ensure all 

the pods are running.  
```
kubectl get pods --namespace=kubesystem
```

- You can create private loadbalancer and access 

over vpn, or can edit security log to only your 

ip to access the kibana, you can add a CNAME in 

DNS to your loadbalancer. Need to think about 

this separately. Just for learning leave like 

this and continue.  

```
kubectl get service --namespace=kube-system
```
Output:  
```
NAME                    TYPE           CLUSTER-IP 

      EXTERNAL-IP                                 

                                 PORT(S)          

AGE
elasticsearch-logging   ClusterIP      

100.71.166.174   <none>                           

                                            

9200/TCP         3m
kibana-logging          LoadBalancer   

100.65.53.221    

a05ad585bda8911e89e5b0681892b51f-1278573830.eu-

central-1.elb.amazonaws.com   5601:31588/TCP   3m
kube-dns                ClusterIP      

100.64.0.10      <none>                           

                                            

53/UDP,53/TCP    2d
```

- Browse to 

http://a05ad585bda8911e89e5b0681892b51f-

1278573830.eu-central-1.elb.amazonaws.com:5601/ 

to `Kibana Dashboard`.  
- First time view, you will create an index. For 

kibana to run, you have to create at least one 

index. Index name or pattern should be logstash-* 

. Fluentd is replacement for logstash, even we 

don't use logstash, fluentd is logstash 

compatible and * will be replaced as timestamp.  
- Go to `Discovery`, you will see the ingested 

logs in `JSON` format.  
- Also, `LogTrail` plugin is installed 

additionally to default. You can do search by 

pods and see all logs centrally.     

- All this is just a good start, need to dig into 

elastic search, how to use it in production and 

logging parameters. All is very customizeable. 
Example search experssion:  
```
kubernetes.container_name.keyword:"elasticsearch-logging"
``` 



###### NOT FINISHED, JUST STARTED AND WILL BE 

CONTINUED.....









QUESTIONS: VOLUME CLAIM TEMPLATES.  difference of 

volumeclaims.  
- where the configurations are taken from.  
- how to configure fluentd logging. find links
- how to configure elastic search autoscalable. find links. 


