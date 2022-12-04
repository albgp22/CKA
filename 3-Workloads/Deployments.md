# Deployments

```bash
k create deploy <name> --image=<img> --replicas=<n> -n <ns>
```

Mapping between Deployment and Pod via labels. In

- metadata.labels
- spec.selector.matchLabels
- spec.template.metadata.labels

ReplicaSet created by Deployment instance.

> k get deploy,po,rs