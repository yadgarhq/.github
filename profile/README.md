# yadgar

[![yaadgaar](https://img.shields.io/github/v/tag/yadgarhq/chart?sort=semver&label=yaadgaar)](https://github.com/yadgarhq/chart/tags)
[![client](https://img.shields.io/github/v/tag/yadgarhq/yadgar?sort=semver&label=client)](https://github.com/yadgarhq/yadgar/tags)
[![gateway](https://img.shields.io/github/v/tag/yadgarhq/gateway?sort=semver&label=gateway)](https://github.com/yadgarhq/gateway/tags)

yadgar (Persian for "memento, keepsake") is a persistent memory engine for AI
coding agents that works with any MCP client. This organisation builds its
successor architecture: Rust, one repository per module, deployed as
microservices. Clients speak MCP to one edge, the gateway. Behind it, small
services speak gRPC. Each module has a logic service and a `-db` twin that is
the only writer of its store.

The successor modules are built, except the audit store, which is designed but
not built. They run on a development cluster, not as a hosted service. The
original memory engine is a separate project,
[`m-agahi/yadgar`](https://github.com/m-agahi/yadgar). The successor starts with
empty databases and migrates nothing from it.

## Architecture at a glance

| Area | Repositories |
| ---- | ------------ |
| Client | [`yadgar`](https://github.com/yadgarhq/yadgar): the `yaadgaar` binary, an MCP proxy and CLI |
| Edge | [`gateway`](https://github.com/yadgarhq/gateway): MCP/JSON outward, gRPC inward |
| Identity | [`iam`](https://github.com/yadgarhq/iam) and [`iam-db`](https://github.com/yadgarhq/iam-db): users, teams and credentials |
| Tasks | [`task`](https://github.com/yadgarhq/task) and [`task-db`](https://github.com/yadgarhq/task-db) |
| Projects | [`project`](https://github.com/yadgarhq/project) and [`project-db`](https://github.com/yadgarhq/project-db): the project registry |
| Seeding | [`seeder`](https://github.com/yadgarhq/seeder): the one writer of org-class project rows |
| Contract | [`proto`](https://github.com/yadgarhq/proto): the shared `.proto` files, gated by `buf breaking` |
| Shared crates | [`dial`](https://github.com/yadgarhq/dial) (client-side balancing), [`lifecycle`](https://github.com/yadgarhq/lifecycle) (start, stop, restart), [`telemetry`](https://github.com/yadgarhq/telemetry) (spans, wide events, metrics), [`store`](https://github.com/yadgarhq/store) (pool, transactions, migrations) |
| Deployment | [`chart`](https://github.com/yadgarhq/chart) (parent Helm chart), [`config`](https://github.com/yadgarhq/config) (one ConfigMap per service), [`platform`](https://github.com/yadgarhq/platform) (CA chain, Gateway listener, Valkey, NATS, bootstrap Jobs) |
| Delivery | [`actions`](https://github.com/yadgarhq/actions) (shared CI workflows and pre-commit hooks), [`argocd`](https://github.com/yadgarhq/argocd) (GitOps control plane), [`estate`](https://github.com/yadgarhq/estate) (behavioural contract suite and parity auditor) |

The design record (decisions, invariants, install runbook) lives in a private
repository. Public READMEs cite its decisions by number, for example D16 and
ADR-0706.

## Install

One parent Helm chart installs the whole estate. It declares every module chart
as a dependency pinned to a released version. The chart is published to
`ghcr.io` and pulls without credentials:

```sh
helm show chart oci://ghcr.io/yadgarhq/charts/yadgar
```

An adopter commits one Argo CD `Application` that points at
`ghcr.io/yadgarhq/charts`, chart `yadgar`, at one version. The minimum values
are your hostname and your edge certificate issuer. The
[`chart` README](https://github.com/yadgarhq/chart#readme) has the full
`Application` example, the values, and a one-command install on a kind cluster.

## Status and licence

- The shared [`proto`](https://github.com/yadgarhq/proto) contract is v1 and not
  yet ratified. It changes additively.
- Every active public repository is licensed under Apache-2.0; each carries
  the same `LICENSE` file.
