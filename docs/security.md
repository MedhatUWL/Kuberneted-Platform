# Security baseline

- Keep RKE2 tokens, kubeconfigs, TLS keys, and database passwords outside Git.
- Use a secret manager, External Secrets, or Sealed Secrets for application credentials.
- Enforce the Restricted Pod Security profile on application namespaces.
- Use non-root containers, dropped Linux capabilities, resource limits, and pinned image tags.
- Restrict Gateway routes by namespace and hostname; expose only required listeners.
- Review Argo CD project destinations and source repositories before granting sync rights.
- Add network policies and image scanning as the application surface grows.
