# Scaling

## Manual scaling

```bash
k scale deployment <name> --replicas=<n>
k scale statefulset <name> --replicas=<n>
```

## Autoscaling

```bash
k autoscale deployment app-cache --cpu-percent=80 --min=3 --max=5
# Creates a HPA
```