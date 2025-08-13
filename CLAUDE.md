
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an RKE2 Ansible automation project for deploying and managing Rancher Kubernetes Engine 2 clusters. The project provides a complete automation solution for setting up highly available Kubernetes clusters using RKE2.

## Architecture

The repository follows standard Ansible best practices with the following structure:

- **`inventory/`**: Contains host inventory files with master and worker node definitions
- **`playbooks/`**: Main playbooks for cluster deployment and management
  - `rke2-install.yml`: Primary installation playbook
- **`roles/`**: Reusable Ansible roles
  - `rke2-server/`: Handles master node configuration and cluster initialization
  - `rke2-agent/`: Manages worker node setup and cluster joining
- **`group_vars/`**: Environment and group-specific variables
- **`files/`**: Static files for deployment

## Common Commands

```bash
# Test connectivity to all nodes
ansible all -i inventory/hosts.yml -m ping

# Deploy RKE2 cluster
ansible-playbook -i inventory/hosts.yml playbooks/rke2-install.yml

# Check syntax of playbooks
ansible-playbook --syntax-check playbooks/*.yml

# Run in check mode (dry run)
ansible-playbook -i inventory/hosts.yml --check playbooks/rke2-install.yml

# Deploy only server nodes
ansible-playbook -i inventory/hosts.yml playbooks/rke2-install.yml --limit rke2_servers

# Deploy only agent nodes  
ansible-playbook -i inventory/hosts.yml playbooks/rke2-install.yml --limit rke2_agents
```

## Configuration

Before deployment:

1. **Update inventory**: Edit `inventory/hosts.yml` with your actual master and worker node IP addresses
2. **Configure variables**: Modify `group_vars/` files for your environment:
   - `all.yml`: Global cluster settings
   - `rke2_servers.yml`: Master node specific configuration
   - `rke2_agents.yml`: Worker node specific configuration
3. **TLS SANs**: Add load balancer IPs/hostnames to `rke2_tls_san` in `group_vars/rke2_servers.yml`

## Key Implementation Details

- First server node initializes the cluster with `cluster-init: true`
- Subsequent server nodes join using the first server's endpoint
- Join tokens are automatically generated and shared between nodes
- Server nodes are tainted by default to prevent workload scheduling
- Canal CNI is used by default but can be changed via `rke2_cni` variable
- Cluster verification is performed after deployment
