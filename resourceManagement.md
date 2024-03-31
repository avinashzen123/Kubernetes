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


# QoS Class

Kubernetes classifies the Pods that you run and allocates each Pod into a specific quality of service (QoS) class. Kubernetes uses that classification to influence how different pods are handled. Kubernetes does this classification based on the resource requests of the Containers in that Pod, along with how those requests relate to resource limits. This is known as Quality of Service (QoS) class.

Kubernetes uses QoS classes to make decision about evicting Pods when Node resources are exceeded.

When Kubernetes creates a Pod it assigns one of these QoS classes to the Pod:

- Guaranteed : Pods that are guaranteed have the strictest resource limit and are least likely to face eviction. They are guaranteed not be killed until they exceed their limit or there are no lower priority POds that can preempted from the Node. 
    * Every container in Pod must have memory limit and a memory request
    * For every container in the Pod, memory limit must equal to memory request
    * Every container in the Pod must have a CPU Limit and a CPU request
    * For every container in the Pod, CPU limit must be equal to CPU request

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```

- Burstable : Pods that are Burstable have some lower-bound resource guarantees based on the request, but do not require a specific limit. If a limit is not specified, it defaults to a limit equivalent to the capacity of the Node, which allows the Pods to flexibly increase their resources if resources are available. In the event of Pod eviction due to Node resource pressure, these Pods are evicted only after all BestEffort Pods are evicted. Because a Burstable Pod can include a Container that has no resource limits or requests, a Pod that is Burstable can try to use any amount of node resources.

    * The Pod does not meet the criteria for QoS class Guaranteed.
    * At least one Container in the Pod has a memory or CPU request or limit.    

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-2
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```

- BestEffort : Pods in the BestEffort QoS class can use node resources that aren't specifically assigned to Pods in other QoS classes. For example, if you have a node with 16 CPU cores available to the kubelet, and you assign 4 CPU cores to a Guaranteed Pod, then a Pod in the BestEffort QoS class can try to use any amount of the remaining 12 CPU cores.

Pod is BestEffort only if none of the Containers in the Pod have a memory limit or a memory request, and none of the Containers in the Pod have a CPU limit or a CPU request. Containers in a Pod can request other resources (not CPU or memory) and still be classified as BestEffort.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-3
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-3-ctr
    image: nginx
```

**LimitRange** : LimitRange helps us define default values to be set for container in POD that are created without request of limit specified in Pod-definition file. This is applicable at namespace level.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constrain
spec:
  limits:
    - type: Pod
      min:
        cpu: 500m
        memory: 5Mi
      max:
        cpu: "2"
        memory: 2Gi
    - type: PersistentVolumeClaim
      min:
        storage: 1Gi
      max:
        storage: 10Gi
    - type: Container
      default:
        cpu: "1"
        memory: 2Gi
      defaultRequest:
        cpu: 500m
        memory: 1Gi
      max:
        cpu: "1"
        memory: 2Gi
      min:
        cpu: 500m
        memory: 1Gi
      maxLimitRequestRatio:
        cpu: "2"
        memory: "2"
```


# ResourceQuota

When several users or teams share a cluster with a fixed number of nodes, there is a concern that one team could use more than its fair share of resources.

Resource quotas are a tool for administrators to address this concern.

**ResourceQuota** is for limiting the total resource consumption of a namespace, for example:

> kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
spec:
  hard:
    configmaps: "10" 
    persistentvolumeclaims: "4" 
    replicationcontrollers: "20" 
    secrets: "10" 
    services: "10"
```
Compute Resource Quota

|Resource Name |	Description|
|-----------|----------------
|limits.cpu |	Across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value.|
|limits.memory|	Across all pods in a non-terminal state, the sum of memory limits cannot exceed this value.|
|requests.cpu|	Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value.|
|requests.memory|	Across all pods in a non-terminal state, the sum of memory requests cannot exceed this value.|
|hugepages-<size>|	Across all pods in a non-terminal state, the number of huge page requests of the specified size cannot exceed this value.|
|cpu|	Same as requests.cpu|
|memory|	Same as requests.memory|



**Resource Quota**
* Quota are restrictions that are applied to namespace.
* If quota are set on a namespace, application started in that namespace must have resource

> kubectl create ns restricted
  kubectl create quota -n restricted --hard=cpu=2,memory=1G,pod=3

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespacequota
  namespace: default
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "4Gi"
    limits.cpu: "4"
    limits.memory: "8Gi"
    pods: "10"
    secrets: "10"
    configmaps: "10"
    services: "10"
    services.loadbalancers: "2"
    services.nodeports: "2"
    replicationcontrollers: "5"
    persistentvolumeclaims: "4"

```
