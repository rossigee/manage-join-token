# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a lightweight Kubernetes infrastructure tool that automatically maintains fresh join tokens for cluster nodes. The tool runs as a scheduled systemd service on control plane nodes, generating new kubeadm join tokens every 8 hours and storing them securely in HashiCorp Vault.

### Core Purpose
- Automates Kubernetes join token rotation for security and node bootstrapping
- Integrates with HashiCorp Vault for secure token storage
- Enables automated cluster node provisioning without manual token management
- Designed for GitOps environments where new nodes need autonomous cluster joining

## Architecture

### Core Components
1. **manage-join-token** - Bash script that generates join tokens via kubeadm and stores them in Vault
2. **systemd service/timer** - Schedules token rotation every 8 hours
3. **Debian packaging** - Standard dpkg build system for distribution
4. **GitHub Actions** - Automated release pipeline with tarball and .deb artifacts

### Key Design Patterns
- **Single-purpose tool**: Does one thing well - token management
- **Environment-driven configuration**: Uses environment variables for Vault integration
- **Systemd integration**: Proper service lifecycle management
- **Security-first**: Tokens stored in external secret management system
- **Infrastructure as Code**: Designed for automated deployment and configuration

## Development Workflow

### Building and Testing

```bash
# Build Debian package
dpkg-buildpackage

# Manual installation for testing
sudo cp manage-join-token /usr/bin/
sudo cp systemd/* /etc/systemd/system/
sudo systemctl daemon-reload
```

### Local Development Setup

The script requires:
- `kubeadm` binary available in PATH
- `vault` CLI configured and authenticated
- Environment variables: `VAULT_SECRET_PATH`, `VAULT_ADDR`, `VAULT_TOKEN`

### Testing the Core Functionality

```bash
# Test token generation (requires kubeadm)
kubeadm token create --print-join-command

# Test Vault integration (requires vault CLI and auth)
export VAULT_SECRET_PATH="test/path"
export VAULT_ADDR="https://your-vault.example.com"
export VAULT_TOKEN="your-token"
./manage-join-token
```

### Systemd Service Configuration

The service requires environment configuration via systemd override:

```bash
sudo systemctl edit manage-join-token.service
```

Required environment variables:
- `VAULT_SECRET_PATH` - Vault KV path for storing the token
- `VAULT_ADDR` - Vault server URL
- `VAULT_TOKEN` - Vault authentication token (typically from file)

## Release Process

### Automated GitHub Actions Pipeline
1. **Trigger**: Push tags with format `v[0-9]+.[0-9]+.[0-9]+`
2. **Artifacts**: Creates both tarball and Debian package
3. **Release**: Automated GitHub release with changelog generation
4. **Changelog**: Uses git-chglog for conventional commit formatting

### Manual Release Steps
```bash
# Update version in .github/workflows/release.yml
# Create and push tag
git tag v1.0.2
git push origin v1.0.2
```

### Version Management
- Version defined in GitHub Actions workflow (`APP_VERSION`)
- Debian changelog managed manually in `debian/changelog`
- Semantic versioning for tags and releases

## Deployment Integration

### Infrastructure Context
- **Target Environment**: Kubernetes control plane nodes
- **Dependencies**: kubeadm, vault CLI, systemd
- **Security**: Requires cluster-admin permissions for token generation
- **Monitoring**: Systemd service status and timer execution

### GitOps Integration
- Designed for Ansible/configuration management deployment
- Environment-specific Vault paths and authentication
- Systemd service overrides for site-specific configuration
- Fits into broader infrastructure automation workflows

## Key Files and Their Purpose

- `manage-join-token` - Core bash script (6 lines, single purpose)
- `systemd/` - Service and timer definitions for scheduled execution
- `debian/` - Complete Debian packaging configuration
- `.github/workflows/release.yml` - Automated build and release pipeline
- `.chglog/config.yml` - Changelog generation configuration

## Security Considerations

- Script runs with root privileges (required for kubeadm)
- Vault token must be securely managed (typically via file)
- Join tokens have limited lifetime (24 hours by default)
- Environment variable configuration prevents secrets in code
- Designed for trusted control plane node execution only