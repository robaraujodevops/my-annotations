# Kubernetes

## Pod

To create pods:

`kubectl create -f pod-definition.yml`

or easy way

`kubectl run nginx --image=nginx`

or generate yml

`kubectl run nginx --image=nginx --dry-run=client -o yml > nginx-pod.yml` ( create file but do not create pod )

To get pods:

`kubectl get pods`

To get details from pod:

`kubectl describe pods { name_from_pod }`

To update pod state:

`kubectl apply -f pod-definition.yml`

`kubectl edit pod redis` ( will literaly edit on runtime )

### Pod Definition
```
apiVersion: v1
kind: Pod
metadata:
  name:
  labels:
    app:
    type:
spec:
  containers:
  - name: nginx-container
    image: nginx
```

## Replication Controller:
To create replication controller:

`kubectl create -f controller-definition`

To get replication controllers

`kubectl get replicationcontroller`

## Replication Controller Definition:
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name:
      labels: my-app-pod
        app: my-app
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
    replicas: 3
```

## Replica Set:

To create replicaset:
`kubectl create -f replicaset-definition.yml`

To get replicatsets:

`kubectl get replicaset`

To scale replicaset:

* edit replicaset definition and run: 

`kubectl replace -f replicaset-definition.yml`

* or

`kubectl scale --replicas=6 -f replicaset-definition.yml`

* or

`kubectl scale --replicas=6 replicaset my-app-rs` 
( will not edit definition file )

To delete replicasets ( will delete undelying pods ):
`kubectl delete replicaset my-app-rs`

### Replica set Definition: 

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: my-app-rs
 labels:
  app: myapp
  type: front-end
spec:
 template:
  metadata:
   name:
   labels: my-app-pod
    app: my-app
    type: front-end
  spec:
   containers:
    - name: nginx-container
      image: nginx
 replicas: 3
 selector:
  matchLabels:
   type: front-end
```

## Deployment

> (already creates replicaset and pods)

To create deployment:

`kubectl create -f deployment-definition.yml`

* or easy way:

`kubectl create deployment --image=nginx nginx`

* or generate yml

`kubectl create deployment --image=nginx --dry-run=client -o yml > nginx-deployment.yml`
> ( create file but do not create pod )

To get deployments:

`kubectl get deployments`

Deployment Definition:
```
apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp-deployment
 labels:
    app: myapp
    type: front-end
spec:
 template:
  metadata:
   name: myapp-pod
   labels:
    app: myapp
    type: front-end
  spec:
   containers:
   - name: nginx-container
     image: nginx
replicas: 3
selector:
 matchLabels:
  type: front-end
```

## Namespaces:

url service: db-service.dev.svc.cluster.local
  
To get pods by namespace, if not, always return default namespace:

`kubectl get pods --namespace=kube-system or --all-namespace(for all)`

To create a pod in specific namespace:

`kubectl create -f pod-definition.yml --namespace=dev`

To create a namespace

`kubectl create -f namespace-definition.yml`

* or

`kubectl create namespace dev`

To config default namespace based in the context:

`kubectl config set-context $(kubectl config current-context) --namespace=dev`

Namespace Definition:

* At resource
```
apiVersion: v1
kind: Pod
metadata:
 name: my-app
 labels: 
  app: my
  type: front
 namespace: dev
spec:
 containers:
  - name: nginx-container
    image: nginx
```

* At namespace
```
apiVersion: v1
kind: Namespace
metadata:
 name: dev
```
  
## Resourcequota:

To define quota for a namespace

`kubectl create -f compute-quota.yml`

Resource quota definition:

```
apiVersion: v1
 kind: ResourceQuota
 metadata:
  name: app-quota
  namespace: dev
 spec:
  hard:
   pods: “10”
   requests.cpu: “4”
   requests.memory: 5Gi
   limits.cpu: “10”
   limits.memory: 10Gi
```

## Services - Node Port:

Can choose an port between 30000 and 32767 range

To create a service:

`kubectl create -f service-definition.yml`

To get services

`kubectl get services`

Services Node Port Definition:
```
apiVersion: v1
kind: Service
metadata:
 name: my-app-service

spec:
 type: NodePort
 ports:
  - targetPort: 80
  port: 80
  nodePort: 300008
 selector:
  app: myapp
  type: front-end
```

## Services - Cluster IP

To create a cluster ip service:

`kubectl create -f service-definition.yml`

To get services

`kubectl get services`
 
Services Cluster IP definition:
```
apiVersion: v1
kind: Service
metadata:
 name: back-end

spec:
 type: ClusterIP
 ports:
  - targetPort: 80
    port: 80
 selector:
   app: myapp
  type: back-end
```

## Services - Load Balancer
Just works with cloud providers that supports this service

