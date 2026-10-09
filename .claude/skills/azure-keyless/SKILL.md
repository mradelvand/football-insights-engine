---
name: azure-keyless
description: How this project calls Microsoft Foundry models and agents without keys, and deploys to Azure Container Apps with a managed identity. Use when writing code in src/explain, running az commands, deploying, or debugging 401/403 errors from Azure.
user-invocable: false
---

# Keyless Azure for this project

## Rules

- Never put API keys, connection strings or tokens in code, commits, logs or chat. `.env` files are blocked from reading in `.claude/settings.json`.
- Always authenticate with `DefaultAzureCredential` from `azure-identity`. If a Microsoft sample uses `AzureCliCredential`, switch it. `DefaultAzureCredential` uses `az login` in the codespace and the managed identity on Azure, with no code change.
- Always ask CESC before running any `az` command.

## Setup facts (checked Oct 8, 2026)

- Packages: `azure-ai-projects>=2.3.0` (2.8.0 was current), `azure-identity`, and `agent-framework-foundry` for agents. Add them to `requirements.txt` when week 4 starts.
- Project endpoint format: `https://<resource>.services.ai.azure.com/api/projects/<project>`. Read it from an environment variable such as `FOUNDRY_PROJECT_ENDPOINT`; it isn't a secret, but it keeps code portable.
- Use CESC's own model deployment name, not the one in Microsoft's samples.
- Role for calling models and agents: **Foundry User** on the Foundry resource. Give it to CESC's own user (week 4) and to the container app's system-assigned managed identity (week 6).
- A 403 right after a role assignment usually means waiting 5–10 minutes for it to apply.
- Sign in from the codespace with `az login --use-device-code`.

## Deploy (week 6)

```bash
az extension add --name containerapp --upgrade
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights
az containerapp up --name football-insights --resource-group rg-football-insights --source .
```

Then turn on the system-assigned identity (container app → Identity), give it Foundry User, and set `FOUNDRY_PROJECT_ENDPOINT` as an environment variable on the app.

IT comparison: the managed identity is the app's own account, like a computer account in Active Directory. RBAC on the Foundry resource is the ACL.
