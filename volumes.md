Containers are meant to be transient in nature which means that they are meant to last only for short period of time. 

***Acess Modes**
- ReadOnlyMany
- ReadWriteOnce
- ReadWriteMany


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