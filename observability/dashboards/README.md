# Dashboards

Reference SigNoz dashboards for a Refold on-prem deployment — import these into your own SigNoz to
get the views Refold uses to operate the platform. **7 dashboards**, one JSON file each, SigNoz
dashboard schema `v6`.

| Dashboard | Panels | What it shows |
|---|---:|---|
| [HTTP + gRPC — Refold](http-grpc-refold.json) | 22 | Request rate, error rate and latency percentiles per service, for HTTP and gRPC traffic |
| [LLM Usage — By Agent](llm-usage-by-agent.json) | 11 | Per-agent LLM usage: volume, token mix, cost, error rate, latency and prompt-cache efficiency |
| [Node.js Runtime — Refold](node-js-runtime-refold.json) | 20 | Event-loop lag, heap, GC and handle counts for the Node.js services, averaged per service |
| [Redis metrics](redis-metrics.json) | 6 | Redis connections, memory, evictions, hit ratio and TTL |
| [Service Health — Single Pane — Refold](service-health-single-pane-refold.json) | 15 | One screen for whether the platform is healthy: latency, errors, CPU/memory against limits, restarts and HPA headroom |
| [Thanos Workflow Health — Refold](thanos-workflow-health-refold.json) | 18 | Temporal workflow and activity health from the SDK metrics — completions, failures, task latency, worker capacity |
| [Workflow On-call — Refold](workflow-on-call-refold.json) | 12 | Triage view for a workflow that is stuck or failing: failure rate, in-flight load, queue back-pressure, worker capacity |

## Importing

```bash
BASE="$SIGNOZ_URL"       # your SigNoz base URL
KEY="$SIGNOZ_API_KEY"    # a SigNoz API key

for f in dashboards/*.json; do
  jq '{name, tags, spec, schemaVersion}' "$f" \
    | curl -s -X POST "$BASE/api/v2/dashboards" \
        -H "SIGNOZ-API-KEY: $KEY" -H "Content-Type: application/json" --data @-
done
```

The `jq` filter drops the server-owned fields (`id`, timestamps, `locked`, `legacy`, `source`) so
each file creates a fresh dashboard. To update one in place instead, `PUT` to
`$BASE/api/v2/dashboards/<id>` using the id of the dashboard on *your* instance.

Requires SigNoz with the v2 dashboard API; the v1 API is deprecated and rejects these calls.

## No substitution needed

Unlike the [alert rules](../alerts/), these dashboards need no editing before import. Every
environment-specific filter is a **dashboard variable** populated from your own data at view time —
pick your namespace, service and pod from the dropdowns:

| Dashboard | Variables |
|---|---|
| HTTP + gRPC — Refold | `Environment (k8s namespace)`, `Service`, `Pod` |
| LLM Usage — By Agent | none — shows all data |
| Node.js Runtime — Refold | `Environment (k8s namespace)`, `Service`, `Pod` |
| Redis metrics | none — shows all data |
| Service Health — Single Pane — Refold | `Environment (k8s namespace)`, `Service`, `Pod` |
| Thanos Workflow Health — Refold | `Environment (k8s namespace)`, `Org ID`, `Workflow type`, `Task queue`, `Worker type` |
| Workflow On-call — Refold | `Environment (k8s namespace)`, `Workflow type`, `Task queue`, `Org ID` |

## What they assume

- Refold service metrics scraped into SigNoz from each service's `/metrics` endpoint
  (`request_response_latency_seconds`, `nodejs_*`, Temporal SDK `temporal_*`).
- Kubernetes metrics from the OpenTelemetry Kubernetes receivers (`k8s_container_*`, `k8s_hpa_*`,
  `k8s_namespace_name` / `k8s_pod_name` attributes).
- The Redis dashboard needs a Redis Prometheus exporter; the LLM usage dashboard needs the Refold
  AI service's usage metrics.

Panels whose metrics you do not collect render empty rather than failing — start with Service
Health, and add exporters as gaps show up.

## Provenance

Exported 2026-09-22 from the SigNoz instance Refold runs against its own deployments. Dashboard
titles keep the `— Refold` suffix they carry in SigNoz; rename them freely after import.
