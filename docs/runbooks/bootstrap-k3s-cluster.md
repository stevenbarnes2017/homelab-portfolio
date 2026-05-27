# Runbook: Bootstrap k3s Cluster on Proxmox

## Overview

This runbook documents the process for provisioning the homelab Kubernetes platform from Proxmox VMs through a functional k3s cluster.

The workflow uses:

- Terraform to provision Proxmox virtual machines
- Ansible to configure the operating system
- Ansible playbooks to install:
  - Kubernetes control plane nodes
  - Worker nodes
  - Load balancer
  - Longhorn prerequisites

---

## Build Flow

```text
Proxmox
  ↓
Terraform provisions VMs
  ↓
Ansible configures OS baseline
  ↓
Ansible installs k3s control planes
  ↓
Ansible installs worker agents
  ↓
Ansible configures load balancer
  ↓
Longhorn prerequisites installed
  ↓
Cluster ready for GitOps workloads

Prerequisites
Proxmox host available
Cloud-init templates created
SSH key access configured
Terraform installed
Ansible installed
DNS entries planned
Sufficient CPU, RAM, and disk for Kubernetes + Longhorn
Terraform Provisioning

Terraform is used to create the VM infrastructure on Proxmox.

Responsibilities
Clone VM templates
Assign CPU and memory
Attach disks
Configure cloud-init
Inject SSH keys
Assign hostnames
Assign network settings
Expected Output
Control plane VMs created
Worker VMs created
Load balancer VM created
SSH access available from Ansible controller
Ansible Baseline Configuration

Ansible is used to prepare the nodes for Kubernetes.

Baseline Tasks
Update packages
Configure hostnames
Configure time synchronization
Install common tools
Configure SSH access
Disable swap if needed
Ensure required kernel modules are available
Load Balancer Setup

A dedicated load balancer is configured in front of the k3s control plane nodes.

Purpose
Provides a stable Kubernetes API endpoint
Supports control plane failover
Prevents clients from depending on a single control plane node
Responsibilities
Install load balancer package
Configure backend control plane nodes
Expose Kubernetes API endpoint
k3s Control Plane Installation

The first control plane node initializes the cluster.

Additional control plane nodes join using the shared cluster token.

Validation
kubectl get nodes
kubectl get pods -A

Expected:

All control plane nodes are Ready
Core system pods are Running
k3s Worker Installation

Worker nodes join the cluster as agents.

Validation
kubectl get nodes -o wide

Expected:

Worker nodes appear in Ready state
Workloads can be scheduled onto workers
Longhorn Prerequisites

Before installing Longhorn, nodes must have required dependencies.

Common Requirements
open-iscsi
nfs-common
iscsi services enabled
sufficient disk space
mount propagation support
compatible filesystem layout
Validation

Run Longhorn prerequisite checks before deploying Longhorn.

Post-Install Validation
kubectl get nodes
kubectl get pods -A
kubectl get storageclass

Confirm:

All nodes Ready
CoreDNS Running
Traefik Running
Longhorn Ready
Default storage class configured as intended
Known Issues / Lessons Learned
Longhorn requires host-level prerequisites before deployment
Kubernetes storage issues are often node dependency issues, not Helm chart issues
Load balancer stability is critical for HA control plane behavior
Documenting node bootstrap makes rebuilds much easier
Future Improvements
Fully automate cluster rebuild from scratch
Add validation tasks to Ansible
Add Longhorn backup target
Add disaster recovery documentation
Add version pinning for k3s, Terraform provider, and Helm charts

This one is very portfolio-worthy because it shows the complete platform lifecycle: **provisio
