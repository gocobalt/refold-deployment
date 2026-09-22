# Alert Rules

Reference alert rules for a Refold on-prem deployment, for import into your own SigNoz instance.
One JSON file per rule — **79 rules** covering the Refold services, Kubernetes workload health, the
data stores, and persistent volumes.

These are derived from the rules Refold runs against its own deployments, with environment-specific
values replaced by placeholders. They are a starting point: review the thresholds against your own
capacity and traffic before relying on them.

## Before importing

Substitute these placeholders in every file you import:

| Placeholder | Replace with | Appears in |
|---|---|---:|
| `<NAMESPACE>` | The Kubernetes namespace Refold is deployed into | 199 selectors |
| `<OBSERVABILITY_NAMESPACE>` | The namespace your SigNoz/ClickHouse runs in | 18 selectors |
| `<MONGO_CLUSTER>` | The `cluster` label your MongoDB exporter reports | 8 selectors |

```bash
# substitute and import every rule
BASE="$SIGNOZ_URL"; KEY="$SIGNOZ_API_KEY"
NAMESPACE=refold; OBSERVABILITY_NAMESPACE=observability; MONGO_CLUSTER=refold

for f in alerts/*.json; do
  sed -e "s/<NAMESPACE>/$NAMESPACE/g" \
      -e "s/<OBSERVABILITY_NAMESPACE>/$OBSERVABILITY_NAMESPACE/g" \
      -e "s/<MONGO_CLUSTER>/$MONGO_CLUSTER/g" "$f" \
    | curl -s -X POST "$BASE/api/v1/rules" \
        -H "SIGNOZ-API-KEY: $KEY" -H "Content-Type: application/json" --data @-
done
```

`preferredChannels` is empty in every rule — nothing routes anywhere until you set it. Create your
notification channels in SigNoz first, then add their names to each rule (or set them in the UI
after import). Severity is carried on the `severity` label (`1` = page, `2` = urgent, `3` = ticket)
so you can route on it.

## What the rules assume

- **Metric sources.** `kube-state-metrics` and cAdvisor (`kube_*`, `container_*`, `kubelet_*`) for
  workload and volume health; the Refold services' own `/metrics` endpoints scraped into SigNoz
  (`nodejs_*`, `up`, `process_*`); the MongoDB and Redis Prometheus exporters. A rule whose metrics
  you do not collect will simply never fire.
- **Workload names.** Rules select Refold's deployments by name (`audit-service-v2`,
  `bkout-integration-auth-backend`, `bkout-one-workflow-backend`, `integration-service`,
  `workflow-queue-workers`, `ai-python-app`, `ai-python-worker`). These come from the Refold Helm
  chart and are the same in any deployment — no substitution needed unless you have renamed them.
- **Single namespace.** Each rule scopes to one `<NAMESPACE>`. If you run several Refold
  environments in one cluster, import one copy per namespace, or widen the selector to a regex.
- **Runbook text.** Every rule carries a `summary` and a `resolution` annotation with the `kubectl`
  steps to triage it. Adapt them to your own procedures.

## The rules

### Refold services (54)

