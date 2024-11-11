# Install K3s on Linux

## Prerequisites

- A Linux machine with a unique hostname.

- Ensure your system meets the K3s [requirements](https://docs.k3s.io/reference/resource-profiling).

## Step 1: Install K3s

K3s provides an installation script that makes it easy to install K3s as a service on systemd or openrc based systems. To install K3s, run the following command:

```bash
curl -sfL https://get.k3s.io | sh -
```

What the Script Does:

- Installs K3s and its dependencies.

- Configures K3s to start automatically after node reboots or if the process crashes.

- Installs additional utilities including `kubectl`, `crictl`, `ctr`, `k3s-killall.sh`, and `k3s-uninstall.sh`.

- Writes a kubeconfig file to `! k3s.yaml` and configures `kubectl` to use it.

## Step 2: Verify Installation

After the installation, verify that K3s is running and the node is ready:

```bash
sudo k3s kubectl get nodes
```

## Step 3: Configure K3s for Your User

To use `kubectl` as a non-root user, copy the kubeconfig file to your home directory and set the appropriate permissions:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER ~/.kube/config
sudo chmod 600 ~/.kube/config

# Optionally, add this line to your .bashrc or .zshrc to set KUBECONFIG environment variable
export KUBECONFIG=~/.kube/config
```

## Step 4: Install Additional Agent Nodes

To add additional agent nodes to your cluster, run the installation script with the `K3S_URL` and `K3S_TOKEN` environment variables. Replace `https://myserver:6443` with the URL of your K3s server and `mynodetoken` with the token from your server node.

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

## Note

- The `K3S_URL` parameter configures K3s as an agent instead of a server.

- The `K3S_TOKEN` value can be found at `/var/lib/rancher/k3s/server/node-token` on  server node.

## Step 5: Install kubectl (Optional)

If you haven't already installed kubectl, you can do so with the following commands:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

## Step 6: Enable Command Completion for kubectl (Optional)

To enable command completion for kubectl, add the following to your shell configuration file (`~/.bashrc` or `~/.zshrc`):

```bash
source <(kubectl completion bash)
```

For zsh users, use:

```bash
source <(kubectl completion zsh)
```

## Or Simply Execute This Bash Script

```bash
#!/bin/bash

# Disable firewall (for development only)
sudo systemctl disable firewalld --now

# Install K3s
curl -sfL https://get.k3s.io | sh -

# Verify installation
sudo k3s kubectl get nodes

# Configure kubeconfig for the current user
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER ~/.kube/config
sudo chmod 600 ~/.kube/config

# Add KUBECONFIG to .bashrc
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
source ~/.bashrc

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Enable kubectl command completion
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc

# For zsh users
if [ -n "$ZSH_VERSION" ]; then
  echo 'source <(kubectl completion zsh)' >> ~/.zshrc
  source ~/.zshrc
fi

echo "K3s installation and configuration complete!"
```
