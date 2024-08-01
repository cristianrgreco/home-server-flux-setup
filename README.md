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

1. Enable Minecraft backups
   https://github.com/itzg/minecraft-server-charts/blob/master/charts/minecraft/values.yaml#L435.
