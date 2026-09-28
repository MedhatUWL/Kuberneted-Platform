# RKE2 bootstrap

Install the first server from the official RKE2 channel, then place the example
config at `/etc/rancher/rke2/config.yaml`:

```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=server sh -
systemctl enable --now rke2-server
```

On `cp-2` and `cp-3`, add `server: https://<cp-1-control-ip>:9345` and use the
same private token. For a production lab, put the API endpoint behind a stable
VIP or DNS record and verify that all peer addresses resolve to the control
network, never the NAT interface.

After the three etcd members are healthy, install Cilium from its pinned Helm
chart using `values.yaml`, then install Gateway API and the platform operators.
