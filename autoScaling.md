Kinds of autoscaling

- HPA (Horizontal Pod autoscalar) : Will adjust number of pods (increase/decrease)
- VPA (Verical Pod Autoscalar) : Will adjust resources (CPU/Memroy) as per demand of running pod instead of creating new one
- CA (Cluster autoscalar) : Will add nodes to the cluster to accommodate more number of pods. Will add ndoes if any pod is stuck because of lack or resources in the cluster


```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```