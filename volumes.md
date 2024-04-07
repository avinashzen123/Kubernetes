Containers are meant to be transient in nature which means that they are meant to last only for short period of time. 


To use Volume, speicfy the volumes to provide for the Pod in .spec.volumes and declare where to mount those into a containers in .spec.containers[*].volumeMounts.

Types of volume:
<details>
<summary>ConfigMap</summary>
A ConfigMap provides a way to inject configuration data inot pods. The data stored in a ConfigMa can be referenced in a volume of type 'configMap' and the consumed by containerized application running in a Pod.

kubectl create configmap log-config --from-literal=log_level=DEBUG

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```
</details>
<details>
<summary>Secret</summary>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      command:
        - ls
        - "-l"
        - "/etc/secret-volume"
      volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"

```
</details>

<details>
<summary>emptyDir</summary>
For a Pod that defines an emptyDir volume, the volume is created when the Pod is assigned to a node. As the name says, the emptyDir volume is initially empty. All containers in the Pod can read and write the same files in the emptyDir volume, though that volume can be mounted at the same or different paths in each container. When a Pod is removed from a node for any reason, the data in the emptyDir is deleted permanently.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: emptydirvol
  name: emptydirvol
spec:
  volumes:
  - name: cache
    emptyDir: 
      sizeLimit: 100Mi
  containers:
  - image: busybox
    name: emptydirvol
    command: ['/bin/sh', '-c', 'while true; do echo "Print From vol" && tail -f /dev/null; done;']
    volumeMounts:
    - name: cache
      mountPath: /cache
```
</details>


<details>
<summary> hostPath </summary>
A hostPath volume mounts a file or directory from the host node's filesystem into your Pod

Using the hostPath volume type presents many security risks. If you can avoid using a hostPath volume, you should. For example, define a local PersistentVolume, and use that instead.

hostPath volume types;

--------------------------
|Value | Behavior          |
-------|-------------------
|""                | Default is for backward compatibility, which means no check will be performed before mounting the hostPath volume |
| DirectoryOrCreate | If nothing exist at given paht, an empty directory will be created there as needed with permission set of 0755,  having the same group and ownership with Kubelet  |
| Directory         | A directory must exist at given path |
| FileOrCreate      | If nothing exist at given path, an empty file will be created there as needed with permssion set of 0644, having the same group and ownership with Kubelet | 
| File              | A file must exist at given path |
| Socket            | A Unix socket must exist at given path | 
| CharDevice        | (Linux nodes only) A character device must exist at given path |
| BlockDevice       | (Linux nodes only) A block device must exist at the given path |


```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: hostpathvol
  name: hostpathvol
spec:
  volumes:
  - name: hostvol
    hostPath:
      path: /data/foo
      type: DirectoryOrCreate
  containers:
  - image: nginx
    name: hostpathvol
    resources: {}
    command: [ '/bin/sh', '-c', 'while sleep 10; do echo $(date) >> /usr/share/nginx/html/index.html; done;' ]
    ports:
    - containerPort: 80
    volumeMounts:
    - name: hostvol
      mountPath: /usr/share/nginx/html
```
</details>

<details>
<summary>local</summary>
A local volume represents a mounted local storage device such as a disk, partition or directory.

Local volumes can only be used as a statically created PersistentVolume. Dynamic provisioning is not supported.

Compared to hostPath volumes, local volumes are used in a durable and portable manner without manually scheduling pods to nodes. The system is aware of the volume's node constraints by looking at the node affinity on the PersistentVolume.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

You must set a PersistentVolume nodeAffinity when using local volumes. The Kubernetes scheduler uses the PersistentVolume nodeAffinity to schedule these Pods to the correct node.

</details>

# 
A **PersistentVolume (PV)** is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

A **PersistentVolumeClaim (PVC)** is a request for storage by a user.

There are two ways PVs may be provisioned: statically or dynamically.

**Acess Modes**
- ReadOnlyMany
- ReadWriteOnce
- ReadWriteMany
- ReadWriteOncePod

**Reclaim Policy**
* Retain -- manual reclamation
* Recycle -- basic scrub (rm -rf /thevolume/*)
* Delete -- delete the volume

**Phase**:
Available : a free resource that is not yet bound to a claim
Bound : the volume is bound to a claim
Released : the claim has been deleted, but the associated storage resource is not yet reclaimed by the cluster
Failed : the volume has failed its (automated) reclamation

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

```yaml
# Empty dir
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2 # don't forget to change the name during copy paste, must be different from the first container's name!
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  volumes: #
  - name: myvolume #
    emptyDir: {} #
```


# 

There are two different set of objects in Kubernetes cluster.
- Administrator creates Persistent Volume
- User Creates PersistentVolumeClaims

Once the PersistentVolume claims are created, Kubernetes binds the PersistentVolume to Claims based on request and properties set on volume.

PersistentVolumeClaim is bound to a Single PersistentVolume

During the binding process Kubernetes tries to find a PersistentVolume that has sufficient capacity as requested by Claim and other requested properties such as 'accessMode', 'volumeModes', 'storageClass'

if we have multiple possible matches for a claim and we would like to specifically use a particular volume we could still use 'labels' and 'slectors' to bind the right target.

```yaml
# PV
labels:
  name: my-pv
```

```yaml 
# pvc
selector:
  matchLabels:
    name: my-pv
```

Smaller claims may get bound to larger volume if all the other criteria matches and tehre is not better options. 

If there is no volume available then PersistentVolumeClaim will remain in pending state until newer volume are made available in the cluster.


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

**persistentVolumeReclaimPolicy**
* Delete
* Recycle



# Code

```yaml
# Volume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-vol
spec:
  accessModes:
    - ReadWriteMany
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: normal
  hostPath:
    path: /data
```

```yaml
# Claim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
```

```yaml
# Pod
apiVersion: v1
kind: Pod
metadata:
  name: pvcpod
spec:
  volumes:
  - name: datavolume
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - name: busyboxcont
    image: busybox
    command: ['/bin/sh']
    args: ['-c','while true; do echo $(date) >> /data/dates.txt; sleep 10; done;']
    volumeMounts:
    - mountPath: /data
      name: datavolume
```

#


```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: podvolume
  name: podvolume
spec:
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
  containers:
  - image: busybox
    name: podvolume
    command: ['/bin/sh']
    args: ['-c', "while true; do echo $(date) >> /data/dates.txt; sleep 10; done"]
    volumeMounts:
    - mountPath: /data
      name: data-volume
```

Multi container volume share emptyDir

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: podvolume
  name: podvolume
spec:
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
  containers:
  - image: busybox
    name: podvolume
    command: ['/bin/sh']
    args: ['-c', "while true; do data > /data/dates.txt; sleep 10; done"]
    volumeMounts:
    - mountPath: /data
      name: data-volume
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```


```yaml
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```