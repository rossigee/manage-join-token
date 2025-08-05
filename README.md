# Kubernetes Join Token Manager

A lightweight infrastructure tool that automatically maintains fresh join tokens for Kubernetes cluster nodes. This tool runs as a scheduled systemd service on control plane nodes, generating new kubeadm join tokens every 8 hours and storing them securely in HashiCorp Vault.

## Overview

The primary use case is to automate the bootstrapping of newly launched cluster nodes in GitOps environments. New nodes can autonomously obtain fresh join tokens from Vault using bootstrap credentials, enabling fully automated cluster expansion without manual token management.

### Key Features

- **Automated Token Rotation**: Generates new join tokens every 8 hours for enhanced security
- **Vault Integration**: Securely stores tokens in HashiCorp Vault KV store
- **Systemd Native**: Proper service lifecycle management with timer-based scheduling
- **GitOps Ready**: Designed for infrastructure-as-code and configuration management
- **Single Purpose**: Focused tool that does one thing well

## Installation

### Option 1: Debian Package (Recommended)

Download the latest `.deb` package from [GitHub Releases](https://github.com/your-org/manage-join-token/releases):

```bash
# Download and install
wget https://github.com/your-org/manage-join-token/releases/download/v1.0.0/manage-join-token_1.0.0_all.deb
sudo dpkg -i manage-join-token_1.0.0_all.deb
```

### Option 2: Manual Installation

```bash
# Copy the script
sudo cp manage-join-token /usr/bin/manage-join-token
sudo chmod +x /usr/bin/manage-join-token

# Install systemd units
sudo cp systemd/* /etc/systemd/system/
sudo systemctl daemon-reload
```

## Configuration

### 1. Environment Variables

Configure the service with your Vault settings:

```bash
sudo systemctl edit manage-join-token.service
```

Add the following configuration:

```ini
[Service]
Environment=VAULT_SECRET_PATH=k8s-clusters/your-cluster-name
Environment=VAULT_ADDR=https://vault.yourdomain.com
EnvironmentFile=/etc/kubernetes/vault-token
```

### 2. Vault Token Setup

Create the Vault token file:

```bash
# Option 1: Static token file
echo "your-vault-token" | sudo tee /etc/kubernetes/vault-token

# Option 2: Use Vault agent for token renewal (recommended)
# Configure vault-agent to maintain the token file
```

### 3. Enable and Start Service

```bash
# Enable the timer for automatic execution
sudo systemctl enable manage-join-token.timer
sudo systemctl start manage-join-token.timer

# Check service status
sudo systemctl status manage-join-token.timer
```

## Usage

### Automatic Operation

Once configured, the service runs automatically every 8 hours via systemd timer. No manual intervention required.

### Manual Execution

For testing or immediate token generation:

```bash
# Run once manually
sudo systemctl start manage-join-token.service

# Check logs
sudo journalctl -u manage-join-token.service -f
```

### Retrieving Tokens

New nodes can retrieve the join token from Vault:

```bash
# Example: Retrieve join token for cluster bootstrapping
vault kv get -field=join_command k8s-clusters/your-cluster-name
```

## Requirements

### Control Plane Node

- `kubeadm` binary in PATH
- Cluster admin permissions for token generation
- Root access for systemd service execution

### Vault Environment

- HashiCorp Vault server accessible from control plane
- Vault CLI installed and configured
- Valid authentication token with write permissions to specified KV path
- KV secrets engine enabled

## Monitoring

### Service Health

```bash
# Check timer status
sudo systemctl status manage-join-token.timer

# View recent executions
sudo journalctl -u manage-join-token.service --since "24 hours ago"

# Check next scheduled run
sudo systemctl list-timers manage-join-token.timer
```

### Token Validation

```bash
# Verify token exists in Vault
vault kv get k8s-clusters/your-cluster-name

# Test token validity (kubeadm will validate format)
kubeadm token list
```

## Security Considerations

- Service runs with root privileges (required for kubeadm operations)
- Vault tokens should use least-privilege access (write to specific KV path only)
- Join tokens have 24-hour default expiration (kubeadm default)
- Consider using Vault agent for token renewal instead of static tokens
- Deploy only on trusted control plane nodes

## Troubleshooting

### Common Issues

**Service fails to start:**
```bash
# Check configuration
sudo systemctl status manage-join-token.service
sudo journalctl -u manage-join-token.service -n 20
```

**Vault authentication errors:**
```bash
# Test Vault connectivity
vault auth -method=token
vault kv put test/path key=value
```

**Missing kubeadm:**
```bash
# Ensure kubeadm is installed and in PATH
which kubeadm
kubeadm version
```

### Log Analysis

```bash
# View detailed logs
sudo journalctl -u manage-join-token.service -f

# Check timer execution history
sudo journalctl -u manage-join-token.timer
```

## Development

### Building from Source

```bash
# Build Debian package
dpkg-buildpackage

# Run tests
./test-script.sh  # If test suite exists
```

### Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes with appropriate tests
4. Submit a pull request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## License

[Add appropriate license information]

## Related Projects

- [flux-golder](../flux-golder/) - Main GitOps infrastructure management
- [ansible-playbooks](../ansible-playbooks/) - Infrastructure automation
- [k8s-node-packer](../k8s-node-packer/) - Automated node image builds
