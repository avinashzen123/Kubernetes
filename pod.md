A pod is an abstraction of a server. It runs multiple container within single namespace exposed by a single IP address.

**Ways of creating Kubernetes resource**
- Imperative : we create resource from command line
- Declarative : We create resource using manifest files


kubectl run mynginx --image=nginx

kubectl get pods [-o yaml]      
    -o yaml provides insight in all the Pod parameters in yaml format

kubectl describe pods <<Pod_name>>
    Shows all detail about a pod including information about containers running within pod with events  
    ex. kubectl get pods mynginx


**Basic YAML Manifest ingredients**
- apiVersion: specifies which version of API to use for this object
- kind: indicates the type of object (Deployment, pod)
- metadata: contains adminstrative information about the object
- spec: contains the specifics for the object

> kubectl explain   
    To get more information about the basic properties.

**Container component**
- name: the name of the container
- image: image that should be used.
- command: The command container should run
- args: argument that are used by the command
- env: Environment variable that should be used by the container.
Above all parts of the *pod.spec.container.spec* which can be checked with *kubectl explain*


```
kubectl explain pod | less
kubectl explain pod.spec | less

To view different parameters we user --recursive 

kubectl explain --recursive pod | less
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
  labels:
    name: mypod
spec:
  containers:
  - name: busybox
    image: busybox
    command:
      - sleep
      - "3600"
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: nginx
    image: nginx:1.7.9
    command:
      - sleep
      - "3600"
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80 
```
once we have manifest file. We can use 'apply', 'create' or 'replace' command 

> kubectl apply -f busybox.yaml

| create | apply    | replace |
|-------|-----------|---------|
| Will create resource | Will create resource if does not exist otherwise update resource's modifiable properties | Will replace a resource with the new configuration file |

The kubectl replace command has a --force option which actually does not use the replace, i.e., PUT, API endpoint. It forcibly deletes (DELETE) and then recreates, (POST) the resource using the provided spec.


> kubectl delete -f my.yaml 
    Will remove everything specified in the YAML file.

**Generating YAML File**
    --dry-run=client -o yaml > poddef.yaml 
    Ex. kubectl run mynginx --image=nginx --dry-run=client -o yaml > mynginx.yaml
    
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mynginx
  name: mynginx
spec:
  containers:
  - image: nginx
    name: mynginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```


> kubectl run mybusybox --image=busybox -o yaml -- sleep 3600 > busyboxpod.yaml

> kubectl run -i -t upstox --image=upstox --restart=Never

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: commandtest
spec:
  containers:
  - name: commandtest
    image:  busybox
    command:
      - echo
      - "Hello from commandtest pod"
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**kubectl create -f commandArgument.yaml**
pod/commandtest created
**kubectl logs pod/commandtest**
Hello from commandtest pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: commandtest
spec:
  containers:
  - name: commandtest
    image:  busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> ~/dates.txt; sleep 10; done"]
```

**There are some define use case where we might want to run multiple container in a POD.**
- *Sidecar container*: a container that enhances the primary application for instance for logging
- *Ambassador container*: a container that represents the primary container to the outside world such as a proxy
- *Adapter container* : used to adopt the traffic or data pattern to match the traffic or data pattern in other applicaiton in the cluster.

Note : Sidecar containers etc are just a multi container POD

Side car is useful in Service mesh ex: <a href="https://istio.io/latest/about/service-mesh/">Istio</a>




Setting up hostanme and environment variable.

kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: hostname
  name: hostname
spec:
  hostname: busybox-test
  hostAliases:
  - ip: "192.168.1.140"
    hostnames:
    - "test1.local"
    - "test2.local"
  containers:
  - image: busybox
    name: hostname
    command: ['printenv']
    # command: ['cat /etc/hosts']
    # command: ['hostname']
    env:
    - name: ENVTEST1
      value: "This is ENVTEST1 Value"
    - name: ENVTEST2
      value: "This is ENVTEST2 Value"
```

**kubectl logs hostname**     
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=busybox-test
ENVTEST1=This is ENVTEST1 Value
ENVTEST2=This is ENVTEST2 Value
HOME=/root

127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.1.1.3        busybox-test

Entries added by HostAliases.
192.168.1.140   test1.local     test2.local


# Pod Placement 

* nodeName
* nodeSelector
* affinity : for advanced option like Exists, Gt, Ln, etc.
  * nodeAffinity : to choose node
  * podAffinity : to co-locate pod with pod labels topology key
  * podAntiAffinity : to keep away pods with pod label topologyKey
* Taint : addded onto node. Only Podw which can tolerate this taint can be scheduled.
  * NoSchedule : Hard (Do not schedule pods if they can't tolerate taint)
  * PreferNoSchedule : Soft (Can be scheduled if no other nodes available)
  * NoExecute : Strict (Delete running pods also if they can't tolerate newly added taint)

# Node Name

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nodename
  name: nodename
spec:
  nodeName: minikube-m03
  containers:
  - image: nginx
    name: nodename
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

# Node Selector 
nodeSelector is the simplest recommended form of node selection constraint. It specifies map of key value pairs. For the pod to be eligible to run on a ndoe, the node must have each of the indicated key-value pairs as labels.

Let say we have three node cluster of which two node are smaller in terms of hardware resource. We have different kind of workloads running in our cluster. We would like to dedicate the data processing workload that require higher processing power to larger node as that is the only node which will not run Out of resources in case of job demands extra resource

To solve this we need to add limitation on pod, which we can do in two ways:
	- Node selector (simpler and easier)
	- Node affinity.

Before creating pod we must label node
> kubectl label nodes <Node_name> <label_key>=<label_value>
  kubectl label nodes node-1 size=Large

```yaml
spec:
  nodeSelector:
    size: Large
  containers:
  - name: busybox
    image: busybox
