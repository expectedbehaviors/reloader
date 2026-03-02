# Reloader Helm Chart

Wrapper around [Stakater Reloader](https://github.com/stakater/Reloader). Watches ConfigMaps and Secrets and triggers rolling updates of Deployments, DaemonSets, StatefulSets. Annotate workloads with `reloader.stakater.com/auto: "true"` or `configmap.reloader.stakater.com/reload: <name>`.

## Subcharts

| Subchart | Source | Values prefix | Description |
|----------|--------|---------------|-------------|
| **reloader** | [Stakater stakater-charts](https://github.com/stakater/charts) | `reloader.*` | Reloader controller: watch options, deployment, securityContext. |
| **onepassworditem** | [expectedbehaviors/OnePasswordItem-helm](https://github.com/expectedbehaviors/OnePasswordItem-helm) | `onepassworditem.*` | Optional secrets sync into the release namespace. |

All inputs: **`reloader.*`** (see Stakater chart values), **`onepassworditem.enabled`**, **`onepassworditem.items`**. Defaults: see `values.yaml`.

## What it does

- Deploys the [Reloader](https://docs.stakater.com/reloader) controller from the official Stakater chart.
- When a ConfigMap or Secret used by an annotated workload is updated, Reloader performs a rolling restart of that workload so it picks up the new config.
- Workloads opt in via annotations: `reloader.stakater.com/auto: "true"` for auto-discovery, or `configmap.reloader.stakater.com/reload: "my-cm"` / `secret.reloader.stakater.com/reload: "my-secret"` for specific resources.

## Prerequisites

- **1Password Connect** (optional): if you use `onepassworditem.enabled: true` to sync secrets used by annotated workloads. The chart depends on [expectedbehaviors/OnePasswordItem-helm](https://github.com/expectedbehaviors/OnePasswordItem-helm). Set `onepassworditem.enabled: false` if you do not need 1Password.

## Key values (reloader subchart)

| Key | Purpose |
|-----|---------|
| `reloader.autoReloadAll` | If `true`, reload on any ConfigMap/Secret change for annotated workloads (default in this wrapper: `true`). |
| `reloader.ignoreConfigMaps` / `reloader.ignoreSecrets` | Set to `true` to skip watching ConfigMaps or Secrets (default: `false`). |
| `reloader.reloadOnCreate` | Reload when a referenced ConfigMap/Secret is created (default: `true`). |
| `reloader.ignoreNamespaces` | Namespaces to skip (e.g. `kube-system,argo-cd`); default in wrapper may exclude system namespaces. |
| `reloader.deployment.securityContext` | This wrapper sets a hardened pod/container security context (non-root, read-only rootfs, drop all caps). |

See [Reloader docs](https://docs.stakater.com/reloader) for more options (e.g. `reloadStrategy`, `namespaceSelector`, metrics, HA). **isArgoRollouts:** Set `reloader.isArgoRollouts: true` only if you use the [Argo Rollouts](https://argoproj.github.io/argo-rollouts/) CRD; otherwise leave `false`. (Argo CD is unrelated — it deploys this chart.)

## Values (onepassworditem)

| Key | Description |
|-----|-------------|
| `onepassworditem.enabled` | If `true` (default), the subchart is enabled. Set `false` if you do not use 1Password for secrets. |
| `onepassworditem.items` | List of `{ item, name, type }` for secrets to sync into the release namespace. |

## Install

**From this repo:**

```bash
helm dependency update
helm install reloader . -n cluster-tools --create-namespace
```

**From Helm repo (expectedbehaviors):**

```bash
helm repo add expectedbehaviors https://expectedbehaviors.github.io/reloader
helm install reloader expectedbehaviors/reloader -n cluster-tools --create-namespace
```

## Version and upgrade

- **Wrapper:** version in Chart.yaml. **Dependency:** reloader from [Stakater Helm repo](https://github.com/stakater/charts); see Chart.yaml.
- **Upgrade:** Bump the `version` under `dependencies` in Chart.yaml, run `helm dependency update`, then upgrade the release.

## Security

This wrapper sets a hardened pod/container security context: non-root (runAsUser 65534), read-only root filesystem, all capabilities dropped, RuntimeDefault seccomp. Adjust under `reloader.deployment.securityContext` and `reloader.deployment.containerSecurityContext` if needed.

## Argo CD

Deploy as an Argo CD Application. Point the Application at this chart (repo path and target revision) and set the destination namespace (e.g. `reloader` or `cluster-tools`). No special Reloader config required for Argo CD.

## Future: Prometheus / PodMonitor

When Prometheus (and the Prometheus Operator) is available, enable Reloader metrics by setting `reloader.serviceMonitor.enabled` or `reloader.podMonitor.enabled: true` and configuring labels so your Prometheus selects the monitor.

## Template filenames

This chart wraps the Stakater Reloader subchart; template filenames follow the upstream chart. The wrapper adds no custom resource templates.
