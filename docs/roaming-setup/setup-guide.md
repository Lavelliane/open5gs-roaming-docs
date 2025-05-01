# Setup Guide

This guide provides a comprehensive, sequential walkthrough of setting up a 5G roaming environment using Open5GS on Docker.

## 1. Architecture Overview

The roaming setup simulates communication between two network operators:

- **Home Network (H-PLMN)** with MCC 001, MNC 01
- **Visiting Network (V-PLMN)** with MCC 999, MNC 70

The components in this architecture include:

### Home Network Components (MCC-001, MNC-01)

- **h-nrf**: Network Repository Function - Central registry for network functions
- **h-ausf**: Authentication Server Function - Handles authentication
- **h-udm**: Unified Data Management - Manages subscriber data
- **h-udr**: Unified Data Repository - Stores subscriber data
- **h-sepp**: Security Edge Protection Proxy - Secures inter-PLMN communication

### Visiting Network Components (MCC-999, MNC-70)

- **v-nrf**: Network Repository Function - Central registry for network functions
- **v-ausf**: Authentication Server Function - Handles authentication
- **v-nssf**: Network Slice Selection Function - Manages network slicing
- **v-bsf**: Binding Support Function - Assists with policy binding
- **v-pcf**: Policy Control Function - Manages policy enforcement
- **v-amf**: Access and Mobility Management Function - Manages mobility and access
- **v-smf**: Session Management Function - Manages user sessions
- **v-upf**: User Plane Function - Handles user data traffic
- **v-sepp**: Security Edge Protection Proxy - Secures inter-PLMN communication

### Supporting Components

- **db**: MongoDB database for storing subscriber information
- **webui**: Web user interface for managing the system
- **tshark**: Packet capture utility
- **packetrusher**: gNB and UE simulator

### Network Topology

All components run as Docker containers on a bridge network `open5gs` with subnet `10.33.33.0/24`.

## 2. System Requirements

- Ubuntu 22.04 LTS (recommended)
- Minimum 8GB RAM
- Minimum 4 CPU cores
- 50GB of free disk space
- Virtualization capability if running on a non-Linux OS

## 3. Environment Setup

### 3.1 Setting Up Ubuntu (if needed)

If not running Ubuntu 22.04 natively, set up a virtual machine using VirtualBox or VMware:

1. Download virtualization software (VirtualBox/VMware)
2. Download Ubuntu 22.04 LTS ISO
3. Create a VM with at least 8GB RAM, 4 cores, and 50GB disk space
4. Install Ubuntu and update the system:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

### 3.2 Installing Required Dependencies

All required dependencies can be installed automatically using the provided script:

1. Clone the repository:

   ```bash
   git clone https://github.com/roastedbeans/open5gs-roaming.git
   cd open5gs-roaming
   ```

2. Run the installation script:
   ```bash
   chmod +x install-dep.sh
   ./install-dep.sh
   ```

This script installs:

- Docker and Docker Compose
- Git
- GTP5G Kernel Module
- Other necessary dependencies

## 4. Building Docker Images

Choose one of these methods to build the Docker images:

### 4.1 Using Docker Bake (Recommended)

```bash
# Build all components at once
docker buildx bake -f docker-bake.hcl
```

### 4.2 Using Make

```bash
# Build all components
make all
```

### 4.3 Verify the Build

Check that all images were created successfully:

```bash
docker images
```

You should see images for base-open5gs, all network functions (amf, ausf, etc.), and webui.

## 5. Configuration

The configuration files for all components are located in the `configs/roaming` directory:

### 5.1 Key Configuration Parameters

- **Home Network (H-PLMN)**: MCC 001, MNC 01
- **Visiting Network (V-PLMN)**: MCC 999, MNC 70
- **Network Slice**: SST 01, SD 000001
- **SEPP TLS Configuration**:
  - Both SEPPs use their own private keys and certificate chains
  - Both verify peer certificates using a shared CA bundle
  - h-sepp connects to Home NRF at `nrf.5gc.mnc001.mcc001.3gppnetwork.org:80`
  - v-sepp connects to Visiting NRF at `nrf.5gc.mnc070.mcc999.3gppnetwork.org:80`

