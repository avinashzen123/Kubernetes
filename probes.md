
Becasue of bugs, DB Connection failure, OOM etc we expect Pod to restart but kubernetes does not restart because by default Kubernetes checks container main process and decides if the Container is healthy or not it does not check internal functionality of our application 

Kubernetes probes let Kubernetes know application is not functioning as per our expectation and let Kubernetes restart it.

Probes: Investigates the Pod if they are working correctly or not

Kubernetes provie three kind of Probes
1. Liveness
2. Readiness
3. Startup

```yaml
livenessProbe:
  exec:
    command:
      - mongo
      - --eval
      - "db.adminCommadn('ping')"
```

if Liveness probe command return non zero value which indicates failure, kubernetes assumes pod unhealty and kublet restarts the pod.

Note : Liveness probe defined at container level.

Exec:
```yaml
exec:
  command:
    - mongo
    - --eval
    - "db.adminCommand('ping')"
```

Sucess : 0 Filure: 1

HTTP:
```yaml
httpGet:
  path: /health
  port: 8080
```

Success : 200-399 other than that failure


TCP:
```yaml
tcpSocket:
  port: 8080
```
Success : If port accepts traffice, Failure: if port can't accept traffic

# Probing customization

|                           | Purpose           | Default value |
|---------------------------|-------------------|---------------|
| initialDelaySeconds       | Delay to run the probe initially | 0 seconds |
| periodSeconds             | How frequently probe should execute after initial delay | 10 Seconds |
| timeoutSeconds            | Timeout period to mark as failure | 1 Second |
| failure/success Threshold | how many time to retry in case of failure | 3 times |


# livenessProbe

When livenessProbe probe fails kubernetes will restart pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mongodb
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    resources: {}
    env:
      - name: MONGO_INITDB_ROOT_USERNAME
        value: admin
      - name: MONGO_INITDB_ROOT_PASSWORD
        value: admin 
    livenessProbe:
      exec:
        command:
          - mongosh
          - --eval
          - "db.adminCommand('ping')"
      initialDelaySeconds: 1
      periodSeconds: 10
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 2
```

# readinessProbe

This probe identifies when container can handle external traffic received from a service. When readinessProbe fails Kubernetes remvoes IP address of POD from all services it belongs to. Once readiness  probe succeed IP is added back to service so that it can receive traffic.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mongodb
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    resources: {}
    env:
      - name: MONGO_INITDB_ROOT_USERNAME
        value: admin
      - name: MONGO_INITDB_ROOT_PASSWORD
        value: admin 
    readinessProbe:
      exec:
        command:
          - mongosh
          - --eval
          - "db.adminCommand('ping')"
      initialDelaySeconds: 1
      periodSeconds: 10
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 2
```

# startupProbe
livenessProbe and readinessProbe executes only when startupProbe succeeds. if startupProb fails then Pod is killed and is restarted as per restartPolicy



 