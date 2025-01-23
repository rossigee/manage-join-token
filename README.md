# Manage K8S join token

This program is designed to be run on a regular schedule on a control plane node to generate a new K8S join token and store it in a Vault secret.

the primary use case for this is to automate the bootstrapping of a newly launched cluster node. The newly launched node will have been provisioned with a temporary 'bootstrap' Vault token, which should allow it to obtain the K8S join token and any other keys or secrets it may need to complete it's bootstrap.

## Basic usage

Manual install.

Copy the script into place.

```bash
cp -a manage-join-token /usr/bin/manage-join-token
```

Copy the systemd scripts into place, and enable the scheduled service.

```bash
cp -a systemd/* /etc/systemd/system
systemctl enable manage-join-token.timer
systemctl start manage-join-token.timer
```

Override the service to provide the Vault KV location to which to write the join token.

```bash
systemctl edit manage-join-token.service
```

Add in the following site-specifics...

```
[Service]
Environment=VAULT_SECRET_PATH=k8s-your-clusters/some-cluster
Environment=VAULT_ADDR=https://vault.yourdomain.com
EnvironmentFile=/etc/kubernetes/vault-token
```

In this case, we have our `VAULT_TOKEN` stored in the `/etc/kubernetes/vault-token` file.

It is assumed that the necessary Vault environment, including any authentication configuration, has already been set up.
