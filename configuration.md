ConfigMap is used to pas configuration data in form of Key value in Kubernetes

**Imperative way**

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
**Declarative way**
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

**Imperative way**
> kubectl create secret generic <Secret_name> --from-literal=<key>=<value>
    kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=password

**Declarative way**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_Host: bXlzcWwK
  DB_User: cm9vdAo=
  DB_Password: cGFzc3dvcmQK
```

> kubectl create -f secrets.yaml

Note : we must provide Encoded values in manifest file  


```yaml
# Refering in pod, we need envFrom -> secretRef -> name
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
      - secretRef:
          name: app-config
```


```yaml
# Referring single value we need env-> name -> valueFrom -> secretKeyRef
    env:
      - name: APP_COLOR
        valueFrom:
          secretKeyRef:
            name: app-config
            key: APP_COLOR
```

```yaml
# Mount secret on volume
apiVersion: v1
kind: Pod
metadata:
  name: secretpod
spec:
  volumes:
  - name: app-secret-volume
    configMap:
      name: app-config
  containers:
  - name: busyboxpod
    image: busybox
    command: ['/bin/sh']
    args: ['-c', 'while true; do echo $(date) > /dev/stdout; sleep 10; done']
    volumeMounts:
    - mountPath: /data/config
      name: app-secret-volume
```

Note:
- Secrets are not encrypted only encode which can be decoded easily.
- Secrets are not encrypted in 'etcd' by default. So consider enabling encryption at rest 
- Anyone able to create pod/deployment in the same namespace can access the secrets. Configure least privilege access to secret - Role based access control

Consider third party secret providers : AWS Provider, Azure provider, GCP Provider, Vault provider.



# SecurityContext
Container security can be configured in Kubernetes as well. As we know containers are encapsulated within a pod. So we configure the security setting at pod level or container level
If we configure security at pod level, these setting will carry over to all container within the pod.

If we configure at both pod and container level, the setting of container will override setting on pod.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
spec:
  # securityContext:
  #   fsGroup: 0
  containers:
  - name: nginxcont
    image: nginx
    command: ["/bin/sh"]
    args: ["-c","while true; do echo hello; touch \"/var/test$(date).text\"; sleep 10;done"]
    resources: {}
    securityContext:
      # capabilities:
      #   add: ["SYS_TIME"]
      # privileged: true
      # runAsUser: 0
      # runAsGroup: 0
      # allowPrivilegeEscalation: false
```

# Service account
There are two types of account in Kubernetes:
- User account : used by human
- Service account : Used by machine. An account used by application to interact with Kubernetes cluster.

> kubectl create serviceaccount dashboard-sa

> kubectl get serviceaccount

> kubectl describe serviceaccount dashboard-sa

> kubectl create token dashboard-sa
    To create token

> kubectl set serviceaccount deploy/web-dashboard dashboard-sa
    To set service account of a deployment

Service account token is what we must be used by external application when interacting 

kubectl describe pod <pod_name> command will give us 'Mounts' information where security token is stored

kubectl exec -it my-pod -- ls /var/run/secrets/kubernetes.io/serviceaccount

This is JWT token 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /var/run/secrets/tokens
      name: vault-token
  serviceAccountName: build-robot
  volumes:
  - name: vault-token
    projected:
      sources:
      - serviceAccountToken:
          path: vault-token
          expirationSeconds: 7200
          audience: vault
```


