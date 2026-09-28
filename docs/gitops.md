# GitOps flow

```mermaid
sequenceDiagram
  Developer->>Git: Pull request with manifest change
  Git->>CI: YAML, schema, and secret checks
  CI-->>Developer: Required status checks
  Developer->>Git: Merge after review
  Argo CD->>Git: Poll repository revision
  Argo CD->>Kubernetes: Reconcile desired state
  Kubernetes-->>Argo CD: Health and sync status
```

Git is the source of truth. Argo CD uses a pull-based reconciliation loop with
prune and self-heal enabled for the root application. Direct changes made for an
incident are temporary and must be represented by a follow-up Git commit.
