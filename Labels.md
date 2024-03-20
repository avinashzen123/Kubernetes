Labels and selectors are used to organize and to select subset of objects such as Pods
Annotations on the other hand are used to attach arbitrary non-identifying metadata object

Key have two segment : an optional prefix and name , separated by slash (/)
- Prefix are optional and must be a DNS subdomain
- Name segement : is required and must be 63 characters or less   
    e.g. Key/value pair as google.com/environment:production
- kubernetes.io/ and k8s.io/ prefixes are reserved for kubernetes core components.

**Selectors**
- Used to identify a set of objects
- Equality based selectors
    * Matching object must satisfy all the specified label constraints
    * Three kind of operators are admitted =, == and !=   
        '=' and '==' are same. in case '!=' it selects all resources which does not have label
        e.g. environment = production   
        e.g. tier != frontend  
- Set based selectors
    * Three kind of operators are submitted in 'notin' and 'exists'   
        e.g. environment in (production, qa)

> kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"


> kubectl label pods -l app=nginx tier=fe  
    Command to update label

> kubectl get pods -l app=nginx -L tier 
    Get pod with selector

### Example Selector :
```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - { key: tier, operator: In, values: [cache] }
    - { key: environment, operator: NotIn, values: [dev] }
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
      - name: simple-webapp
        image: nginx

```

> kubectl get pods --selector environment=dev --show-labels
    To view current labels

**kubectl label pods -l app=App1 tier=fe**
pod/simple-webapp-cm9s7 labeled
pod/simple-webapp-hxdwl labeled
pod/simple-webapp-t4266 labeled

Get pod where app = App1 and there is lable named "tier"
**kubectl get pods -l app=App1 -L tier** 
NAME                  READY   STATUS    RESTARTS        AGE     TIER
simple-webapp-cm9s7   1/1     Running   1 (6h53m ago)   7h39m   fe
simple-webapp-hxdwl   1/1     Running   1 (6h53m ago)   7h39m   fe
simple-webapp-t4266   1/1     Running   1 (6h53m ago)   7h39m   fe    
**kubectl get pods -l app=App1**      
NAME                  READY   STATUS    RESTARTS        AGE
simple-webapp-cm9s7   1/1     Running   1 (6h53m ago)   7h39m   
simple-webapp-hxdwl   1/1     Running   1 (6h53m ago)   7h39m 
simple-webapp-t4266   1/1     Running   1 (6h53m ago)   7h39m   
**kubectl label pods -l app=App1 environment=dev tier=frontend --overwrite**
  Will search for pod where lable app = App1 the set environment and tier


**kubectl get pods -l environment=dev,tier=frontend** 
'NAME                  READY   STATUS    RESTARTS        AGE
simple-webapp-cm9s7   1/1     Running   1 (6h59m ago)   7h44m 
simple-webapp-hxdwl   1/1     Running   1 (6h59m ago)   7h44m 
simple-webapp-t4266   1/1     Running   1 (6h59m ago)   7h44m   

**kubectl get pods -l 'environment in (dev), tier in (frontend)'**

NAME                  READY   STATUS    RESTARTS        AGE
simple-webapp-cm9s7   1/1     Running   1 (6h59m ago)   7h45m 
simple-webapp-hxdwl   1/1     Running   1 (6h59m ago)   7h45m 
simple-webapp-t4266   1/1     Running   1 (6h59m ago)   7h45m 
**kubectl get pods -l 'environment in (dev), tier notin (backend)'**
NAME                  READY   STATUS    RESTARTS     AGE
simple-webapp-cm9s7   1/1     Running   1 (7h ago)   7h45m  
simple-webapp-hxdwl   1/1     Running   1 (7h ago)   7h45m  
simple-webapp-t4266   1/1     Running   1 (7h ago)   7h45m  

**kubectl get pods --selector environment=dev**
NAME                  READY   STATUS    RESTARTS   AGE  
simple-webapp-6wmjv   1/1     Running   0          11m  
simple-webapp-86c69   1/1     Running   0          11m  
simple-webapp-glndl   1/1     Running   0          11m  

**exists example**
**kubectl get pods -l 'app'**       
NAME                  READY   STATUS    RESTARTS   AGE
simple-webapp-6wmjv   1/1     Running   0          4m31s  
simple-webapp-86c69   1/1     Running   0          4m31s  
simple-webapp-glndl   1/1     Running   0          4m31s  
**kubectl get pods -l '!app'**
No resources found in default namespace.  


> kubectl get pods -Lapp -Ltier -Lrole  
    To print lable in command output

> kubectl create deploy bluelabel --image=nginx   
    Creates pod with label app=bluelabel

### To remove label use 'key-'
**kubectl label pods -l app=App1 environment=dev tier-**       
pod/simple-webapp-6wmjv unlabeled 
pod/simple-webapp-86c69 unlabeled 
pod/simple-webapp-glndl unlabeled 

## Annotations:
- Attach arbitrary metadata to objects
- Not used to identify and select objects

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    imageregistry: "https://hub.docker.com/"
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```

> kubectl get pods -o wide