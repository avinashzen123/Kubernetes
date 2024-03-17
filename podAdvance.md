# Init containers

- An init container is an additional container in a POD that complets a task before the "regular" container is started
- The regular container will only be started once the init container has been started
- If the init conainer has not run to completion the main container is not started.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container
  labels:
    name: init-container
spec:
  containers:
  - name: nginx
    image: nginx
  initContainers:
  - name: install
    image: busybox
    command: ["/bin/sh","-c","sleep 10"]
```


# Static pod
- Regular pods are managed by the Kubernetes Control plane
- Static pods are managed directly by the Kubelet daemon on a specific node, without the API Server observing them.
- Static pods running on a node are visisble on the API Server, but cannot be controlled from there.

Static POD are always bound on one kubelet on specific node. The kubelet watches each static POD and restarts if it crashes. The Kubelet automatically tries to create a mirror POD in Kuberenetes API server for each static POD. This means that POD running on a NODE are visible on API server but can not be controller from there.

"/etc/kubernetes/manifest" path where Kubelet looks for static POD manifest. Kubelet watches this directory for POD to created and running Kubelet periodically scans the configured directory for change and adds or remove PODS as file appear or disappear in this directory. 

# Jobs

Kubernetes runs a container, performs the computatoin task and exits and pod goes into complete state. But Kubernetes then recreates the container in attemt to leave it running. Again container performs required computation and exits and kubernetes agains tries to bring it back again and this continues to happen until a threashold is reached.  

This is happening because Kubernetetes wants application to live forever. 

This is defined by property "restartPolicy" set on pod. 

> kubectl create job my-job --image=busybox -o yaml --dry-run=client -- expr 3 + 2 > firstJob.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  backoffLimit: 4
  completions: 1
  parallelism: 1
  template:
    metadata:
    spec:
      containers:
      - command: ['expr', '3', '+', '2']
        image: busybox
        name: my-job
        resources: {}
      restartPolicy: Never
status: {}
```
backoffLimit: to specify the number of retries before considering a Job as failed
completion : how many successful pods are needed to complete a job
parallelism: How many succeessful pod replicase should run in parallel
activeDeadlineSeconds: Maximum duration the job can run.

# Cron job

kubectl create  cronjob throw-dice-cron-job  --image=kodekloud/throw-dice --schedule="30 21 * * *"  --dry-run=client -o yaml >  throw-dice-cron-job.yaml

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: throw-dice-cron-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: throw-dice-cron-job
    spec:
      template:
        spec:
          containers:
          - image: kodekloud/throw-dice
            name: throw-dice-cron-job
            resources: {}
          restartPolicy: OnFailure
  schedule: 30 21 * * *
status: {}
```

schedule: Scheduled time for cron job to be creaetd and executed 
successfulJobsHistoryLimit: how many successfully completed jobs should be kept in job history
suspend       <boolean>
    This flag tells the controller to suspend subsequent executions, it does not
    apply to already started executions.  Defaults to false.


> kubectl create job test-job --from=cronjob/a-cronjob  
    Creates a job from cron job



# Deployment

Deployment add expanded support fopr Software development and development lifecycle.
It is task of deployment to ensure that enough Pods are running at all times.

- Describe the desired state of a component of the application
- Deployment involve one or more ReplicaSet
- Change are propagated without input from the user

It offers featuers that add to the scalability and reliability of the application
- Scalability : Scaling the number of application instances
- Updates and Update strategy : Zero downtime application updates.

> kubectl create deployment myweb --image=nginx --replicas=3 -o yaml --dry-run=client > deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myweb
  name: myweb
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - image: nginx
        name: nginx
```

If we delete any pod which is managed by deployment, deployment will create replacement of that pod.

**kubectl get all**
NAME                        READY   STATUS    RESTARTS   AGE    
pod/myweb-9794cbc77-nxhnr   1/1     Running   0          2m47s  
pod/myweb-9794cbc77-rq6tp   1/1     Running   0          2m47s  
pod/myweb-9794cbc77-xspl7   1/1     Running   0          2m47s  

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5m32s 
    
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE    
deployment.apps/myweb   3/3     3            3           2m47s  

