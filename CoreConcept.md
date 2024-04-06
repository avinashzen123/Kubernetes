Kubernetes is an Open source system for automating deployment, scaling and managing of containerized application.

Kubernetes is an ecosystem providing core solution with many third party add-ons provided by approved projects, focusing on different areas.
- Networking
- Ingress
- Monitoring
- Packaging
- many more.

Kubernetes helps us in managing pod, monitoring pods, monitoring the containers in the pod, replacing failing pods and scaling pods.

Node is an machine physical or virtual on Which Kubernetes is installed.
Node is cluster can function as 
- Worker : Hosts pod that are component of the application
- Master : Manages the worker node and pod in the cluster.  If we need to have cluster to be highly available then we need to configure multiple master nodes. 
    Master component provide the cluster 'control plane'. They make global decision about the cluster for example scheduling.

Cluster is a group of node grouped together.

Master is a kubernetes node which watches over the nodes in the cluster and is responsible for actual orchestration of container on the worker node.

Cluster components
- Master components
    - kube-controller-manager
    - kube-apiserver
    - etcd
    - kube-scheduler
- Node components
    - kubelet
    - kube-proxy
    - Container runtime

Basic terms : System parts:
- Kubernetes : The whole orchestration system
- Kubectl : CLI to configure Kubernetes and manage apps
- Node : Single server in Kubernetes Cluster
- Kubelet : Kubernetes agent running on node
- Control plan : Set of container that manages cluster, including API Server, Scheduler, Controller manager, etcd ... Sometimes called master

etcd : is distributed storage system for key value 

kube-scheduler : is the master component that watches newly created pod that have no nod assigned and selects a ndoe for them to run on.

Scheduler container : Which will control how and where containers are placed on the nodes in object called pods

Controller manager : Which looks at state of whole cluster and everything that is running in it, and it uses API to do that. It takes the orders that are giving it or specs and determines difference between what we are asking it to do and what is going on. 

Controllers in kube-control manager : 
- Node controller : it is responsible for noticing and responding when node goes down
- Replication controller : This controller is responsible for maintaining the correct number of pod for every replication controller object in this system.
- Endpoint conroller : Its job is to populate the endopoint object that is to join service with their respective pods.
- Service account and token controller : Creates the default servicce account and API Tokens for new name spaces.

DNS : built in by default. (Core DNS)

**Components of Kubernetes cluster**:
- API Server : Acts as front end of Kubernetes. 



**Kubernetes Container abstractions :**
- Pod : One or more container running together on one node. Basic unit of deployment. WE don't deploy container directly inside container we deploy pod.
- Controller : For creating/updating pods and other objects. Many type of controller inlcudes Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob etc.
- Service : Network endpoint to connect to a POD.  Kubernetes Service is a logical abstraction for deployment groups of pods. 
Since pods are ephemeral, a service enables a group of pods, which provide specific function (web service, image processing etc) to be assigned a name and unique IP address (Cluster IP)

- Namespace : Filtered group of objects in cluster. It is just a filter on view at command line. It is not a security feature.
- Secret
- ConfigMap
- more ....

Resource categories :
- Workload: manage and run the application container on the cluter. Common controler : Deployment, StatefulSet, and Jobs
    Pods are created by Various controllers and Pods run container and provide environment dependencies such as a shared resource or persistent storage volume and configuration or secret data injection into the container.
- Discovery and load balancing (Service API resource) : 
- Config and Storage
- Cluster Resource 
- Metadata resource

> kubectl api-resources
    To list available resources.

> kubectl api-versions
    To list unique api versions 

**API Terminology** :
- Resource type 
- Resource kind
- Collection
- Resource

**NameSpace** : Are way to divide cluster resource between multiple user.

**Resource scoping** :  All resource type is scoped to entier cluster or a namespace.
1. Cluster Scoped resource
    * GET /api/VERSION/RESOURCETYPE  
        Eg. /api/v1/pods        
    * GET /api/VERSION/RESOURCETYPE/name  
2. Namespace scoped resource
    * GET /api/VERSION/namespaces/NAMESPACE/RESOURCETYPE  
        /api/V1/namespaces/test/pods
    * GET /api/VERSION/namespaces/NAMESPACE/RESOURCETYPE/NAME 

Limit the response size:  
    Eg. GET /api/v1/pods?limit=500 along with data we will get CONTINUE token which we need to use in next call.  
    Eg. GET /api/v1/pods/limit=500&continue=ENCODED_CONTINUE_TOKEN  

Representation of resource is by default application/json

Resources are deleted in two phases:
1. Finalization
2. Removal

