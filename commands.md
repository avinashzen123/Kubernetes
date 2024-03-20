

> kubectl create namespace myns -o yaml --dry-run=client

> kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml

> kubectl get po --all-namespaces

> kubectl run busybox --image=busybox --command --restart=Never -it --rm -- env 
    Note :  -it will help in seeing the output, --rm will immediately delete the pod after it exits

> kubectl run busybox --image=busybox --command --restart=Never -- env

> kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml

> kubectl set image pod/nginx nginx=nginx:1.7.1

to check pods image of running container .
    kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'


> kubectl get po nginx -w
    To watch pods creation

> kubectl create namespace mynamespace
    kubectl run nginx --image=nginx --restart=Never -n mynamespace

kubectl run nginx --image=nginx --port=80

NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')

kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'

kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')

kubectl get po nginx -o yaml

kubectl get po nginx -oyaml

kubectl get po nginx --output yaml

kubectl get po nginx --output=yaml

kubectl describe po nginx

kubectl logs nginx

kubectl logs nginx -p

kubectl logs nginx --previous

kubectl exec -it nginx -- /bin/sh

kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
or 
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'

> To get pod delete automatically
    kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'

>   Set environment
    kubectl run nginx --image=nginx --restart=Never --env=var1=val1
    kubectl exec -it nginx -- env
    kubectl exec -it nginx -- sh -c 'echo $var1'

kubectl describe po nginx | grep val1

kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env

kubectl run nginx --image nginx --restart=Never --env=var1=val1 -it --rm -- sh -c 'echo $var1'


> kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
    kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
    kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
    # or
    for i in `seq 1 3`; do kubectl run nginx$i --image=nginx -l app=v1 ; done

> Show all labels
    kubectl get po --show-labels

> Change label:
    kubectl label po nginx2 app=v2 --overwrite


> Get only the 'app=v2' pods
    kubectl get po -l app=v2
    kubectl get po -l 'app in (v2)'
    kubectl get po --selector=app=v2

> Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels
    kubectl label po -l "app in(v1,v2)" tier=web

> Remove labels from pod
    kubectl label po nginx1 nginx2 nginx3 app-
    kubectl label po nginx{1..3} app-
    kubectl label po -l app app-


# Annotation
> Check annotation for pod
    kubectl annotate pod nginx1 --list
    kubectl describe po nginx1 | grep -i 'annotations'
    kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description

> Add an annotation 'owner: marketing' to all pods having 'app=v2' label
    kubectl annotate po -l "app=v2" owner=marketing

> Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value
    kubectl annotate po nginx1 nginx2 nginx3 description='my description'
    kubectl annotate po nginx{1..3} description='my description'

> Remove annotation
    kubectl annotate po nginx{1..3} description- owner-

> Delete pod
    kubectl delete po nginx{1..3}


# Pod Placement

> Node selector
```yaml
spec:
  nodeSelector:
    size: Large
  containers:
  - name: busybox
    image: busybox
```


>  Node affinity
    Node affinity feature provides us with advanced capability to limit pod placement on specific node. 

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

> Taint 
Taint a node with key tier and value frontend with the effect NoSchedule. Then, create a pod that tolerates this taint

    kubectl taint node node1 tier=frontend:NoSchedule # key=value:Effect
    kubectl describe node node1 # view the taints on a node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "tier"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
```


> Create a pod that will be placed on node controlplane. Use nodeSelector and tolerations.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    kubernetes.io/hostname: controlplane
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```

# Deployment


> Scale the deployment to 5 replicas
    kubectl scale deploy nginx --replicas=5

> Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%
    kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
    #. view the horizontalpodautoscalers.autoscaling for nginx
    kubectl get hpa nginx



kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property

#. OR you can find rs directly by:
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
#. you could also just do kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml

    kubectl get po # get all the pods
    #. OR you can find pods directly by:
    kubectl get po -l run=nginx # if you created deployment by 'run' command
    kubectl get po -l app=nginx # if you created deployment by 'create' command
    kubectl get po nginx-7bf7478b77-gjzp8 -o yaml


> Delete the deployment and the horizontal pod autoscaler you created
    kubectl delete deploy nginx
    kubectl delete hpa nginx
    #Or
    kubectl delete deploy/nginx hpa/nginx

> Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio
```yaml
spec:
  type:
    strategy: RollingUpdate
    rollingUpdate:
     maxUnavailable: 1
     maxSurge: 1
  replicas: 2
```

# Rollout
> Check how the deployment rollout is going
    kubectl rollout status deploy nginx

> Check the rollout history and confirm that the replicas are OK
    kubectl rollout history deploy nginx
    kubectl get deploy nginx
    kubectl get rs # check that a new replica set has been created
    kubectl get po

> Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)
    kubectl rollout undo deploy nginx
    #. wait a bit
    kubectl get po # select one 'Running' Pod
    kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.18.0

> Verify that something's wrong with the rollout
    kubectl rollout status deploy nginx
    #. or
    kubectl get po # you'll see 'ErrImagePull' or 'ImagePullBackOff'
> Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8
    kubectl rollout undo deploy nginx --to-revision=2
    kubectl describe deploy nginx | grep Image:
    kubectl rollout status deploy nginx # Everything should be OK
> Check the details of the fourth revision (number 4)
    kubectl rollout history deploy nginx --revision=4

> Pause the rollout of the deployment
    kubectl rollout pause deploy nginx

> Resume the rollout
    kubectl rollout resume deploy nginx




> Update the nginx image to nginx:1.19.8
    kubectl set image deploy nginx nginx=nginx:1.19.8
    #. alternatively...
    kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image


# Job
> kubectl create job pi  --image=perl:5.34 -- perl -Mbignum=bpi -wle 'print bpi(2000)'

kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
or 
kubectl wait --for=condition=complete --timeout=300s job pi

Another way to terminate a Job is by setting an active deadline. Do this by setting the .spec.activeDeadlineSeconds field

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5 # to specify the number of retries before considering a Job as failed
  activeDeadlineSeconds: 100 # Terminate job after 100 seconds
  completions: 5 # Successful 5 completion will mark job complete
  parallelism: 5 # Runs 5 parrallel jobs
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never

```

> Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

> Get Job status
```bash
kubectl get jobs --watch
kubectl get po --show-labels
```

startingDeadlineSeconds: 17 # Terminate cron job if could not start in 17 seconds.

kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 # add this line
  jobTemplate:
    metadata:
      name: time-limited-job
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml

kubectl create job --from=cronjob/sample-cron-job sample-job

# Logs
> Follow logs
    kubectl logs busybox-ptx58 -f # follow the logs


# Observability

> Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it

kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    readinessProbe: # declare the readiness probe
      httpGet: # add this line
        path: / #
        port: 80 #
    livenessProbe: # our probe
      initialDelaySeconds: 5 # add this line
      periodSeconds: 5 # add this line as well
      exec: # add this line
        command: # command definition
        - ls # ls command
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```


Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

kubectl get events -o json | jq -r '.items[] | select(.message | contains("failed liveness probe")).involvedObject | .namespace + "/" + .name'


kubectl get events | grep -i error # you'll see the error here as well

kubectl delete po busybox --force --grace-period=0


> CPU Memory Utilization
    kubectl top nodes


https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md
