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


  
### Create a new ClusterIP service named my-cs (in headless mode)
kubectl create service clusterip my-cs --clusterip="None"

-- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward