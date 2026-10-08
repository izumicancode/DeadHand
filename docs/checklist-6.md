# Launch readiness

Use a test recipient and non-sensitive file for the final end-to-end check.
Do not create a switch for live data until every applicable item passes.

## Production configuration

- [ ] Confirm the production database has the expected schema and migrations.
- [ ] Configure required environment variables; keep their values private.
- [ ] Confirm cron authorization and hosting-plan support for its schedule.
- [ ] Confirm the sending domain is verified and a test email is delivered.

## End-to-end verification

- [ ] Check in to a test switch and confirm its deadline moves forward.
- [ ] Miss a test check-in; confirm the warning arrives before grace ends.
- [ ] Verify a wrong cancel password is rejected.
- [ ] Cancel with the correct password; confirm no release occurs.
- [ ] Let a separate test switch expire; confirm the encrypted email arrives.
- [ ] Decrypt the received file and confirm its contents match the test file.
- [ ] Review cron logs and delivery events; resolve outstanding failures.

## Go / no-go

- [ ] Record the deployed revision and date of the successful verification.
- [ ] Obtain launch sign-off before creating switches for live data.
