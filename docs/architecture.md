# Architecture

```mermaid
flowchart TB
  User --> DNS[External DNS]
  DNS --> GW[Gateway API / Cilium]
  GW --> Route[HTTPRoute]
  Route --> Rollout[Argo Rollouts]
  Rollout --> App[platform-demo]
  App --> DB[(PostgreSQL PVC)]
  Git[Git repository] --> Argo[Argo CD]
  Argo --> Rollout
  Cilium[Cilium eBPF] --> Nodes[3 RKE2 servers]
  Nodes --> Etcd[(Embedded etcd quorum)]
```

Three control-plane nodes provide an etcd quorum of two and tolerate one node
failure. RKE2 servers use the control-network IP for the supervisor and API
traffic. Cilium is the sole CNI, uses native routing, and replaces kube-proxy.

The platform boundary is deliberate: RKE2, Cilium, CoreDNS, and kube-system
components are bootstrapped operationally; application resources live under
`gitops/apps` and are reconciled by Argo CD.
