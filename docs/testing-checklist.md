# Testing checklist

Use a non-sensitive file and a test recipient. Do not use a real switch until
every applicable step passes.

## Check-in and warning

- [ ] Create a short-duration test switch and save both links securely.
- [ ] Check in, then confirm the next deadline moves forward.
- [ ] Miss a check-in; confirm the warning arrives before grace ends.

## Cancellation and release

- [ ] Use the wrong cancel password; confirm the switch stays armed.
- [ ] Use the correct password; confirm release is prevented.
- [ ] Let a separate test switch expire; confirm the encrypted email arrives.

## Decryption

- [ ] Decrypt in `/decrypt` (passphrase) or with the recipient's PGP key.
- [ ] Confirm the decrypted content matches the original test file.
- [ ] Remove test switches and test files when testing is complete.