| Rule | Severity | Fires when | Eval / freq |
|---|---|---|---|
| [AuditCpuHigh](AuditCpuHigh.json) | sev2 | audit pod … CPU at … of limit | 10m0s / 1m0s |
| [AuditCpuModerate](AuditCpuModerate.json) | sev3 | audit pod … CPU at … of limit | 20m0s / 1m0s |
| [AuditInstanceDown](AuditInstanceDown.json) | sev1 | audit instance … (ns=…) is down | 5m0s / 1m0s |
| [AuditMemoryHigh](AuditMemoryHigh.json) | sev2 | audit pod … memory at … of limit | 10m0s / 1m0s |
| [AuditOOMKilled](AuditOOMKilled.json) | sev1 | audit pod … OOMKilled | 5m0s / 1m0s |
| [AuditReadyRatioLow](AuditReadyRatioLow.json) | sev1 | audit ready replicas < 50% | 5m0s / 1m0s |
| [AuditRestartStorm](AuditRestartStorm.json) | sev1 | audit pod … (ns=…) restarted >3 times in 10m | 5m0s / 1m0s |
| [AuthCpuHigh](AuthCpuHigh.json) | sev2 | auth pod … CPU at … of limit | 10m0s / 1m0s |
| [AuthCpuModerate](AuthCpuModerate.json) | sev3 | auth pod … CPU at … of limit | 20m0s / 1m0s |
| [AuthEventLoopLagHigh](AuthEventLoopLagHigh.json) | sev2 | auth event-loop p99 lag … (pod …) | 5m0s / 1m0s |
| [AuthInstanceDown](AuthInstanceDown.json) | sev1 | auth instance … (ns=…) is down | 5m0s / 1m0s |
| [AuthMemoryHigh](AuthMemoryHigh.json) | sev2 | auth pod … memory at … of limit | 10m0s / 1m0s |
| [AuthOOMKilled](AuthOOMKilled.json) | sev1 | auth pod … OOMKilled | 5m0s / 1m0s |
| [AuthReadyRatioLow](AuthReadyRatioLow.json) | sev1 | auth ready replicas < 50% | 5m0s / 1m0s |
| [AuthRestartStorm](AuthRestartStorm.json) | sev1 | auth pod … (ns=…) restarted >3 times in 10m | 5m0s / 1m0s |
| [IntegrationCpuHigh](IntegrationCpuHigh.json) | sev2 | integration pod … CPU at … of limit | 10m0s / 1m0s |
| [IntegrationCpuModerate](IntegrationCpuModerate.json) | sev3 | integration pod … CPU at … of limit | 20m0s / 1m0s |
| [IntegrationEventLoopLagHigh](IntegrationEventLoopLagHigh.json) | sev2 | integration event-loop p99 lag … (pod …) | 5m0s / 1m0s |
| [IntegrationInstanceDown](IntegrationInstanceDown.json) | sev1 | integration instance … (ns=…) is down | 5m0s / 1m0s |
| [IntegrationMemoryHigh](IntegrationMemoryHigh.json) | sev2 | integration pod … memory at … of limit | 10m0s / 1m0s |
| [IntegrationOOMKilled](IntegrationOOMKilled.json) | sev1 | integration pod … OOMKilled | 5m0s / 1m0s |
| [IntegrationReadyRatioLow](IntegrationReadyRatioLow.json) | sev1 | integration ready replicas < 50% | 5m0s / 1m0s |
| [IntegrationRestartStorm](IntegrationRestartStorm.json) | sev1 | integration pod … (ns=…) restarted >3 times in 10m | 5m0s / 1m0s |
| [JarvisCpuHigh](JarvisCpuHigh.json) | sev2 | jarvis pod … CPU at … of limit | 10m0s / 1m0s |
| [JarvisCpuModerate](JarvisCpuModerate.json) | sev3 | jarvis pod … CPU at … of limit | 20m0s / 1m0s |
| [JarvisMemoryHigh](JarvisMemoryHigh.json) | sev2 | jarvis pod … memory at … of limit | 10m0s / 1m0s |
| [JarvisOOMKilled](JarvisOOMKilled.json) | sev1 | jarvis pod … OOMKilled | 5m0s / 1m0s |
| [JarvisPodNotReady](JarvisPodNotReady.json) | sev1 | jarvis pod … (ns=…) NotReady | 5m0s / 1m0s |
| [JarvisReadyRatioLow](JarvisReadyRatioLow.json) | sev1 | jarvis … ready ratio <50% in ns=… | 5m0s / 1m0s |
| [JarvisRestartStorm](JarvisRestartStorm.json) | sev1 | jarvis pod … (ns=…) restarted >3 times in 10m | 5m0s / 1m0s |
| [PlatformCpuHigh](PlatformCpuHigh.json) | sev2 | … pod … CPU at … of limit | 10m0s / 1m0s |
| [PlatformCpuModerate](PlatformCpuModerate.json) | sev3 | … pod … CPU at … of limit | 20m0s / 1m0s |
| [PlatformMemoryHigh](PlatformMemoryHigh.json) | sev2 | … pod … memory at … of limit | 10m0s / 1m0s |
| [PlatformOOMKilled](PlatformOOMKilled.json) | sev1 | … pod … OOMKilled | 5m0s / 1m0s |
| [PlatformPodNotReady](PlatformPodNotReady.json) | sev1 | … pod … (ns=…) NotReady for 5m | 5m0s / 1m0s |
| [PlatformReadyRatioLow](PlatformReadyRatioLow.json) | sev1 | … ready ratio <50% in ns=… | 5m0s / 1m0s |
| [PlatformRestartStorm](PlatformRestartStorm.json) | sev1 | … pod … (ns=…) restarted >3 times in 10m | 5m0s / 1m0s |
| [WorkflowApiCpuHigh](WorkflowApiCpuHigh.json) | sev2 | workflow API pod … CPU at … of limit | 10m0s / 1m0s |
| [WorkflowApiCpuModerate](WorkflowApiCpuModerate.json) | sev3 | workflow API pod … CPU at … of limit | 20m0s / 1m0s |
| [WorkflowApiEventLoopLagHigh](WorkflowApiEventLoopLagHigh.json) | sev2 | workflow API event-loop p99 lag … (pod …) | 5m0s / 1m0s |
| [WorkflowApiInstanceDown](WorkflowApiInstanceDown.json) | sev1 | workflow API instance … (ns=…) is down | 5m0s / 1m0s |
| [WorkflowApiMemoryHigh](WorkflowApiMemoryHigh.json) | sev2 | workflow API pod … memory at … of limit | 10m0s / 1m0s |
| [WorkflowApiOOMKilled](WorkflowApiOOMKilled.json) | sev1 | workflow API pod … OOMKilled | 5m0s / 1m0s |
| [WorkflowApiReadyRatioLow](WorkflowApiReadyRatioLow.json) | sev1 | workflow API ready replicas < 50% | 5m0s / 1m0s |
| [WorkflowApiRestartStorm](WorkflowApiRestartStorm.json) | sev1 | workflow API pod … (ns=…) restarted >3 times in 10m | 5m0s / 1m0s |
| [WorkflowPodChurn](WorkflowPodChurn.json) | sev1 | workflow service … (ns=…) is churning pods | 10m0s / 1m0s |
| [WorkflowQueueWorkerCpuHigh](WorkflowQueueWorkerCpuHigh.json) | sev2 | workflow queue-worker pod … CPU at … of limit | 10m0s / 1m0s |
| [WorkflowQueueWorkerCpuModerate](WorkflowQueueWorkerCpuModerate.json) | sev3 | workflow queue-worker pod … CPU at … of limit | 20m0s / 1m0s |
| [WorkflowQueueWorkerInstanceDown](WorkflowQueueWorkerInstanceDown.json) | sev1 | workflow queue-worker instance … (ns=…) is down | 5m0s / 1m0s |
| [WorkflowQueueWorkerMemoryHigh](WorkflowQueueWorkerMemoryHigh.json) | sev2 | workflow queue-worker pod … memory at … of limit | 10m0s / 1m0s |
| [WorkflowQueueWorkerOOMKilled](WorkflowQueueWorkerOOMKilled.json) | sev1 | workflow queue-worker pod … OOMKilled | 5m0s / 1m0s |
| [WorkflowQueueWorkerReadyRatioLow](WorkflowQueueWorkerReadyRatioLow.json) | sev1 | workflow queue-workers ready replicas < 50% | 5m0s / 1m0s |
| [WorkflowQueueWorkerRestartStorm](WorkflowQueueWorkerRestartStorm.json) | sev1 | workflow queue-worker pod … (ns=…) restarted >3 times in 10m | 5m0s / 1m0s |
| [WorkflowUnderReplicated](WorkflowUnderReplicated.json) | sev1 | workflow deployment … (ns=…) is under-replicated | 5m0s / 1m0s |

