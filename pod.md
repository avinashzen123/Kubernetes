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

busybox-test



<hr/>






     containers:
      - name: fibonacci
        image: ubuntu
        command: ["bash"]
        args: ["-c",  "max=200; a=0; b=1; for((i=0;i<=max;i++)); do echo -n \"$a \"; fn=$((a + b)); a=$b; b=$fn; done"]
      restartPolicy: Never