NAME                              DESIRED   CURRENT   READY   AGE   
replicaset.apps/myweb-9794cbc77   3         3         3       2m47s 
**kubectl delete pod/myweb-9794cbc77-xspl7**
pod "myweb-9794cbc77-xspl7" deleted 
**kubectl get all**                         
NAME                        READY   STATUS              RESTARTS   AGE  
pod/myweb-9794cbc77-f7trc   0/1     ContainerCreating   0          2s   
pod/myweb-9794cbc77-nxhnr   1/1     Running             0          2m54s    
pod/myweb-9794cbc77-rq6tp   1/1     Running             0          2m54s    

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5m39s 

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE 
deployment.apps/myweb   2/3     3            2           2m54s  

NAME                              DESIRED   CURRENT   READY   AGE   
replicaset.apps/myweb-9794cbc77   3         3         2       2m54s 

<hr/>
After deployment was introduced, scalability is managed through Deployment and no longer in ReplicaSet

**Deployment updates.** 
-  We can execute command
     kubectl scale deployment myweb --replicas=4
- We can edit deployment using 'kubectl edit deployment myweb' command and change value of 'replicas' and set value. 
    Note with 'edit' command only editable properties can be changed.
- kubectl set image deployment myweb nginx=nginx:1.17
    kubectl set env deploy mywe type=web-app

Kubectl set commands: Available Commands:
- env           :    Update environment variables on a pod template   
- image         :    Update the image of a pod template   
- resources     :    Update resource requests/limits on objects with pod templates    
- selector      :    Set the selector on a resource   
- serviceaccount:   Update the service account of a resource 
- subject       :   Update the user, group, or service account in a role binding or cluster role binding 

When an update is applied, a new ReplicaSet is created with new properties:
- Pod with new properties are started in the new ReplicaSet
- After successful updates, the old ReplicaSet is no longer user and may be deleted
- The deployment.spec.revisionHistoryLimit is set to keep the 10 last ReplicaSets

type: RollingUpdate


Note : To know API Version - kubectl api-resources | grep deployment

> **kubectl port-forward  deployment.apps/myweb  32000:80**
    Expose service for testing . Forward to host port without creating service 

> kubectl get all --selector app=myweb

Note : if we change label of deployment after creation, labels will not be inherited by existing resource, it will be inherited by new resource created by deployment.
    kubectl label deployment myweb env=dev  
    kubectl get all --show-labels   

Update strategy: When a deployment changes, the Pods are immediately updated according to the update strategy.
- **Recreate** : all pods are killed and new pods are created. This will lead to temporary unavailability. Usefule if we can not run different version of an application.
- **RollingUpdate** : Update pod one at a time to guarantee availability of the application. This is preferred approach and we can further tune its behavior.

When making chang, this change will be applied as a rolling update:  the changed version is deployed in a new ReplicaSet
After the update has been confirmed as successful, the old version of ReplicaSet is scaled to 0 to deactivate.
We can use '**kubectl rollout history**' to get details about recent changes.
The RollingUpdate options are used to guarantee a  certain minimal and maximal number of Pods are always available:
- **maxUnavailable** : determine the maxmimum number of PODs that re  upgraded at the same time
- **maxSurge** : the number of Pods that can run beyound the desired number of Pods as specified in a replicas to guarantee minimal availability.

## Deployment history
- In deployment updates, the Deployment creates a new ReplicaSet  that uses the new properties.
- The old ReplicaSet is kept, but the number of Pods will be set to zero
- This makes it easy to undo a change
- **kubectl rollout history** will show the rollout history of a specific deployment which can easily be reverted as well.
- Use 'kubectl rollout history deployment myweb --revision=1' to observer changes between version.

**kubectl rollout history deployment myweb**
deployment.apps/myweb   
REVISION  CHANGE-CAUSE  
2         <none>    

