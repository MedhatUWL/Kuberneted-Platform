# Operations and failure evidence

## Bootstrap order

1. Provision three VMs across distinct failure domains with the three interfaces.
2. Set internal interface MTU to 1400 and validate private routes.
3. Install RKE2 on `cp-1`, then join `cp-2` and `cp-3` with the same token.
4. Install Cilium, Gateway API CRDs, Argo Rollouts, and Argo CD.
5. Install External Secrets and configure `platform-secret-store` for the lab secret backend.
6. Replace repository placeholders, register the repo in Argo CD, and sync the root app.

## Progressive delivery

Change the image in `rollout.yaml` through a pull request. Argo Rollouts updates
10%, pauses, then 25%, 50%, and 100% through the Gateway API plugin. Observe each
step with `kubectl argo rollouts get rollout platform-demo -n platform-demo --watch`.
Abort a bad release with `kubectl argo rollouts abort platform-demo -n platform-demo`;
the controller restores stable traffic. The command is an emergency operation;
record it and follow up with a Git revert.

## Failure test matrix

| Test | Expected result | Evidence to capture |
| --- | --- | --- |
| Stop one control-plane VM | etcd keeps quorum; API remains available | node status, etcd health, request result |
| Delete an app pod | Deployment/Rollout recreates it | pod events and recovery time |
| Push a deliberately bad image | rollout pauses or fails; stable service remains | Rollout conditions and Gateway response |
| Abort canary | traffic returns to stable revision | rollout history and HTTP checks |
| Restart database pod | StatefulSet remounts its PVC | pod UID, PVC, and row count |
| Move database node | local-path data is node-bound; document limitation | scheduling events and storage status |

The Longhorn volume demonstrates persistence across pod restart and supports
replica-based recovery after a node failure. Verify the Longhorn replica count and
health before claiming data availability; a PVC being Bound alone is not proof of
replication.
