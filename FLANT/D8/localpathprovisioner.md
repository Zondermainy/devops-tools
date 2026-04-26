https://deckhouse.ru/modules/local-path-provisioner/cr.html#localpathprovisioner

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: LocalPathProvisioner
metadata:
  name: localpath-monitoring
spec:
  nodeGroups:
  - monitoring
  path: "/mnt/monitoring"
```

