# SentinelOne — Bulk Resolve Identity Alerts (Postman)

A Postman collection for bulk-resolving SentinelOne Identity alerts via the Unified Alerts GraphQL API. It fetches alert IDs in pages of x (defaults to 250), then fires a single mutation per page that resolves each alert, sets the analyst verdict to **False Positive – User Error**, and attaches a closing note.

## How it works

The collection uses Postman's `pm.execution.setNextRequest` to create a two-step loop:

1. **`getAlertIds`** — Queries for up to 250 `NEW` Identity alerts in the target scope and stores their IDs in a collection variable.
2. **`resolveAlertBatch`** — Dynamically builds an `alertTriggerActions` mutation from those IDs (using `or` filter clauses), fires it, and logs success/failure/skip counts to the Postman console.

## Importing into Postman

You need to import two files: the **collection** (the requests and scripts) and the **environment** (your configuration variables).

### 1. Import the collection

1. Open Postman.
2. Click **Import** in the top-left corner.
3. Drag `SentinelOne GraphQL Resolve Alerts.postman_collection.json` onto the import dialog, or click **files** and select it.
4. Click **Import** to confirm. The collection will appear in the **Collections** sidebar on the left.

### 2. Import the environment

1. Click **Import** again.
2. Drag `Console-Name-Global-Account-service_user.postman_environment.json` onto the import dialog, or click **files** and select it.
3. Click **Import** to confirm.
4. In the top-right corner of Postman, open the environment dropdown (it will say **No environment**) and select the newly imported environment.

### 3. Fill in your environment variables

With the environment selected, click the **eye icon** next to the environment dropdown (or go to **Environments** in the left sidebar and click on the environment name) to open the variable editor. Fill in the **Current Value** column for each variable:

| Variable | Description |
|---|---|
| `SERVICE_USER_TOKEN` | API bearer token for your SentinelOne service user |
| `URL` | Your SentinelOne console base URL (e.g. `https://usea1-abc.sentinelone.net`) |
| `SCOPE_ID` | The account or site ID to target |
| `SCOPE_TYPE` | Scope type — `ACCOUNT` or `SITE` |

## Running the collection

The collection must be run with the **Collection Runner** — sending requests one at a time from the sidebar will not work because the loop logic relies on the runner's request sequencing.

1. In the **Collections** sidebar, hover over **SentinelOne GraphQL Resolve Alerts** and click the **▶ Run** button (or right-click and choose **Run collection**).
2. In the Collection Runner panel, make sure both **getAlertIds** and **resolveAlertBatch** are checked.
3. Set **Iterations** to `1` (or more as desired) (the collection loops itself internally via `setNextRequest`).
4. Click **Run SentinelOne GraphQL Resolve Alerts**.
5. Watch the **Postman Console** (View → Postman Console, or `Ctrl/Cmd + Alt + C`) for per-batch output showing how many alerts were fetched and resolved.

The runner will keep cycling through `getAlertIds → resolveAlertBatch` until there are no more `NEW` Identity alerts in the target scope, then stop automatically.

## What the mutation does to each alert

- Status → **Resolved**
- Analyst verdict → **False Positive – User Error**
- Adds a note: *"Alert bulk closed while addressing False Positives related to Over Pass-The-Hash attacks. Exclusions have been added for the False Positives and this alert will regenerate on next attempt."*
