# SentinelOne — Bulk Resolve Identity Alerts (Postman)

A Postman collection for bulk-resolving SentinelOne Identity alerts via the Unified Alerts GraphQL API. It fetches alert IDs in pages of x (defaults to 250), then fires a single mutation per page that resolves each alert, sets the analyst verdict to **False Positive – User Error**, and attaches a closing note.

## How it works

The collection uses Postman's `pm.execution.setNextRequest` to create a two-step loop:

1. **`getAlertIds`** — Queries for up to 250 `NEW` Identity alerts in the target scope and stores their IDs in a collection variable.
2. **`resolveAlertBatch`** — Dynamically builds an `alertTriggerActions` mutation from those IDs (using `or` filter clauses), fires it, and logs success/failure/skip counts to the Postman console.

Run the collection with the Postman Collection Runner to process alerts in batches. Ensure that both request are checked. getAlertIds will call resolveAlertBatch.

## Setup

Fill in the environment file (`Console-Name-Global-Account-service_user.postman_environment.json`) with your values before running:

| Variable | Description |
|---|---|
| `SERVICE_USER_TOKEN` | API bearer token for your SentinelOne service user |
| `URL` | Your SentinelOne console base URL (e.g. `https://usea1-abc.sentinelone.net`) |
| `SCOPE_ID` | The account or site ID to target |
| `SCOPE_TYPE` | Scope type — `ACCOUNT` or `SITE` |

## What the mutation does to each alert

- Status → **Resolved**
- Analyst verdict → **False Positive – User Error**
- Adds a note: *"Alert bulk closed while addressing False Positives related to Over Pass-The-Hash attacks. Exclusions have been added for the False Positives and this alert will regenerate on next attempt."*
