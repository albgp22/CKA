# Configuration

```yaml
spec.containers[].env[]
```

## ConfigMap

Options
- --from-literal
- --from-env-file
- --from-file (dir or single file)

### Env from CM
```yaml
spec:
  containers:
  - envFrom:
    - configMapRef:
        name: <name>
```

### Volume from CM
```yaml
spec:
  containers:
  - columeMounts:
    - name: <volume name>
      mountPath: <path>
  volumes:
  - name: <volume name>
    configMap:
      name: <CM name>
```

## Secret
Same options as with CM.

### Env from Secret
```yaml
spec:
  containers:
  - envFrom:
    - secretRef:
        name: <name>
```

### Volume from CM
```yaml
spec:
  containers:
  - columeMounts:
    - name: <volume name>
      mountPath: <path>
  volumes:
  - name: <volume name>
    secret:
      secretName: <secret name>
```