```

# Affinity and Anti-affinity 
Node affinity feature provides us with advanced capability to limit pod placement on specific node. 
<a herf="https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/">Assigning POD to Node</a>

- The affinity/anti-affinity langauage is more expressive. It gives us more control over the selection logic.
- WE can indicate that a rule is soft or preferred so that the scheduler still schedules the pod even if it can't find a matching node.
- We can constraint a Pod using labels on other Pods' running on the node ( or other topological domain), instead of just node lables which allow us to define rule for which Pods can be co-lcated on a Node.

With nodeSelector we can not provide complex condition like "rank>3" or some label exists. Because of which more powerful mechanism was introduction called "Affinity"

Affinity it is of two types:
- NodeAffinity
- PodAffinity

There are two type of node affinity : 
1. **requiredDuringSchedulingIgnoredDuringExecution**: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
2. **preferredDuringSchedulingIgnoredDuringExecution**: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

Operator we can use 
- In
- NotIn
- Exists
- DoesNotExist
- Gt
- Lt

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    type: backend
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: type
            operator: In
            values:
            - backend
            - frontend
  containers:
  - image: nginx
    name: nginx
    resources: {}
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:  
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 90
        preference:
          matchExpressions:
          - key: type
            operator: In
            values:
            - frontend
      - weight: 10
        preference:
          matchExpressions:
          - key: type
            operator: In
            values:
            - backend
  containers:
  - image: nginx
    name: nginx
```

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```

**Inter pod affinity and anti affinity** : nter pod affinity and anti affinity allows us to constrain which nodes or Pod can be scheduled based on the labels of Pod already running that nodes instead of the node label.

Inter-pod affinity and anti-affinity rules take the form "This pod should (or, in the case of anti-affinity should not) run in an X if that X is already running one or more Pod that meets the rule Y", Where X is a topology domain like node, rack, cloud provider zone or region or similar and Y is the rule Kubernetes tries to Satisfy.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: registry.k8s.io/pause:2.0
```

# Taint and toleration
Taints and Tolerance are used to set restrictions on what pods can be scheduled on a node 

When pods are created Kubernetes scheduler tries to place these pods on the available worker nodes.  As there is no restriction or limitation, scheduler will try to place pods across all the nodes equally to balance them out equally. 

Now lets us assume we have dedicated resource on a node for a particular use case or application so we would like only those pods that belongs to this application to be placed on that node.

First we prevent all pods from being placed on that node by placing a taint on that node. By default pods have no toleration which means unless specified otherwise none of pod can tolerate any taint.

Now we have to enable certain pods to be able to placed on that node. 

kubectl taint nodes node-name key=value:taint-effect

**taint-effect**
- **NoSchedule** : Pods will not be scheduled on the node 
- **PreferNoSchedule** : Which means the system will try to avoid placing pod on the node but is not guaranteed
- **NoExecute** : Which means that new pods will not be scheduled on the node and existing pod on the node if any will be evicted if they don't tolerate the taint. 

> kubectl taint nodes node1 app=blue:NoSchedule

```yaml
  containers:
  - name: busybox
    image: busybox
    tolerations:
    - key: "node-role.kubernetes.io/master"
      operator: "Exists" # Equal
      effect: "NoSchedule"
      value: blue
```

If Nodes' taint and Pods toleration configuration matches then pod will be scheduled on that node.

Note : Taint and Tolerance are only meant to restrict nodes from accepting certain pods 

To remove taint from a node, run taint command and "-" at end.
> kubectl taint nodes node1 app=blue:NoSchedule-




**kubectl get pod --selector name=mypod** 
NAME    READY   STATUS    RESTARTS   AGE  
mypod   0/2     Pending   0          33s  

# Cleaning up resource:
- kubectl delete all
- kubectl delete all --all
- kubectl delete all --all --force --grace-period=-1
    don't do force kill.



<hr/>

     containers:
      - name: fibonacci
        image: ubuntu
        command: ["bash"]
        args: ["-c",  "max=200; a=0; b=1; for((i=0;i<=max;i++)); do echo -n \"$a \"; fn=$((a + b)); a=$b; b=$fn; done"]
      restartPolicy: Never