### Kubernetes / workload health (8)

| Rule | Severity | Fires when | Eval / freq |
|---|---|---|---|
| [ClickHouseDiskFillingCritical](ClickHouseDiskFillingCritical.json) | sev1 | ClickHouse PVC … (ns=…) at …% - acts NOW or it wedges | 30m0s / 1m0s |
| [ContainerCrashLooping](ContainerCrashLooping.json) | sev2 | …/… container … is crash-looping | 10m0s / 1m0s |
| [ContainerOOMKilled](ContainerOOMKilled.json) | sev2 | …/… container … was OOMKilled | 5m0s / 1m0s |
| [DeploymentUnderReplicated](DeploymentUnderReplicated.json) | sev2 | …/… under-replicated (<desired Ready for 10m) | 10m0s / 1m0s |
| [NodeEventLoopLagHigh](NodeEventLoopLagHigh.json) | sev2 | …/… (…) Node event-loop p99 lag …s for 5m+ | 5m0s / 1m0s |
| [PlatformPodChurn](PlatformPodChurn.json) | sev1 | …/… is churning pods (>8 created in 1h) | 10m0s / 1m0s |
| [PodMemoryNearLimit](PodMemoryNearLimit.json) | sev2 | …/… (…) using …% of its container memory limit for 15m+ | 15m0s / 1m0s |
| [PodsUnschedulable](PodsUnschedulable.json) | sev2 | …/… cannot be scheduled for 5m | 5m0s / 1m0s |

