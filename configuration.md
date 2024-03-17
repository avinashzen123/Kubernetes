ConfigMap is used to pas configuration data in form of Key value in Kubernetes

> kubectl create configmap <ConfigMap_Name> --from-literal=<Key>:<Value>
    kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod

```yaml
# app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod  
```

> kubectl create -f configMap.yaml

```yaml
# Refering in pod, we need envFrom -> configMapRef -> name
apiVersion: v1
kind: Pod
metadata:
  name: configmappod
spec:
  containers:
  - name: busyboxpod
    image: busybox
    command: ['/bin/sh']
    args: ['-c', 'while true; do echo $(date) > /dev/stdout; sleep 10; done']
    envFrom:
      - configMapRef:
          name: app-config
```

```yaml
# Referring single value we neee env-> name -> valueFrom -> configMapKeyRef
    env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: APP_COLOR
```

```yaml
# Mount configmap on volume
apiVersion: v1
kind: Pod
metadata:
  name: configmappod
spec:
  volumes:
  - name: app-config-volume
    configMap:
      name: app-config
  containers:
  - name: busyboxpod
    image: busybox
    command: ['/bin/sh']
    args: ['-c', 'while true; do echo $(date) > /dev/stdout; sleep 10; done']
    volumeMounts:
    - mountPath: /data/config
      name: app-config-volume
```

# Secrets 
Secrets allow for storage of sensitive data such as password, auth tokens and SSH Keys
Using secrets makes it so the data does not have to put in a Pod and reduces the risk of accidental exposure.
Some secrets are autoamtically created by the system, users can also use secrets
System created secrets are important for Kubernetes resources to connect to other cluster resources
Secrets are not encrypted they are base64 encoded.

Three types of secrets are offered:
- docker-registry : used for connecing to Docker registry
- TLS : used to store TLS key material
- generic : Creates a secret from local file, directory or literal values.

> kubectl create secret generic ....

Note : To access the Kubernetes API all kubernetes resources need to access to TLS keys

