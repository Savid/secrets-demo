# Secrets demo

Example of some in project stored secrets tools.

## Setup

Install;
- [SOPS](https://github.com/mozilla/sops)
- [git-secret](https://git-secret.io/installation)

### Import GPG keys locally

```bash
gpg --import john@smith.com.public.key
gpg --import john@smith.com.private.key
```

### git-secret

Reveal the current secrets in the project;
```bash
# will decode the some-dirty-secrets/git-secret.yaml file
git secret reveal
```

Update secret;
```bash
# update some-dirty-secrets/git-secret.yaml first

# hide the secret
git secret hide
```

Add new secret file;
```bash
git secret add some-dirty-secrets/new-secret.yaml
git secret hide
```

Add new GPG key;
```bash
# public key must be in local key store
git secret tell bill@bob.com
```

Remove GPG key;
```bash
git secret removeperson bill@bob.com
```

Anytime a person is removed technically you should roll all secrets because the user can still decrypt historical commits (if the project is public).

### SOPS

View secret;
```bash
# decrypt to stdout
sops -d some-dirty-secrets/sops.enc.yaml

# decrypt to .gitignored file
sops -d some-dirty-secrets/sops.enc.yaml > some-dirty-secrets/sops.yaml
```

update secret;
```bash
# in place edit in vscode (will decode/encode automatically)
EDITOR="code --wait" sops -i some-dirty-secrets/sops.enc.yaml
```

manually encrypt secret;
```bash
# this can setup a bit nicer in the .sops.yaml iirc
# will only excrypt "data" fields of the yaml in this example
sops --encrypt --encrypted-regex '^(data|stringData)$' some-dirty-secrets/sops.yaml > some-dirty-secrets/sops.enc.yaml
```

#### `.sops.yaml`

SOPS will read the config file `.sops.yaml`;
```yaml
creation_rules:
  # john@smith.com - 27A10322070556D52F696C198A21D32E8BB9EC31
  - pgp: >-
      27A10322070556D52F696C198A21D32E8BB9EC31
  # age
  # - age: "age1yt3tfqlfrwdwx0z0ynwplcr6qxcxfaqycuprpmy89nr83ltx74tqdpszlw"
  # AWS KMS
  # - kms: "arn:aws:kms:AWS_REGION:ACCOUNT_ID:key/KMS_KEY"
  # Hashicorp Vault
  # - hc_vault_uris: http://localhost:8200/v1/sops/keys/thirdkey
  # GCP KMS
  # - gcp_kms: projects/mygcproject/locations/global/keyRings/mykeyring/cryptoKeys/thekey
  # Azure Key Vault
  # - azure-kv: https://sops.vault.azure.net/keys/sops-key/some-string
```

SOPS [recommends](https://github.com/mozilla/sops#id8) using [age](https://github.com/FiloSottile/age) over PGP (GPG).

SOPS also supports thirdparty key management solutions such as AWS KMS, Hashicorp Vault etc. This avoids the problem of user access being removed but still being able to decrypt historical commits.

### Cleanup

```bash
gpg --delete-secret-key john@smith.com
gpg --delete-key john@smith.com
```
