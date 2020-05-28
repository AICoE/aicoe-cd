# Secret Management

Secret management is handled using the KSOPs plugin.

Use [sops](https://github.com/mozilla/sops) to encrypt your secrets in vcs. Use the AICOE-SRE [public sops key](https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xefdb9afbd18936d9ab6b2eecbd2c73ff891fbc7e) to encrypt
your secrets so that ArgoCD may use KSOPs to decrypt them.

## Updating Private key

Export the key

```bash
gpg --export-secret-keys "${KEY_ID}" | base64 > private.asc
```

Create a `${target-env}-key.yaml` and copy the contents of `private.asc` onto the field.

`${target-env}-key.yaml`:

```yaml
gpg_key: |
    # Contents of private.asc
```

Encrypt `${target-env}-key.yaml` using ansible-vault:

```yaml
ansible-vault encrypt ${target-env}-key.yaml
```

Use the appropriate vault pass when prompted to encrypt the file and store
the file in `vars/gpg-keys`.
