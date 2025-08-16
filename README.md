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

### Notes and usage

- The workflow uses short-lived credentials via `Azure/login`. Ensure the service principal assigned to `AZURE_CLIENT_ID` has permissions to update App Service Plans and SQL databases in the target subscription and resource group.
- The `DB_CAPACITY` values in the workflow are integers passed to `az sql db update --capacity`. Adjust these values to match supported SKUs/tiers for your SQL configuration.
- The App Service Plan SKU values used in the workflow are examples (`B1` for idle, `P1v3` for event). Change them to match the SKUs available in your subscription and desired pricing tier.
- The workflow contains simple conditional steps to set environment variables; you can extend it to perform additional checks or notify teams when scaling occurs.

### Example: trigger manually

1. In GitHub, go to the Actions tab and select the `scale production` workflow.
2. Click "Run workflow" and choose `idle` or `event` for the `scale` input.

Security

- Keep secrets scoped to the repository or organization and rotate service principal credentials regularly.
