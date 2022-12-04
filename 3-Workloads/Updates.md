# Updates

## Rollout

Either using 

```bash
k edit deploy
```

or 

```bash
k set image deployment <name> <img>=<img:ver> --record 
```

### Status:

```bash
k rollout status deployment <name>
k rollout history deployment <name>
k rollout history deployments <name> --revision=2
```

## Rollback

```bash
k rollout undo deployment <name> --to-revision=1
```

