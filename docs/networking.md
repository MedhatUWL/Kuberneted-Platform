# Network design

Each VM has three interfaces:

| Network | Purpose | Routing rule |
| --- | --- | --- |
| Control | RKE2 supervisor, Kubernetes API, etcd | Private node-to-node only, MTU 1400 |
| Pod | Cilium native pod routes | Private pod and node traffic, MTU 1400 |
| NAT | Downloads and image pulls | Egress only; never a cluster peer address |

Native routing avoids the extra encapsulation overhead of VXLAN. Nodes advertise
pod CIDR routes on the pod network, so a packet travels node-to-node without an
encapsulation header. VXLAN would work where the underlay cannot route pod CIDRs,
but costs MTU and adds tunnel processing. With MTU 1400 on the lab underlay,
Cilium is configured to use 1400 and tunnel mode is disabled.

Before joining nodes, verify `ip route`, interface MTUs, and that `cp-1` can reach
ports 9345 and 6443 on the control addresses. A failed NAT ping must not be used as
evidence that cluster networking is broken.

Gateway API separates listener ownership (Gateway) from application routing
(HTTPRoute). Namespaces can be isolated with `allowedRoutes`; multiple tenants can
receive distinct hostnames and routes while a platform team owns the GatewayClass.
