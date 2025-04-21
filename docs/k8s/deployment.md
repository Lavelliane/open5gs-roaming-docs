# Deploying Open5GS to MicroK8s

This guide explains how to deploy Open5GS with roaming capabilities to MicroK8s using scripts from the [vagrant-open5gs-roaming](https://github.com/Lavelliane/vagrant-open5gs-roaming) and [open5gs-roaming-microk8s](https://github.com/Lavelliane/open5gs-roaming-microk8s) repositories.

## Overview

The deployment process consists of two main steps:

1. Importing Docker images to MicroK8s using `docker-to-k8s.sh`
2. Deploying the Open5GS components using `k8-deploy.sh`

## Script Explanations

### docker-to-k8s.sh

This script transfers Docker images to MicroK8s. It performs the following actions:

- Iterates through a list of required Docker images for Open5GS components
- Saves each Docker image locally using `docker save`
- Imports each image directly into MicroK8s using the container registry
- Processes all core Open5GS network functions (NRF, AMF, SMF, etc.)
- Includes the PacketRusher testing tool and MongoDB database

```bash
# Example of how the script imports an image
docker save udm:v2.7.5 | microk8s ctr image import -
```

### k8-deploy.sh

This script automates the deployment of all Open5GS components in MicroK8s. It:

- Creates an `open5gs` namespace if it doesn't exist
- Deploys components from three main directories:
  - `home/` - Home network components (AUSF, NRF, SEPP, UDM, UDR)
  - `shared/` - Shared components (MongoDB, PacketRusher)
  - `visiting/` - Visiting network components (AMF, AUSF, BSF, NRF, NSSF, PCF, SEPP, SMF, UPF)
- For each component, applies the corresponding:
  - ConfigMap (configuration data)
  - Deployment (pod specifications)
  - Service (network exposure)
- Waits for all pods to be ready
- Displays the status of all deployed resources

## Deployment Process

1. Clone the repositories:
   ```bash
   git clone https://github.com/Lavelliane/vagrant-open5gs-roaming
   git clone https://github.com/Lavelliane/open5gs-roaming-microk8s
   ```

2. Run the Docker to MicroK8s import script:
   ```bash
   # If using Vagrant:
   /vagrant/docker-to-k8s.sh
   
   # If not using Vagrant:
   cd vagrant-open5gs-roaming
   ./docker-to-k8s.sh
   ```

3. Run the Kubernetes deployment script:
   ```bash
   # If using Vagrant:
   /vagrant/k8-deploy.sh
   
   # If not using Vagrant:
   cd ../open5gs-roaming-microk8s
   ./k8-deploy.sh
   ```

4. Verify the deployment:
   ```bash
   microk8s kubectl get pods -n open5gs
   ```

## Deployment Structure

The deployment is organized to simulate roaming between two networks:

- **Home Network**: Core network components for the home operator
- **Visiting Network**: Core network components for the visiting operator
- **Shared Components**: Common resources used by both networks
