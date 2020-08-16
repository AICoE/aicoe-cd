# Update gpg key

## Prerequisites

* sops 3.6+
* imported aicoe-sre gpg key

## Instructions

Export the key

```bash
$ gpg --export-secret-keys "${KEY_ID}" | base64 > private.asc
```

```bash
# From the repo root
$ cd manifests/overlays/prod/resources/secrets/gpg
$ sops secret.enc.yaml
```

Copy the contents of `private.asc` into the `private.key` field.

Save the file, exit. Commit and make a PR.