## Apply - Update all the shit
`kubectl apply -f nginx.yml`

## Labels, Selectors and Annotations:

To get a pod by label:

`kubectl get pods --selector app=App1`

> Its necessary to group resources
Definition:
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: simple-webapp
 labels:
  app: App1
  function: Front-End
 annotations:
  buildversion: 1.34
spec:
 replicas: 3
 selector:
  matchLabels:
   app: App1
 template:
  metadata:
   labels:
    app: App1
    function: Front-End
   spec:
    containers:
      - name: simple-webapp
        image: simple-webapp
```

## Taint and tolerations

Used to config the conditions to a node accept a pod

To config a taint to a node:

`kubectl taint nodes node01 key=value:taint-effect*`

> *taint-effect could be: NoSchedule|PreferNoSchedule|NoExecute )

To config a toleration to a pod ( doing this way the pod will be placed in a node that accepts its tolerations):

pod-definition.yml
```
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
   - name: nginx-controller
     image: nginx
 tolerations:
   - key: “app”
     operator: “Equal”
     value: “blue”
     effect: “NoSchedule”
```
 
## Node Selector:
 
Can define in a pod a node selector to put the container:
  
pod-definition.yml
```
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 nodeSelector:
   size: Large
```

To use this in a POD we need to label nodes:

`kubectl label nodes node-1 size=Large`

## Node Affinity

Node affinity types:

Available

> requiredDuringSchedulingIgnoredDuringExecution

> preferredDuringSchedulingIgnoredDuringExecution
 
Planned

> requiredDuringSchedulingRequiredDuringExecution

Can define in a pod a node selector with more especification in the query:

pod-definition.yml

```
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 affinity
    nodeAffinity:
     requireDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
     - matchExpressions:
       - key: size
         operator: In
         values:
            - Large
           size: Large
```

##Resources and limits

pod-definition.yml

```
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
resources:
 requests; 
  memory: “1Gi”
  cpu: 1
 limits;
  memory: “2Gi”
  cpu: 2
```

Obs: For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
   memory: 512Mi
    defaultRequest:
      memory: 256Mi
  type: Container
```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

```
apiVersion: v1
kind: LimitRange
metadata:
name: cpu-limit-range
spec:
 limits:
 - default:
    cpu: 1
    defaultRequest:
     cpu: 0.5
  type: Container
```

Best way to edit pods especifications is by editing the deployment

## DaemonSets
 
Sets up especific pods that always will be running in every node, especified by kube-api

To create a daemon set:

`kubectl create -f daemon-set-definition.yml`

Daemon Set Definition:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```
   
## Static Pods

Operate kubelet direct over a node with docker, created by kubelet it self

To kubelet could know how to place the pod we need to put pod-manifest files inside a designated path: /etc/kubernetes/manifests/pod1.yaml

We can specifie this path inside kubelet.service config daemon:

```
ExecStart=/usr/local/bin/kubelet \\
   --container-runtime=remote \\
   --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
   --pod-manifest-path=/etc/kubernetes/manifests \\
   --kubeconfig=/var/lib/kubelet/kubeconfig \\
   --network-plugin=cni \\
   --register-node=true \\
   --v=2
```

* OR

In kubeconfg file: kubeconfig.yaml

> staticPodPath: /etc/kubernetes/manifest

To view the static pods running at a node:

`docker ps -a`

To view the static pods from the master:

`kubectl get pods` ( static pods create a mirror config file at the master node )


## Additional Scheduler

To deploy an aditional scheduler:

Create a service template

> my-custom-scheduler.service

```
ExecStart=/usr/local/bin/kube-scheduler \\
--config=/etc/kubernetes/config/kube-scheduler.yaml \\
--scheduler-name=my-custom-scheduler
```

Create a static pod

/etc/kubernetes/manifests/kube-scheduler.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - comand:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    - --scheduler-name=my-custom-scheduler
    - --lock-object-name=my-custom-scheduler
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

To use custom scheduler, define in pod definition:

pod-definition.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-custom-scheduler
```

## Metrics Server

To get current metric infos about k8s cluster (In memory)
 
> git clone https://github.com/kubernetes-incubator/metrics-server.git

`kubctl create -f deploy/1.8+/`
  
To check metrics:

`kubectl top node`
`kubectl top pod`

## Updates and Rollout

To create a deployment with first version:

`kubectl create -f deployment-definition.yml`

To get deployments:

`kubectl get deployments`

To update a pod version:

> Edit the deployment config file:

`kubectl apply -f deployment-definition.yml`

* OR

Set image trough a command:

`kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1` (this option doesn’t edit the file.

To get status of versions:

`kubectl rollout status deployment/myapp-deployment`

To get history of versions:

`kubectl rollout history deployment/myapp-deployment`

To rollback the version:

`kubectl rollout undo deployment/myapp-deployment`

 
## Commands and Arguments

We can define a command that will act like entrypoint in the dockerfile and args that will act like cmd in the dockerfile.

Pod definition

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: [“sleep2.0”]
      args: [“10”]
```

## Config Maps

We can use to pass env vars to the container:
  
To define a config map:

* Imperative

```
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MOD=prod
  --from-file=app_config.properties
```
 
* Declarative

config-map-definition.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

`kubectl create -f config-map-definition.yaml`

To view configmaps:

`kubectl get configmaps`

To use in pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    envFrom:
      - configMapRef:
        name: app-config
```

* OR

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    env:
     - name: APP_COLOR
       valueFrom:
         configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

##Secrets

To store sensitive data.

* Imperative
  * Literal
    * `kubectl create secret generic app-secret --from-literal=DB_host=mysql`
 * OR From file
    * `kubectl create secret generic app-secret --from-file=app_secret.properties`

* Declarative
  * `kubectl create secret -f secret-definition.yaml`
 
secret-definition.yaml

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: wrRteXNxbMK0 (converted to a hash: echo -n ´mysql´ | base64: mysql )
  DB_User: wrRyb290wrQ= (converted to a hash: echo -n ´mysql´ | base64: root )
  DB_Password: wrQxMjMjNDU2wrQ= (converted to a hash: echo -n ´mysql´ | base64: 123#456 )
```

To use with a pod:

pod-definition.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    envFrom: (Reference secret)
      - secretRef:
        name: app-secret 
   env: (Reference single env)
    - name: DB_Password
      valueFrom:
        secretKeyRef:
         name: app-secret
         key: DB_Password
    volumes: (Reference as a volume, will store files with values)
      - name: app-secret-volume
        secret:
          secretName: app-secret
```

# Init Containers

Used to start first containers:

pod-definition.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
  initContainers:
   - name: init-myservice
     image: busybox:1.28
     command: [“sh”,”-c”]
```

## Cluster Maintenance

To clean a node and do not lost current pods that will be flushed after 5min ( as default, can change with kube-controller-manager --pod-eviction-timeout=5m0s )

`kubectl drain node-1 ( will recreate the pods in other nodes until the node-1 comes back )`
`kubectl uncordon node-1 ( will re-schedule the node after the drain )`
`kubectl cordon node-2 ( will unschedule the node, so this can’t gain new pods )`
 
## Kubernetes Upgrades

First of all upgrade the master node with your components, needs to be upgrated one by one minor version, 1.11 -> 1.12 -> 1.13: 

Using kubeadm tool:

##### At master node:

`kubeadm upgrade plan`

Will be solicited to first upgrade the kubeadm tool: 

`apt-get upgrade -y kubeadm=1.12.0-00`

`kubeadm upgrade apply v1.12.0`

##### if master node is running pods, update kubelet

`apt-get upgrade -y kubelet=1.12.0-00`

`systemctl restart kubelet`

##### At worker nodes:

First drain node:

`kubectl drain node-1`

Upgrade kubeadm and kubelet:

`apt-get upgrade -y kubeadm=1.12.0-00`

`apt-get upgrade -y kubelet=1.12.0-00`

`kubeadm upgrade node config --kubelet-version v1.12.0`

`systemctl restart kubelet`

Re-schedule the node:

`kubectl uncordon node-1`

## Backup and Restore

Can make a backup of resources configuration files and store in a git repository:

`kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml`

Can make a backup of ETCD:

Copy the data dir, can be founded at unit service template

Use the etcdctl cli to make a snapshot:

```
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
```

To view status from backup:

`ETCDCTL_API=3 etcdctl snapshot status snapshot.db`

To restore from this snapshot.db

`service kube-apiserver stop`

```
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
     --data-dir /var/lib/etcd-from-backup
```

Change the data dir path in etcd service template: /etc/systemd/system/etcd.service

`systemctl daemon-reload`

`service etcd restart`

`service kube-apiserver start`


## Authentication

> Certificate (Public Key) => *.crt, *.pem

> Private Key => *.key, *-key.pem

Needs to certificate all parts of kubernetes

* Servers
  * kubelet servers(nodes)
    * At host => kubelet.crt, kubelet.key
    * At master => kubelet-client.crt, kubelet-client.key (clients)
    * At apiserver => apiserver-kubelet-client.crt, apiserver-kubelet-client.key (clients)

  * kube-api server 
    * At host => apiserver.crt, apiserve 
    * At host to talk with etcd => apiserver-etcd-client.crt, apiserver-etcd-client.key (clients)

  * etcd server => etcdserver.crt, etcdserver.key

* And all services of kubernetes needs too (clients)
  * admin(kubctl REST api) => admin.crt, admin.key
  * kube-scheduler => scheduler.crt, scheduler.key
  * kube-controller-manager => controller-manager.crt, controller-manager.key
  * kube-proxy => kube-proxy.crt, kube-proxy.key

## To generate certificates

First we need to generates a CA, this could be made at master node and will be used to assign all the certs needed

`openssl genrsa -out ca.key 2048 (generates key)`
`openssl req -new -key ca.key -subj “/CN=KUBERNETES-CA” -out ca.csr (Certificate Signing Request)`
`openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt (Sign certificates)`

Then we need to generate the certs of all the Clients:


#### First the admin

> Generates key

`openssl genrsa -out admin.key 2048`

> Certificate Signing Request
> added a O in subject to set the group, in this case admin gain the most privileged

`openssl req -new -key admin.key -subj “/CN=kube-admin/O=system:masters” -out admin.csr`

> Sign certificate

`openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt`
 
#### Then the other Clients:

*kube-scheduler, kube-controller-manager, kube-proxy*

in this cases we must prefix the CN with system:

> Generates key

`openssl genrsa -out kube-scheduler.key 2048`

> Certificate Signing Request
> prefixed CN with system

`openssl req -new -key admin.key -subj “/CN=system:kube-scheduler” -out kube-scheduler.csr`

> Sign certificate

`openssl x509 -req -in kube-scheduler.csr -CA ca.crt -CAkey ca.key -out kube-scheduler.crt`
 
#### After the clients we need to generate the servers certificates:

> First ETCD server:

`openssl genrsa -out etcdserver.key 2048`
`openssl req -new -key admin.key -subj “/CN=etcdserver” -out etcdserver.csr`
`openssl x509 -req -in etcdserver.csr -CA ca.crt -CAkey ca.key -out etcdserver.crt`

If is a cluster of etcd servers needs to generate the certs for the peers:

> And kube-api server:

This case we must especify all the subjects name that other names use to comunicate:

`openssl genrsa -out apiserver.key 2048`

Set an openssl.cnf file to especify alternative names:

```
[req]
req_extensions = v3_req
[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation,
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 172.17.0.87
```

`openssl req -new -key apiserver.key -subj “/CN=kube-apiserver” -out apiserver.csr \
  --config openssl.cnf (Certificate Signing Request)`

`openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt`

Set at systemd unit exec start of kube-apiserver

```
ExecStart=/usr/local/bin/kube-apiserver \\
 --etcd-cafile=/var/lib/kubernetes/ca.pem \\
 --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt \\
 --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key \\
 --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
 --kubelet-client-certificate=/var/lib/kubernetes/apiserver-etcd-client.crt \\
 --kubelet-client-key=/var/lib/kubernetes/apiserver-etcd-client.key \\
 --client-ca-file=/var/lib/kubernetes/ca.pem \\
 --tls-cert-file=/var/lib/kubernetes/apiserver.crt \\
 --tls-private-key-file=/var/lib/kubernetes/apiserver.key \\
```

And kubelets nodes:

The name(CN) of nodes must follow system:node:node01...02...03

`openssl genrsa -out kubelet-client.key 2048`

`openssl req -new -key kubelet-client.key -subj “/CN=system:node:node01” -out kubelet-client.csr`

`openssl x509 -req -in kubelet-client.csr -CA ca.crt -CAkey ca.key -out kubelet-client.crt`

With the certificate we must add them to the kubelet-config.yml

```
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: “/var/lib/kubernetes/ca.pem”
authorization:
  mode: Webhook
clusterDomain: “cluster.local”
clusterDNS:
  - “10.32.0.10”
podCIDR: “${POD_CIDR}}”
resolvConf: “/run/systemd/resolve/resolv.conf”
runtimeRequestTimeout: “15m”
tlsCertFile: “/var/lib/kubelet/kubelet-node01.crt”
tlsPrivateKeyFile: “/var/lib/kubelet/kubelet-node01.key”
```

TIPS
POD
Create an NGINX Pod

`kubectl run nginx --image=nginx


Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

`kubectl run nginx --image=nginx  --dry-run=client -o yaml



Deployment
Create a deployment

`kubectl create deployment --image=nginx nginx



Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

`kubectl create deployment --image=nginx nginx --dry-run -o yaml



Generate Deployment with 4 Replicas

`kubectl create deployment nginx --image=nginx --replicas=4



You can also scale a deployment using the kubectl scale command.

`kubectl scale deployment nginx --replicas=4



Another way to do this is to save the YAML definition to a file.

`kubectl create deployment nginx --image=nginx--dry-run=client -o yaml > nginx-deployment.yaml

You can then update the YAML file with the replicas or any other field before creating the deployment.



Service
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml

(This will automatically use the pod's labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
(This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service
Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

`kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml

(This will not use the pods labels as selectors)

 