### 5.2 PacketRusher (UE & gNB) Configuration

- **gNB Configuration**:
  - Control Interface: `gnb.packetrusher.org:38412`
  - Data Interface: `gnb.packetrusher.org:2152`
  - PLMN: MCC 999, MNC 70, TAC 000001
  - gNB ID: 000008
  - Slice Support: SST 01, SD 000001
- **UE Configuration**:
  - IMSI: 001010000000001
  - Key (K): 7F176C500D47CF2090CB6D91F4A73479
  - OPC: 3D45770E83C7BBB6900F3653FDA6330F

## 6. Starting the Roaming Environment

### 6.1 Launch the Docker Compose Setup

```bash
cd open5gs-roaming
docker compose -f compose-files/roaming/docker-compose.yaml --env-file=.env up -d
```

### 6.2 Verify Container Status

```bash
docker compose ps
```

All containers should be in the "Up" state.

### 6.3 Adding a Subscriber

1. Access the WebUI at `http://localhost:9999`
2. Login with default credentials:
   - Username: `admin`
   - Password: `1423`
3. Go to "Subscriber" in the left menu and click "+"
4. Enter the following information:
   - IMSI: `001010000000001`
   - Subscriber Key (K): `7F176C500D47CF2090CB6D91F4A73479`
   - OPc: `3D45770E83C7BBB6900F3653FDA6330F`
   - Slice Configuration:
     - SST: `1`
     - SD: `000001`
     - Session-AMBR Downlink: `1 Gbps`
     - Session-AMBR Uplink: `1 Gbps`
     - Default 5QI: `9`
   - Data Network Name (DNN):
     - DNN: `internet`
     - IPv4: Selected
5. Click "Save" to add the subscriber

## 7. Monitoring and Verification

### 7.1 Viewing Container Logs

```bash
# View all logs
docker compose logs

# View logs for a specific container
docker compose logs v-amf

# Follow logs continuously
docker compose logs -f
```

### 7.2 Verify SEPP TLS Setup

```bash
docker exec -it h-sepp ls -la /etc/open5gs/default/tls
docker exec -it v-sepp ls -la /etc/open5gs/default/tls
```

### 7.3 Check NRF Registrations

```bash
# Home NRF
docker exec -it h-nrf curl -s http://nrf.5gc.mnc001.mcc001.3gppnetwork.org:7777/nnrf-nfm/v1/nf-instances | jq

# Visiting NRF
docker exec -it v-nrf curl -s http://nrf.5gc.mnc070.mcc999.3gppnetwork.org:7777/nnrf-nfm/v1/nf-instances | jq
```

## 8. Testing UE Connectivity

### 8.1 Verify UE Registration

Check the PacketRusher logs for successful registration:

```bash
docker compose logs packetrusher
```

Look for messages indicating successful registration and PDU session establishment.

### 8.2 Test Data Connectivity

```bash
docker exec -it packetrusher /bin/sh
ping -I uesimtun0 8.8.8.8
```

A successful ping confirms that the UE has connected to the 5G network with internet access.

## 9. Packet Capture and Analysis

To capture network traffic for analysis:

```bash
# Start a capture
docker exec -it tshark tshark -i any -w /captures/roaming_test.pcap

# In another terminal, press Ctrl+C to stop the capture when done
```

The capture file will be saved in the `captures` volume and can be analyzed using Wireshark.

## 10. Troubleshooting Common Issues

### 10.1 Container Startup Failures

- Check individual container logs for error messages
- Verify that all configuration files are correctly formatted
- Ensure the Docker bridge network is properly created

### 10.2 UE Registration Failures

- Verify the subscriber has been correctly added in the WebUI
- Check that the UE credentials in PacketRusher match those in the database
- Examine v-amf logs for authentication or registration errors

### 10.3 Missing NRF Registrations

- Check NRF logs for connection issues
- Verify DNS resolution is working correctly within the containers
- Restart the affected network functions

## 11. Stopping the System

```bash
# Stop all containers while preserving data
docker compose down

# Completely remove all containers, networks, and volumes
docker compose down -v
```
