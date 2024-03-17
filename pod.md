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

# Resource limitation
By default, a pod will use as much CPU and memory as necessary to it work. This can be managed by using Memory/CPU request and limts in 'pod.spec.containers.resources'

A request is an initial request for resource.
A limit defines the upper threshold of resources a Pod can use.

CPU Limits are expressed in millicore or millicpu, 1/1000 of a CPU Core so 500 millicore is 0.5CPU

When being scheduled the kube-scheduler ensures that the node running the Pod hs all requested resource available.
If a Pod with resource limits can not be scheduled, it will show a status of Pending.

we can use 'kubectl set resource ...' to apply resource limit to running applicaiton in deployments.

**Memeory request :**   
1G (Gigabyte) = 1,000,000,000 bytes  
1M (Megabyte) = 1,000,000 bytes   
1K (Kilobyte) = 1,000 bytes 

1Gi (Gibibyte) = 1,073,741,824 bytes    
1Mi (Mebibyte) = 1,048,576 bytes  
1Ki (Kibibyte) = 1,024 bytes   

In case of CPU, the system throttles the CPU so that it does nto go beyound the specified limit. A container can not use more CPU resources than its limit. However this is not the case with memory. A container can use more memory resource than its limit. So if a pod tries to consume more memory than limits consistently then POD will be terminated with OOM Error.

```yaml
    spec:
      containers:
      - name: naginx-cont
        image: nginx
        resources:
          requests:
            memory: "100Mi"
            cpu: "500m"
          limits:
            memory: "128Mi"
            cpu: 1
```

**LimitRange** : LimitRange helps us define default values to be set for container in POD thatt are created without request of limit specified in Pod-definition file. This is applicable at namespace level.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constrain
spec:
  limits:
    - default:
        cpu: 500m
        memory: 1Gi
      defaultRequest:
        cpu: 200m
        memory: 1Gi
      max:
        cpu: "1"
        memory: 1Gi
      min:
        cpu: 100m
        memory: 1Gi
      type: Container
```

**Resource Quota**
* Quota are restrictions that are applied to namespace.
* If quota are set on a namespace, application started in that namespace must have resource

> kubectl create ns restricted
  kubectl create quota -n restricted --hard=cpu=2,memory=1G,pod=3

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
  namespace: restricted
spec:
  hard:
    requests.cpu: 2
    requests.memory: 4Gi
    limits.cpu: 4
    limits.memory: 8Gi
```

# Taint and toleration
Taints and Tolerance are used to set restrictions on what pods can be scheduled on a node 

When pods are created Kubernetes scheduler tries to place these pods on the available worker nodes.  As there is no restriction or limitation, scheduler will try to place pods across all the nodes equally to balance them out equally. 

Now lets us assume we have dedicated resource on a node for a particular use case or application so we would like only those pods that belongs to this application to be placed on that node.

First we prevent all pods from being placed on that node by placing a taint on that node. By default pods have no toleration which means unless specified otherwise none of pod can tolerate any taint.

Now we have to enable certain pods to be able to placed on that node. 

kubectl taint nodes node-name key=value:taint-effect

**taint-effect**
- NoSchedule : Pods will not be scheduled on the node 
- PreferNoSchedule : Which means the system will try to avoid placing pod on the node but is not guaranteed
- NoExecute : Which means that new pods will not be scheduled on the node and existing pod on the node if any will be evicted if they don't tolerate the taint. 

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
Note : Taint and Tolerance are only meant to restrict nodes from accepting certain pods 

To remove taint from a node, run taint command and "-" at end.
> kubectl taint nodes node1 app=blue:NoSchedule-



# Node Selector 
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
**kubectl get pod --selector name=mypod** 
NAME    READY   STATUS    RESTARTS   AGE  
mypod   0/2     Pending   0          33s  

# Cleaning up resource:
- kubectl delete all
- kubectl delete all --all
- kubectl delete all --all --force --grace-period=-1
    don't do force kill.

# Node affinity
Node affinity feature provides us with advanced capability to limit pod placement on specific node. 
<a herf="https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/">Assigning POD to Node</a>

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

<hr/>

     containers:
      - name: fibonacci
        image: ubuntu
        command: ["bash"]
        args: ["-c",  "max=200; a=0; b=1; for((i=0;i<=max;i++)); do echo -n \"$a \"; fn=$((a + b)); a=$b; b=$fn; done"]
      restartPolicy: Never
