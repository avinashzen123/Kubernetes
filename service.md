In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster

The Service API, part of Kubernetes, is an abstraction to help you expose groups of Pods over a network. Each Service object defines a logical set of endpoints (usually these endpoints are Pods) along with a policy about how to make those pods accessible.

Need of Service:
- Kebernetes Pods are mortal
- Controller create and destroy Pods dynamically
- How to keep track of which IP adderss to connect to access the application
- Multiple tiers in application.

Type of Servies
* ClusterIP
* NodePort
* LoadBalancer
* External

Cluster IP : It exposes the service within the cluster only. It uses cluster internal IP which is not reachable from external network.

### Create a new ClusterIP service named my-cs
kubectl create service clusterip myweb-cs --tcp=80:32000    

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: myweb-cs
  name: myweb-cs
spec:
  ports:
  - name: 32000-80
    port: 32000
    protocol: TCP
    targetPort: 80
  selector:
    app: myweb
  type: ClusterIP
```
**kubectl get  service myweb-cs** 
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE   
myweb-cs   ClusterIP   10.110.231.155   <none>        32000/TCP   11m   
**kubectl get endpoints myweb-cs** 
NAME       ENDPOINTS                   AGE  
myweb-cs   10.1.1.37:80,10.1.1.38:80   11m  


> kubectl run ubuntu --image=ubuntu -it
    root@ubuntu:/# curl myweb-cs:32000

Default type is "ClusterIP" we don’t need to mention "name: ClusterIP" in spec.

Service Maps any incomingPort to targetPort. By default "targetPort" is set to "port" key.

Service will be created in same name space as the POD.


**NodePort Service**
- Exposed on each Nod's IP
- ClusterIP service is automatically created
- Accessible from external network on NodeIP:NodePort.

Along with NodePort Service a ClusterIP service is automatically created and the NodePort service routes the external traffic to the ClusterIP Service. 

> kubectl create deployment my-web --image=nginx --replicas=2

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: myweb-ns
  name: myweb-ns
spec:
  ports:
  - name: 32000-80
    port: 32000
    protocol: TCP
    targetPort: 80
    nodePort: 32100
  selector:
    app: myweb
  type: NodePort
```

minkube ssh
curl localhost:321   00

**LoadBalancer service**
- Exposes the service using a cloud provider's load balancer
- NodePort and ClusterIP services are automatically created.

**ExternalName service**
- Does not have selectors, instead uses DNS Names.
- No proxying of any kind is set up for external name.
- Use case for services without selectors.


#

**External IP**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 49152
  externalIPs:
    - 198.51.100.32
```

**Multi- port services**
- Expose more than one port to application through a service
- Must give names to all port in the manifest.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multiport
  labels:
    app: multiport
spec:
  containers:
  - name: tomcat
    image: tomcat
    ports:
      - containerPort: 8080
  - name: nginx
    image: nginx
    ports:
      - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: multiport
  ports:
  - port: 32000
    targetPort: 80
    name: nginxport
  - port: 33000
    targetPort: 8080
    name: tomcatport 
```

or 

kubectl expose deploy myweb --type=NodePort --port=80 --target-port=80


**kubectl get service**
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)               AGE   
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP               5m34s 
myapp        ClusterIP   10.100.216.176   <none>        32000/TCP,33000/TCP   4m20s 
**kubectl get endpoints**   
NAME         ENDPOINTS                     AGE  
kubernetes   192.168.65.3:6443             9m58s    
myapp        10.1.1.41:80,10.1.1.41:8080   8m44s    


# 

**Headless service**
Sometimes we don't need load balancing and a single IP address. In this case we can create 'Headless service' by explicitly specifying 'None' for the ClusterIP adderss (.spec.clusterIP)

Headless service allows us to reach each pod directly rather than the service acting as LoadBalancer. Headless service allows us to reach each pod directly rather than the service acting as LoadBalancer. We can created HeadlessService by explicitly specifying "none" for the ClusterIP under the  specification. For HeadlessService  a ClusterIP is not allocated, kube-proxy does not handle these service and there is no load balancing done by platform on them.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  clusterIP: None
  ports:
  - port: 32000
    targetPort: 80

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mystatefulset
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx-svc
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: naginx-cont
        image: nginx
        ports:
        - containerPort: 80
```

> kubectl run ubuntu --image=ubuntu -it

> kubectl exec mystatefulset-0 -- hostname -f
    mystatefulset-0.nginx-svc.default.svc.cluster.local

> curl mystatefulset-1.nginx-svc.default.svc.cluster.local
    From inside container it will print webpage.

### Create a new ClusterIP service named my-cs (in headless mode)
kubectl create service clusterip my-cs --clusterip="None"



# Ingress

- Ingress is used to provide external access to internal Kubernetes cluster resources. 
- To do so, Ingress uses a load balancer that is present on the external cluster
- This load balancer is implemented by the ingress controller.
- As an API resource, Ingress uses selector labels to connect to Pods that are used as a Service endpint.
- To access resource in the cluster, DNS must be configured to resolve to the ingress load balancer IP.
- Ingress exposes HTTP and HTTPS routes from outside the cluster to service within the cluster.
- Traffic routing is controlled by rules defined on the Ingress resource.
- Ingress can be configured to the following:
    * Give service externally reachable URLs
    * Load balance traffic
    * Terminate SSL/TLS
    * Offer name based virtual hosting.
- Creating ingress resources without ingress controller has not effect
- Many Ingress controller exists:
    * nginx :
    * haproxy
    * traefik
    * kong
    * contour

**Configuring minikube Ingress**
- minikube start
- minikube dashboard
- minikube addon enable ingress

After enabling we will have one more name space created named 'ingress-nginx'

Create deployment
Create service 
kubectl create ingress nginx-ingress --rule="/=nginxsvc:80" --rule="/hello=nginxdep:8080"

#

-- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward