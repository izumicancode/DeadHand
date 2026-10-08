# Deployment checklist

## Before deployment

- [ ] Create the Vercel project and attach the Postgres database.
- [ ] Apply `schema.sql` for new installs or the required database migrations.
- [ ] Verify the Resend sending domain and choose the matching `FROM_EMAIL`.
- [ ] Set `POSTGRES_URL`, `RESEND_API_KEY`, `FROM_EMAIL`, and `BASE_URL`.
- [ ] Set `SETUP_SECRET` and `CRON_SECRET` in the Vercel environment.
- [ ] Enable cron-job protection and set it to the same value as `CRON_SECRET`.
- [ ] Confirm the plan supports the cron cadence.
- [ ] Use the README's pinger guidance if the plan limits cron frequency.

## Deploy and verify

- [ ] Deploy the intended revision to production.
- [ ] Confirm the deployed URL loads and cron rejects unauthorized requests.
- [ ] Run the [testing checklist](testing-checklist.md) with test data.
- [ ] Record the deployed revision and result of the checks.
