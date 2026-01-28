# Continuous Deployment

Simplismart uses ArgoCD for GitOps-based continuous deployment, enabling declarative, version-controlled deployment management.

---

## Overview

Continuous Deployment in Simplismart provides:

- **Automatic sync**: Changes are automatically deployed
- **Drift detection**: Alert when actual state differs from desired
- **Rollback capability**: Easily revert to previous versions
- **Audit trail**: Full history of deployment changes

---

## Deployment Sync

### Automatic Sync

When enabled, ArgoCD automatically syncs changes by doing rolling updates for:

- New container versions
- Configuration updates
- Scaling adjustments
- Environment variable changes

---

## Related Documentation

- [Deployment Lifecycle](../deployments/lifecycle.md)
- [Health Checks](./health-checks.md)
- [Monitoring](./monitoring.md)
- [Autoscaling](./autoscaling.md)
