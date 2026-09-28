# Private RKE2 Kubernetes Platform

Production-shaped private Kubernetes platform for the Hosting Mosalam intern lab.
The repository separates cluster infrastructure from application delivery:

```text
infra/                 RKE2, Cilium, Gateway API and storage bootstrap
bootstrap/              Argo CD installation and root Application
gitops/apps/            Application manifests managed by Argo CD
docs/                   Architecture, networking, delivery and failure evidence
```

## Target topology

| Node | Role | Control IP | Pod-network IP |
| --- | --- | --- | --- |
| `cp-1` | etcd + control plane | `<control-ip-1>` | `<pod-ip-1>` |
| `cp-2` | etcd + control plane | `<control-ip-2>` | `<pod-ip-2>` |
| `cp-3` | etcd + control plane | `<control-ip-3>` | `<pod-ip-3>` |

Each VM has three interfaces: control, pod, and NAT. The two internal networks use
MTU 1400. NAT is only for package and image downloads; node-to-node traffic must
use the control or pod network addresses.

## Delivery model

1. Provision three VMs and configure the internal routes and MTU.
2. Install RKE2 server nodes using `infra/rke2/server-config.yaml.example`.
3. Install Cilium with native routing and kube-proxy replacement using
	`infra/cilium/values.yaml`.
4. Install Gateway API CRDs, Argo Rollouts, Argo CD, and the root Application.
5. Change application manifests only through pull requests under `gitops/apps`.
6. Argo CD reconciles the merged Git revision. No direct `kubectl apply` is part
	of the normal operating procedure.

Detailed decisions and runbooks are in [docs/architecture.md](docs/architecture.md),
[docs/networking.md](docs/networking.md), and [docs/operations.md](docs/operations.md).

## Local checks

The pull request pipeline runs YAML linting, Kubernetes schema validation, and
checks that manifests do not contain obvious plaintext credentials:

```bash
yamllint .
kubeconform -strict -summary -schema-location default -schema-location 'https://raw.githubusercontent.com/argoproj/argo-rollouts/master/manifests/crds/{{ .ResourceKind }}.yaml' gitops
```

The cluster-specific values in `infra` and the external DNS name are intentionally
placeholders. Replace them in a private environment configuration before rollout;
do not commit real tokens, private keys, or kubeconfigs.