**API Versioning** : Version is set at API Level rather than at the resource or field level. Different API version indicate different level of stability and support .
1. Alpha : e.g. v1alpha1. Recommended for use only in short lived testing cluster. Due to increased risk of bugs and lack of long term support
2. Beta : e.g. v2beta3. Recommended for non-business critical use cases. This version is well tested and featuer are enabled by default. It is recommended for only non-business users because of potential incompatible changes in subsequent release.
3. Stable : 'vx' E.g. v3

### API Groups : Groups are added to kubernetes API as it makes easier to expand. The legacy API are known to be part of the Core group.
* Core Group (Legacy API) : apiVersion is used as it is e.g. v1
* Named group : apiVersion as $GROUP_NAME/$VERSION e.g. batch/v1
* Enabling API Group : Certain api group are enabled by default. 
    - Enable or disbale API Group by setting --runtime-config on apiServer   
        e.g. to disable /batch/v1 :- --runtime-config=batch/v1=false. To enable /batch/v1alpha1 :- --runtime-config=batch/v1alpha1


### PODS :
- Single or Multiple container
    * init containers and app container
- Storage resources 
    * Shared storage volumes
- Unique network IP
    * Containers share the Network Namespaces
- Options governing how the container should run
- Ephemeral and disposable entities
- Kubernetes uses controller to implement POD Scaling and healing
- POD manifest (template)

<details>
    <summary> Pod Code </summary> 

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: testPod
spec:
    containers:
    - name: testContainer
      image: ubuntu
      command: ['sh', '-c', 'echo Hello Kubernetes && sleep 60']
    restartPolicy: Never  
```
</details>

> kubectl apply -f pod1.yaml   
    '-w' option to watch command result

> kubectl get pods

> kubectl logs testPod

### Phases of Pods:
- Pending : Accepted by the cluster, waiting
- Running : Running, starting or restarting
- Succeded 
- Failed
- Unknown

### Container states:
- Once POD is assigned to a node by scheduler, kubectl starts creating container using Container runtime
- Check the state of the container - ***kubectl describe pod [POD_NAME]***
- Waiting
- Running
- Terminated : succefully completed execution of failed.

### Termination of POD
- Pods having runnig processes and no longer required, needs to be terminated
    User -> kubectl delete -> POD   
    Container -> (TERM Sig | KILL Sig) -> POD  
- kubectl delete command supports the '--grace-period-<Seconds>' option
- Force deletion of a Pod deletes it from the Cluster state and etcd immediately

Command line tool supports three way of creating and managing Kubernetes Object :
- Imperative command
    * kubectl run nginx --image nginx
    * kubectl create deployment nginx --image ngnix
- Imperative object configuration
    * kubectl create -f config1.yaml
    * kubectl delete -f config2.yaml
- Declerative object configuration : User does not define the operation to be taken
    * Create, update, delete operation are automatically detected by 'kubectl'
    * kubectl diff -f confgs/
    * kubectl apply -f <directory>/
    * Better support for operating on directories and automatically detecting operation types
    * Harder to debug and understand results.


# Namespace : 
Namespece are a way to divide cluster resource between multiple users. We can specify resource limit. Namespace provide scope for names. Namespace are not nested.

Default namespaces:
- default : For objects with no specified namespace
- kube-system : Objects created by the Kubernetes system
- kube-public : Reserved for cluster usage
- kube-node-lease : conteains lease objects

> kubectl get namespaces   
    kubectl get ns

> kubectl create namespaces <name>  
    kubectl create ns secret

> kubectl ... -n namespace 
        to work in specific namespace

> kubectl get ... --all-namespaces 
        to see resource in all namespaces

> kubectl run --help  | less    
    To get help on a command

> kubectl explain pod.metadata


    Kubectl run mydb --image=mariadb --env MYSQL_ROOT_PASSWORD=password


> kubectl run nginx --image=nginx --namespace=<Namespace_name>   
    kubectl run nginx --image=nginx --n=<Namespace_name>   

> kubectl config set-context --current --namespace=<Namespace_name>   
    To set default namespace for all commands

> kubectl get pods --all-namespaces   
    Or kubectl get all -A to get object in all namespaces
    '-o wide' to know info like IP Address, Nominated Node, READINESS GATES

> kubectl explain pod.metadata

> kubectl exec -it <Pod_name> -- sh    
    To get terminal on running container . If pod has more than one container    
    kubectl exec -it <Pod_name> -c <Container_Name> -- sh

> kubectl run busybox --image=busybox -it --rm --command -- sh
    To start a container is interactive terminal and delete after stopping container.

Note : /proc


kubectl get pods mynginx -o json | less -> To get POD Description in JSON Format

kubectl get pods mynginx -o yaml| less -> To get POD Description in JSON Format

kubectl explain pods.spec.enableServiceLinks





https://github.com/avinashzen123/ckad/tree/master
