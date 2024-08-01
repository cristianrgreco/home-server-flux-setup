# Home Server Flux Setup

## Useful commands

Show status and installed/available addons.
```bash
microk8s status
```

Watch status. Useful after git pushing to the bootstrap repo.
```bash
flux get kustomizations --watch
```

Watch all events.
```bash
flux events --watch
```

Get status of helm releases. Useful to see what's happening. Note that `helmreleases` can be shortened to `hr`.
```bash
flux get helmreleases --all-namespaces
```

Trigger a reconciliation without waiting for the next interval.
```bash
flux reconcile hr -n minecraft minecraft
```

Suspend a release so you can make temporary changes.
```bash
flux suspend helmrelease -n minecraft minecraft
```

Resumes a suspended helm release.
```bash
flux resume helmrelease -n minecraft minecraft
```

## To do

1. 
2. Set `grafana.adminPassword` to anything non default.
   1. Consider using sealed secrets so you can get the value from k8s.
3. Decide whether persistence is required for Grafana. 
   1. As we only use the default dashboards possibly not.
4. Enable Minecraft backups
   https://github.com/itzg/minecraft-server-charts/blob/master/charts/minecraft/values.yaml#L435.
5. Enable Minecraft whitelist to restrict who can join.
   1. Avoids griefers.
