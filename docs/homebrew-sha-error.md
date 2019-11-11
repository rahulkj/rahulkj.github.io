SHA1 error when using brew upgrade
---

Recently I was trying to upgrade my `cloudfoundry-cli` via brew, and I received the SHA mismatch error:

```
Error: SHA1 mismatch
Expected: `f218ed64ce6e7a5d3670acdd6a18e5ed95421d1f`
Got: `3a57f6f44186e0dba34ef8b8fb4a9047e9e5d8a3`
(To retry an incomplete download, remove the file above.)
```

The only hack that worked for me was to run `brew edit cloudfoundry-cli` and copy the `SHA256` and replace the `sha256` in the file, and run `brew upgrade` again.

This would work for any of the package that has SHA mismatch error.
