# Beekeeper Config

Configuration and deployment manifests for [Beekeeper](https://github.com/ethersphere/beekeeper) — the tool used to operate, validate, and exercise Bee clusters on Kubernetes.

This repo serves two purposes:

1. **`config/`** — Beekeeper YAML configs describing clusters, nodes, and the checks/loads/smokes/simulations that run against them.
2. **`deployments/`** — Helm values files that deploy Beekeeper into Kubernetes as Jobs or CronJobs against those clusters.

Beekeeper running in-cluster pulls these configs from this repo at runtime (`--config-git-repo`), so a change merged here ships to the next scheduled run.

## Repository layout

```
config/         # Beekeeper configs — one file per environment (cluster definition + checks)
deployments/    # Helm values for deploying Beekeeper jobs into K8s
  check/        # functional check jobs (pingpong, pushsync, manifest, pss, ...)
  smoke/        # long-running smoke tests
  load/         # load-generation jobs
  node-funder/  # funds node addresses with ETH/BZZ
  stamper/      # keeps postage stamps topped up
  mainnet/      # mainnet-specific jobs (SLA checks, dev-bee-perf, private bee, ...)
  testnet/      # testnet helpers
  debug/        # ad-hoc debug pods
```

Each environment has a matching config in `config/` consumed by Helm values files in `deployments/<kind>/`.

## Clusters

Today the repo targets:

| Cluster                | Config file                          | Used by                              |
| ---------------------- | ------------------------------------ | ------------------------------------ |
| `bee-testnet`          | `config/bee-testnet.yaml`            | public testnet checks / smoke / load |
| `bee-light-testnet`    | `config/bee-light-testnet.yaml`      | light-mode testnet                   |
| `bee-playground`       | `config/bee-playground.yaml`         | scratch / experiments                |
| `mainnet-sla`          | `config/mainnet-sla.yml`             | mainnet SLA monitoring               |
| `mainnet-dev-bee-*`    | `config/mainnet-dev-bee-*.yaml`      | dev mainnet performance / cheq tests |
| `dev-mainnet`          | `config/dev-mainnet.yaml`            | private-bee tooling                  |

## Deploying

All deployments use the [`ethersphere/beekeeper`](https://github.com/ethersphere/helm) Helm chart. From a target subdirectory:

```bash
helm repo add ethersphere https://ethersphere.github.io/helm
helm repo update

# install (or upgrade) a job
helm upgrade --install beekeeper-check-public ethersphere/beekeeper \
  --namespace beekeeper \
  -f deployments/check/beekeeper-check-public.yaml
```

Trigger a CronJob immediately instead of waiting for the schedule:

```bash
kubectl create job --from=cronjob/<name> <name>-immediate --namespace=beekeeper
```

Uninstall:

```bash
helm uninstall <release-name> --namespace=beekeeper
```

Each `deployments/<kind>/` directory has a README with the exact commands for the jobs in that folder.

## Beekeeper config format

Configs in `config/` are grouped into these top-level blocks:

- **`clusters`** — cluster definitions Beekeeper operates on
- **`node-groups`** — groups of Bee nodes sharing the same Kubernetes settings
- **`bee-configs`** — Bee node configuration that can be assigned to node-groups
- **`checks`** — checks Beekeeper can run against a cluster
- **`simulations`** — simulations Beekeeper can run against a cluster
- **`stages`** — stages for dynamic execution of checks and simulations

### Inheritance

Clusters, node-groups, and bee-configs support inheritance via `_inherit`:

```yaml
bee-configs:
  light-node:
    _inherit: default
    full-node: false   # only this field overrides the inherited config
```

### Action types

Checks and simulations can be declared multiple times with different parameters by changing the key while reusing the `type`:

```yaml
checks:
  pushsync-chunks:
    type: pushsync
    options:
      mode: chunks
      upload-node-count: 1
      # ...
  pushsync-light-chunks:
    type: pushsync
    options:
      mode: light-chunks
      upload-node-count: 1
      # ...
```

A Beekeeper run then selects the variant with `--checks=pushsync-chunks` or `--checks=pushsync-light-chunks`.

## Adding a new environment

1. Drop a new file in `config/` defining the cluster, its node-groups, bee-config, and the checks/smoke/load it should run.
2. Add the matching Helm values file in the relevant `deployments/<kind>/` directory, pointing `--cluster-name` and `--checks` at the new config and setting a `schedule:`.
3. Open a PR. Once merged on the branch referenced by the deployment (`--config-git-branch`), the next CronJob run picks it up.

## Maintainers

- [Bee](https://github.com/orgs/ethersphere/teams/bee) team
- [DevOps](https://github.com/orgs/ethersphere/teams/devops) team
