# Reloader Helm Chart

Wrapper around [Stakater Reloader](https://github.com/stakater/Reloader). Watches ConfigMaps and Secrets and triggers rolling updates of Deployments, DaemonSets, StatefulSets. Annotate workloads with `reloader.stakater.com/auto: "true"` or `configmap.reloader.stakater.com/reload: <name>`.

## Install

```bash
helm dependency update
helm install reloader . -n cluster-tools --create-namespace
```
