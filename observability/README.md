# Observability

Reference monitoring configuration for a Refold on-prem deployment — the dashboards and alert rules
Refold uses to operate the platform, published so you can import them into your own
[SigNoz](https://signoz.io) instance instead of building equivalents from scratch.

| Folder | Contents | Editing needed before import |
|---|---|---|
| [dashboards/](dashboards/) | 7 dashboards (schema `v6`) | None — filters are dashboard variables |
| [alerts/](alerts/) | 79 alert rules | Substitute 3 placeholders; set notification channels |

Start with **dashboards** — they work as soon as your metrics are flowing and will show you which
exporters you are missing. Then import the **alerts** once you know which signals you actually
collect and what your normal load looks like.

Each folder's README has the import commands and what the content assumes about your setup.

## Scope

These cover the Refold platform running in Kubernetes: the Refold services themselves, workload and
pod health, MongoDB, Redis, and persistent volumes. They assume SigNoz with the OpenTelemetry
Kubernetes receivers plus the standard MongoDB and Redis Prometheus exporters.

Treat them as a starting point, not a finished monitoring setup. Thresholds reflect Refold's own
capacity and traffic patterns; review them against yours before relying on a page at 3am. Nothing
here is generated or refreshed automatically — it is a curated snapshot (2026-09-22).

## What is not here

Notification channel configuration. Channels hold webhook URLs and integration keys, so the rules
reference them by name only and ship with routing unset — create your own channels in SigNoz and
attach them after import.
