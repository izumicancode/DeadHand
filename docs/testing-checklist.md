# Testing checklist

Use a non-sensitive file and a test recipient. Do not use a real switch until
every applicable step passes.

## Check-in and warning

- [ ] Create a short-duration test switch and save both links securely.
- [ ] Check in, then confirm the next deadline moves forward.
- [ ] Miss a check-in and confirm the warning arrives before the grace period
	ends.

## Cancellation and release

- [ ] Try the cancel link with the wrong password and confirm the switch stays
	armed.
- [ ] Cancel with the correct password and confirm the switch does not release
	the file.
- [ ] Create a fresh test switch, leave it armed through the grace period, and
	confirm the recipient receives the encrypted file.

## Decryption

- [ ] Open the received file with `/decrypt` in passphrase mode, or with the
	recipient's private key in PGP mode.
- [ ] Confirm the decrypted content matches the original test file.
- [ ] Remove test switches and test files when testing is complete.