### Data stores (11)

| Rule | Severity | Fires when | Eval / freq |
|---|---|---|---|
| [MongoConnectionsHigh](MongoConnectionsHigh.json) | sev2 | mongo connections at … of available | 10m0s / 1m0s |
| [MongoHostCpuHigh](MongoHostCpuHigh.json) | sev2 | mongo host node … CPU at … | 10m0s / 1m0s |
| [MongoNoPrimary](MongoNoPrimary.json) | sev1 | mongo replset has no PRIMARY | 5m0s / 1m0s |
| [MongoReplicationLagHigh](MongoReplicationLagHigh.json) | sev2 | mongo replication lag …s (member … state=…) | 5m0s / 1m0s |
| [MongoReplsetMemberNotOperational](MongoReplsetMemberNotOperational.json) | sev1 | mongo replset member … state=… (not PRIMARY|SECONDARY) | 5m0s / 1m0s |
| [MongoResidentMemoryHigh](MongoResidentMemoryHigh.json) | sev2 | mongo resident memory > 85% of node memory | 10m0s / 1m0s |
| [RedisConnectionsHigh](RedisConnectionsHigh.json) | sev3 | Redis … connections at … of maxclients | 10m0s / 1m0s |
| [RedisDown](RedisDown.json) | sev3 | Redis … is DOWN (redis_up=0) | 5m0s / 1m0s |
| [RedisEvictedKeys](RedisEvictedKeys.json) | sev3 | Redis … evicted … keys in 5m (memory pressure) | 5m0s / 1m0s |
| [RedisMemoryHigh](RedisMemoryHigh.json) | sev3 | Redis … memory at … of maxmemory | 10m0s / 1m0s |
| [RedisRejectedConnections](RedisRejectedConnections.json) | sev3 | Redis … rejected … connections in 5m | 5m0s / 1m0s |

### Storage (6)

| Rule | Severity | Fires when | Eval / freq |
|---|---|---|---|
| [PvcInodesHigh](PvcInodesHigh.json) | sev2 | PVC … (ns=…) at … inode usage | 10m0s / 1m0s |
| [PvcUsageCritical](PvcUsageCritical.json) | sev1 | PVC … (ns=…) at … used | 5m0s / 1m0s |
| [PvcUsageHigh](PvcUsageHigh.json) | sev2 | PVC … (ns=…) at … used for 30m | 30m0s / 1m0s |
| [PvcUsageModerate](PvcUsageModerate.json) | sev3 | PVC … (ns=…) at … used | 5m0s / 1m0s |
| [PvcWillFillIn24h](PvcWillFillIn24h.json) | sev1 | PVC … (ns=…) projected to fill in <24h | 10m0s / 1m0s |
| [PvcWillFillIn2d](PvcWillFillIn2d.json) | sev2 | PVC … (ns=…) projected to fill in <2d | 30m0s / 1m0s |

## Provenance

Exported 2026-09-22 from the SigNoz instance Refold runs against its own deployments, then
genericized. Rules that only apply to Refold's hosted topology (MongoDB Atlas, multi-tenant
namespace scoping) were dropped; where a rule existed per environment, the single-namespace form
was kept. Server-owned fields (`id`, timestamps, author, live state) are not included, so each file
POSTs cleanly as a new rule.
