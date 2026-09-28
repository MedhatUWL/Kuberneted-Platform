# Observability

Cilium Hubble is enabled in `infra/cilium/values.yaml` for service dependency,
flow, and network-policy visibility. The first operational checks are:

```bash
hubble status -P
hubble observe -P --namespace platform-demo
kubectl get --raw /healthz
kubectl get pods -A -o wide
```

The platform should add a Prometheus-compatible metrics stack and Grafana through
Argo CD once the lab chooses retention and storage sizing. Application alerts
should cover rollout health, API error rate, request latency, pod restarts, PVC
capacity, and etcd quorum. Keep dashboards and alert rules in Git alongside the
application they describe.

The HTTPS Gateway listener expects a `demo-tls` Secret created by the certificate
manager or secret platform. The certificate and private key must never be stored
in this repository.
