# System Architecture

This section describes the architecture of the Open5GS roaming setup. The system simulates a roaming scenario between two networks: a Home PLMN (H-PLMN) and a Visiting PLMN (V-PLMN).

## Overview

The roaming setup consists of:

- A **Home Network (H-PLMN)** with MCC 001, MNC 01
- A **Visiting Network (V-PLMN)** with MCC 999, MNC 70
- **PacketRusher** as the gNB and UE simulator
- **TLS-secured SEPP** components for inter-PLMN security

## Network Components

### Home Network Components (MCC-001, MNC-01)

The Home Network consists of the following components:

- **h-nrf**: Network Repository Function - Central registry for network functions
- **h-ausf**: Authentication Server Function - Handles authentication
- **h-udm**: Unified Data Management - Manages subscriber data
- **h-udr**: Unified Data Repository - Stores subscriber data
- **h-sepp**: Security Edge Protection Proxy - Secures inter-PLMN communication

### Visiting Network Components (MCC-999, MNC-70)

The Visiting Network consists of the following components:

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

## Network Topology

All components run as Docker containers on a bridge network named `open5gs` with subnet `10.33.33.0/24`.

### IP Addressing Scheme

| Component    | IP Address  | FQDN                                   |
| ------------ | ----------- | -------------------------------------- |
| h-nrf        | 10.33.33.10 | nrf.5gc.mnc001.mcc001.3gppnetwork.org  |
| h-ausf       | 10.33.33.11 | ausf.5gc.mnc001.mcc001.3gppnetwork.org |
| h-udm        | 10.33.33.12 | udm.5gc.mnc001.mcc001.3gppnetwork.org  |
| h-udr        | 10.33.33.13 | udr.5gc.mnc001.mcc001.3gppnetwork.org  |
| h-sepp       | 10.33.33.20 | sepp.5gc.mnc001.mcc001.3gppnetwork.org |
| v-nrf        | 10.33.33.30 | nrf.5gc.mnc070.mcc999.3gppnetwork.org  |
| v-ausf       | 10.33.33.31 | ausf.5gc.mnc070.mcc999.3gppnetwork.org |
| v-nssf       | 10.33.33.32 | nssf.5gc.mnc070.mcc999.3gppnetwork.org |
| v-bsf        | 10.33.33.33 | bsf.5gc.mnc070.mcc999.3gppnetwork.org  |
| v-pcf        | 10.33.33.34 | pcf.5gc.mnc070.mcc999.3gppnetwork.org  |
| v-amf        | 10.33.33.35 | amf.5gc.mnc070.mcc999.3gppnetwork.org  |
| v-smf        | 10.33.33.36 | smf.5gc.mnc070.mcc999.3gppnetwork.org  |
| v-upf        | 10.33.33.37 | upf.5gc.mnc070.mcc999.3gppnetwork.org  |
| v-sepp       | 10.33.33.21 | sepp.5gc.mnc070.mcc999.3gppnetwork.org |
| db           | N/A         | db.open5gs.org                         |
| webui        | N/A         | webui.open5gs.org                      |
| packetrusher | N/A         | gnb.packetrusher.org                   |

## SEPP and TLS Setup

The roaming setup includes Security Edge Protection Proxies (SEPPs) that secure the communication between the two networks using TLS:

- **h-sepp** (Home SEPP): Serves on ports 7777 (SBI), 7778 (N32c), and 7779 (N32f)
- **v-sepp** (Visiting SEPP): Serves on ports 8777 (SBI), 8778 (N32c), and 8779 (N32f)

Both SEPPs generate their TLS certificates during initialization. The certificates are stored in Docker volumes:

- `sepp_certs`: Stores the TLS certificates
- `sepp_ca`: Stores the Certificate Authority information

The SEPPs communicate with each other over the following interfaces:

- **N32-c**: Control plane for capability negotiation and security parameter exchange
- **N32-f**: Forwarding interface for actual message exchange between PLMNs

## User Equipment Configuration

The PacketRusher simulator is configured with the following UE parameters:

- **IMSI**: 001010000000001 (MCC 001, MNC 01)
- **Key**: 7F176C500D47CF2090CB6D91F4A73479
- **OPC**: 3D45770E83C7BBB6900F3653FDA6330F
- **Default DNN**: internet
- **Network Slice**: SST: 01, SD: 000001

## Next Steps

Proceed to the [Installation](/roaming-setup/installation.md) page to set up the Open5GS roaming environment.
