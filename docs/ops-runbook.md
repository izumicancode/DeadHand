# Operations runbook

## Routine operations

- Keep the deployment URL and secrets in a restricted password manager.
- Never put secrets in the repository, tickets, or logs.
- Verify database schema and migrations before production releases.
- After deploy or cron changes, check cron logs and email delivery.

## Rotate access secrets

- Generate distinct `SETUP_SECRET` and `CRON_SECRET` replacements.
- Store both securely; match `CRON_SECRET` in app config and cron protection.
- Redeploy if needed; verify setup access and both cron authorization cases.
- Record the rotation date and operator, never the secret values.
- Remove old values from active configuration after verification.
