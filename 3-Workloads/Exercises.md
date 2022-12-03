1.

```bash
k create deployment --image=nginx:1.17.0 --replicas 2 nginx
```

2.

```bash
k scale deployment nginx --replicas=7
```

3.

```yaml
 apiVersion: autoscaling/v2
 kind: HorizontalPodAutoscaler
 metadata:
   name: nginx-hpa
 spec:
   scaleTargetRef:
     apiVersion: apps/v1
     kind: Deployment
     name: nginx
   minReplicas: 3
   maxReplicas: 20
   metrics:
   - type: Resource
     resource:
       name: cpu
       target:
         type: Utilization
         averageUtilization: 65
   - type: Resource
     resource:
       name: memory
       target:
         type: AverageValue
         averageValue: 1Gi
```

4.

```bash
k set image deployment nginx nginx=nginx:1.21.1
k rollout history deployment nginx
kubectl rollout undo deployment nginx --to-revision=1
kubectl rollout history deployment nginx
```