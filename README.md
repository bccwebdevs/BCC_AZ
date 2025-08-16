# Automated workflows for BCC App

## Scale production workflow

This repository includes a GitHub Actions workflow `.github/workflows/scale.yml` that is used to scale the production environment between an "idle" and an "event" configuration.

### Key points

- Workflow name: `scale production`
- File: `.github/workflows/scale.yml`

### Triggers

- `workflow_dispatch` — manual trigger with an optional `scale` input (`idle` or `event`).
- `schedule` — cron schedule configured to run weekly (the example triggers Monday at midnight) to scale back to `idle`.

### Inputs

- `scale` (choice): `idle` or `event`. Default: `idle`.

### Behavior

- When run with `scale: event`, the workflow will prepare values for scaling up resources (e.g. a higher App Service Plan SKU and larger database capacity).
- When run with `scale: idle` or when triggered by the scheduled job, the workflow will prepare values to scale down resources to save cost.

### Steps summary

- Checkout repository.
- Authenticate to Azure using the `Azure/login` action and the repo secrets.
- Set environment values for `idle` and `event` modes and write them to the GitHub Actions environment.
- Use the Azure CLI action to update the App Service Plan SKU and SQL Database capacity using the prepared values.

### Required secrets and environment variables

The workflow expects the following repository environment secrets to be set (names used in the workflow):

Environment name: `prod`

- `AZURE_TENANT_ID` — Azure tenant id used by `Azure/login`.
- `AZURE_CLIENT_ID` — Azure service principal client id used by `Azure/login`.
- `AZURE_SUBSCRIPTION_ID` — Azure subscription id used by `Azure/login`.
- `APP_SERVICE_PLAN_NAME` — Name of the App Service Plan to update.
- `RESOURCE_GROUP_NAME` — Resource group that contains the resources to update.
- `SQL_DATABASE_NAME` — Name of the SQL database to update.
- `SQL_SERVER_NAME` — Name of the SQL server that hosts the database.

### Example: trigger manually

1. In GitHub, go to the Actions tab and select the `scale production` workflow.
2. Click "Run workflow" and choose `idle` or `event` for the `scale` input.

### Security

- Uses Workload Identity Federation (OIDC) to eliminate the need for long-lived credentials.
