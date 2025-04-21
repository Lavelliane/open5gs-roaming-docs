# Deploying Open5GS with Roaming Support on MicroK8s

This guide explains how to deploy the Open5GS containers with roaming configuration using MicroK8s, a lightweight Kubernetes distribution that runs on almost any hardware or VM.

## Introduction to MicroK8s

MicroK8s is a CNCF certified Kubernetes distribution that focuses on being lightweight and easy to operate. It's ideal for edge computing, IoT, and development environments where resources may be limited. MicroK8s provides a simplified approach to Kubernetes deployment while offering all the essential features needed to run containerized applications.

## Prerequisites

- A Linux system with at least 4GB RAM and 20GB disk space
- `snap` package manager installed
- Basic understanding of Kubernetes concepts
- Open5GS container images (same as used in Docker deployment)

## Setup MicroK8s

### 1. Install MicroK8s

```bash
sudo snap install microk8s --classic
```

### 2. Add your user to the MicroK8s group

```bash
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
newgrp microk8s
```

### 3. Enable required MicroK8s add-ons

```bash
microk8s enable dns storage ingress dashboard repository
```

### 4. Set up kubectl alias for convenience

```bash
alias kubectl='microk8s kubectl'
```