# refold-deployment

Deployment configurations, IAM policies, and prerequisites for all Refold deployment models.

Reference material for teams deploying Refold into their own infrastructure — what you need to set
up before a deployment, and what to run alongside it.

## Contents

| Path | What it is |
|---|---|
| [BYOC/](BYOC/) | Cloud prerequisites for a Bring-Your-Own-Cloud deployment |
| [observability/](observability/) | SigNoz dashboards and alert rules to monitor your deployment |

### BYOC

[BYOC/aws/](BYOC/aws/) holds the two IAM documents you create in your AWS account so Refold can
deploy into it:

- **[managed-policy.json](BYOC/aws/managed-policy.json)** — the permission policy attached to the
  deployment role. Nine statements covering S3 and S3 Express, KMS, STS, EC2 and Elastic Load
  Balancing, IAM, CloudWatch Logs, EKS, and Auto Scaling.
- **[inline-policy.json](BYOC/aws/inline-policy.json)** — the role's trust policy: which principal
  may assume the role, gated on an `sts:ExternalId`.

### Observability

[observability/](observability/) is reference monitoring configuration to import into your own
SigNoz instance:

- **[dashboards/](observability/dashboards/)** — 7 dashboards. Import as-is; every environment
  filter is a dashboard variable populated from your data.
- **[alerts/](observability/alerts/)** — 79 alert rules covering the Refold services, Kubernetes
  workload health, MongoDB, Redis and persistent volumes. Substitute three placeholders
  (`<NAMESPACE>`, `<OBSERVABILITY_NAMESPACE>`, `<MONGO_CLUSTER>`) and attach your own notification
  channels before importing.

Each folder's README has the import commands, what the content assumes about your setup, and what to
review before relying on it.

## Notes

- **Reference, not a live mirror.** Nothing here tracks a running system automatically. The
  observability content is a curated snapshot (2026-09-22) of what Refold runs against its own
  deployments, adapted for a single-namespace on-prem install.
- **Adapt before you adopt.** Alert thresholds reflect Refold's capacity and traffic; IAM policies
  are the permissions a deployment needs, which your security review may want to scope further.
- **No credentials.** Notification channel configuration (webhook URLs, integration keys) is
  deliberately excluded. Instance URLs and API keys are supplied at run time via `$SIGNOZ_URL` /
  `$SIGNOZ_API_KEY`.
