# plasmactl-processors

A [Launchr](https://github.com/launchrctl/launchr) plugin for [Plasmactl](https://github.com/plasmash/plasmactl) that provides template processors for enhanced action functionality.

## Overview

`plasmactl-processors` extends Plasmactl with specialized template functions for processing encrypted data and other advanced use cases in action definitions.

## Features

### Ansible Vault Template Function

Decrypt and extract values from Ansible Vault files using dot-notation key paths directly in action templates.

## Usage

### In Action Definitions

Use the `AnsibleVault` template function to access encrypted vault values:

```yaml
action:
  title: Example with Ansible Vault
  description: Deploy with secrets from vault
runtime:
  type: container
  image: alpine
  command:
    - echo
    - '{{ AnsibleVault "path/to/vault.yaml" "database.password" }}'
```

### Nested Key Access

Access nested values using dot notation:

```yaml
command:
  - '{{ AnsibleVault "secrets/vault.yaml" "api.credentials.token" }}'
  - '{{ AnsibleVault "config/vault.yaml" "services.postgres.password" }}'
```

## Setup

### Store Vault Passphrase

Store your Ansible Vault passphrase securely using [Keyring](https://github.com/launchrctl/keyring):

```bash
plasmactl keyring:set "ansible-vault:path/to/vault.yaml"
```

When prompted, enter your vault passphrase. This is stored securely in the keyring.

### Multiple Vaults

For different vault files, store each passphrase separately:

```bash
plasmactl keyring:set "ansible-vault:secrets/production.vault.yaml"
plasmactl keyring:set "ansible-vault:secrets/staging.vault.yaml"
plasmactl keyring:set "ansible-vault:config/database.vault.yaml"
```

## Vault File Format

Your Ansible Vault files should be encrypted YAML:

```yaml
# vault.yaml (encrypted)
database:
  password: "secret123"
  host: "db.example.com"

api:
  credentials:
    token: "abc123xyz"
    endpoint: "https://api.example.com"
```

Encrypt with:
```bash
ansible-vault encrypt vault.yaml
```

## Workflow Example

```bash
# 1. Create and encrypt vault file
cat > secrets/vault.yaml <<EOF
database:
  password: "MySecretPassword"
EOF

ansible-vault encrypt secrets/vault.yaml

# 2. Store passphrase in keyring
plasmactl keyring:set "ansible-vault:secrets/vault.yaml"

# 3. Use in action
cat > .launchr/deploy.yaml <<EOF
action:
  title: Deploy Application
runtime:
  type: container
  image: deployer:latest
  environment:
    DB_PASSWORD: '{{ AnsibleVault "secrets/vault.yaml" "database.password" }}'
  command:
    - ./deploy.sh
EOF

# 4. Run action
plasmactl deploy
```

## Security

- **Passphrases** are stored in the system keyring (encrypted at rest)
- **Vault files** remain encrypted in the repository
- **Decryption** happens only at runtime, in memory
- **No plaintext** secrets are written to disk

## Best Practices

1. **Never commit unencrypted vaults** to version control
2. **Use separate vaults** for different environments (dev/staging/prod)
3. **Rotate passwords** regularly and re-encrypt vaults
4. **Limit vault access** to only the keys needed for each action
5. **Document key paths** in your action YAML comments

## Error Handling

Common errors and solutions:

### "Vault passphrase not found"
```bash
plasmactl keyring:set "ansible-vault:path/to/vault.yaml"
```

### "Invalid key path"
Check that the dot-notation path matches your vault structure:
```yaml
# For vault.yaml containing:
app:
  config:
    secret: "value"

# Use: AnsibleVault "vault.yaml" "app.config.secret"
```

### "Decryption failed"
Verify your passphrase is correct and the vault file is properly encrypted.

## Documentation

- [Plasmactl](https://github.com/plasmash/plasmactl) - Main CLI tool
- [Plasma Platform](https://plasma.sh) - Platform documentation
- [Launchr Keyring](https://github.com/launchrctl/keyring) - Secure credential storage

## License

Apache License 2.0