**kubectl rollout history deployment myweb --revision=2**
deployment.apps/myweb with revision #2  
Pod Template:   
  Labels:       app=myweb   
        pod-template-hash=6bbcbdd554    
  Containers:   
   nginx:   
    Image:      nginx:1.17  
    Port:       <none>  
    Host Port:  <none>  
    Environment:        <none>  
    Mounts:     <none>  
  Volumes:      <none>            

**kubectl rollout undo** to undo previous change
kubectl rollout undo deployment myweb --to-revision=1

**kubectl rollout -h | less**
Valid resource types include:
*  deployments
*  daemonsets
*  statefulsets

# Auto scaling
- In real world PODs are often automatically scaled based on resource usage properties that are collected by the Metrics server
- The horizontal POD autoscaler observs usage statistics and after passing a threshold will add additional replicas.

> kubectl autoscale deployment myweb --cpu-percent=50 --min=1 --max=10


kubectl autoscale -h | less

Examples:
### Auto scale a deployment "foo", with the number of pods between 2 and 10, no target CPU utilization specified so a default autoscaling policy will be used
kubectl autoscale deployment foo --min=2 --max=10

### Auto scale a replication controller "foo", with the number of pods between 1 and 5, target CPU utilization at 80%
kubectl autoscale rc foo --max=5 --cpu-percent=800

To enable metric server in minikube: **minikube addons enable metrics-server**


# StatefulSet
Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of its Pods


If you want to use storage volumes to provide persistence for your workload, you can use a StatefulSet as part of the solution. Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed

Using StatefulSets
- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

Limitations:
- The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner based on the requested storage class, or pre-provisioned by an admin.
- Deleting and/or scaling a StatefulSet down will not delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
- StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods. You are responsible for creating this Service.
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.
- When using Rolling Updates with the default Pod Management Policy (OrderedReady), it's possible to get into a broken state that requires manual intervention to repair.

StatefulSet maintains a sticky identity for each of their pods
Useful for application which require stable identifiers ordered deployment
When PODs are being deployed, they are created sequentially in order from {0, ... n-1}
When PODs are being deleted they are terminated sequentially in order from {0, ... n-1}

Note : TO achieve graceful termination of the POD in StatefulSet we should scale the StatefulSet down to zero prior to deletion.


```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: whizspace
spec:
  selector:
    app: nginx
  clusterIP: None
  ports:
  - port: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-ss
  namespace: whizspace
spec:
  serviceName: "nginx-svc"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

If we want separate volume for each pod, each POD will need PVC for itself. We can achive this by "VolumeClaimTemplate". It is PersistentVolumeClaim instead it is just templatized. It just means instead of creating a PVC manually and then specifying it in the StatefulSet  definitino file, we can move the entire PVC Definitino into a section called "valueClaimTemplates" under the StatefulSet specification.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mystatefulset
spec:
  selector:
    matchLabels:
      app: myapp
  serviceName: headlessservce
  replicas: 2
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: google-storage #Optional, before that we need to create storage
      resources:
        requests:
          storage: 1Gi
```

NAME                  READY   STATUS    RESTARTS   AGE  
pod/mystatefulset-0   1/1     Running   0          16s  
pod/mystatefulset-1   1/1     Running   0          13s  

NAME                             READY   AGE  
statefulset.apps/mystatefulset   2/2     16s  


```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: 
  name: google-storage
provisioner: kubernetes.io/gce-pd
```


# DaemonSet
- A DaemonSet is a Deployment that starts one POD instance on every node in the cluster
- This is useful in case where a Software component like an agent need to be available on all cluster nodes
- When node is added or removed , the DaemonSet automatically changes the number of POD automatically.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginxdaemon
  namespace: default
  labels:
    k8s-app: nginxdaemon
spec:
  selector:
    matchLabels:
      name: nginxdaemon
  template:
    metadata:
      labels:
        name: nginxdaemon
    spec:
      containers:
      - name: nginx
        image: nginx

```