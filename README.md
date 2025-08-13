# RKE2 Ansible Deployment

Automated deployment of Rancher Kubernetes Engine 2 (RKE2) clusters using Ansible with comprehensive configuration and management capabilities.

## Features

- **Latest RKE2 version support**: Configurable RKE2 version deployment
- **Flexible authentication**: SSH password and key-based authentication
- **System prerequisites**: Automated swap disable and system preparation
- **High availability ready**: Multi-master node support with proper sequencing
- **Registry support**: Custom container registry configuration
- **Complete lifecycle management**: Installation, configuration, and removal playbooks
- **Node management**: Automated hostname assignment and cluster joining

## Architecture

This Ansible playbook collection provides:

- **Master nodes**: RKE2 server configuration with etcd and control plane
- **Worker nodes**: RKE2 agent configuration for workload scheduling  
- **Networking**: Canal CNI with configurable pod/service CIDRs
- **Security**: TLS certificate management and join token handling

## Quick Start

### 1. Configure Inventory

Update `inventory/hosts.yml` with your node information:

```yaml
all:
  children:
    rke2_cluster:
      children:
        rke2_servers:
          hosts:
            master-node:
              ansible_host: YOUR_MASTER_IP
              ansible_user: YOUR_USERNAME
              ansible_ssh_pass: YOUR_PASSWORD
        rke2_agents:
          hosts:
            worker-node-1:
              ansible_host: YOUR_WORKER_IP_1  
              ansible_user: YOUR_USERNAME
              ansible_ssh_pass: YOUR_PASSWORD
            worker-node-2:
              ansible_host: YOUR_WORKER_IP_2
              ansible_user: YOUR_USERNAME  
              ansible_ssh_pass: YOUR_PASSWORD
```

### 2. Configure Variables

Edit `group_vars/all.yml` to set your desired RKE2 version and networking:

```yaml
rke2_version: "v1.32.7+rke2r1"
rke2_service_cidr: "10.43.0.0/16"
rke2_pod_cidr: "10.42.0.0/16"
```

### 3. Test Connectivity

```bash
ansible all -i inventory/hosts.yml -m ping
```

### 4. Deploy RKE2 Cluster

```bash
ansible-playbook -i inventory/hosts.yml playbooks/rke2-install.yml
```

### 5. Verify Deployment

```bash
# SSH to your master node
ssh YOUR_USERNAME@YOUR_MASTER_IP
sudo kubectl get nodes
```

Expected output:
```
NAME           STATUS   ROLES                       AGE   VERSION
master-node    Ready    control-plane,etcd,master   5m    v1.32.7+rke2r1
worker-node-1  Ready    <none>                      2m    v1.32.7+rke2r1
worker-node-2  Ready    <none>                      2m    v1.32.7+rke2r1
```

## Available Playbooks

| Playbook | Purpose |
|----------|---------|
| `rke2-install.yml` | Main installation with node naming |
| `copy-registries.yml` | Copy registries.yaml to all nodes |
| `rke2-uninstall.yml` | Complete RKE2 removal |
| `cleanup-old-nodes.yml` | Remove duplicate/old nodes |

## Configuration Options

### Node Naming
The playbook automatically assigns hostnames based on IP addresses. You can customize the hostname mapping logic in the pre_tasks section of `playbooks/rke2-install.yml`.

### Network Configuration
Default network settings can be modified in `group_vars/all.yml`:
- **Service CIDR**: `10.43.0.0/16`  
- **Pod CIDR**: `10.42.0.0/16`
- **CNI Plugin**: Canal (configurable)

## Registry Configuration

To use custom registries:

1. Place your `registries.yaml` in `files/` directory
2. Run: `ansible-playbook -i inventory/hosts.yml playbooks/copy-registries.yml`

## Uninstall

Complete RKE2 removal:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/rke2-uninstall.yml
```

With hostname revert and reboot:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/rke2-uninstall.yml -e "revert_hostnames=true reboot_after_uninstall=true"
```

## Requirements

- **OS**: Ubuntu/CentOS/RHEL
- **Access**: SSH with sudo privileges
- **Ansible**: 2.9+
- **Network**: All nodes must communicate on ports 6443, 9345

## Troubleshooting

- **Syntax check**: `ansible-playbook --syntax-check playbooks/*.yml`
- **Dry run**: `ansible-playbook --check -i inventory/hosts.yml playbooks/rke2-install.yml`
- **Verbose output**: Add `-v` flag to any playbook command

For detailed configuration options, see `CLAUDE.md`.

