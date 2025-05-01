# Open5GS Roaming Setup

Welcome to the Open5GS Roaming Setup documentation. This section provides comprehensive guidance on setting up a 5G roaming environment using Open5GS on Docker.

## Overview

The roaming setup simulates communication between two network operators:

- A **Home Network (H-PLMN)** with MCC 001, MNC 01
- A **Visiting Network (V-PLMN)** with MCC 999, MNC 70

Our documentation walks you through the entire process from understanding the architecture to testing the roaming functionality.

## Documentation Sections

### [Architecture](/roaming-setup/architecture.md)

Understand the overall system design, including network components, IP addressing scheme, and how the different elements interact with each other.

### [Installation](/roaming-setup/installation.md)

Learn how to install and build the necessary Docker images for the Open5GS roaming environment.

### [Configuration](/roaming-setup/configuration.md)

Configure the various components of the roaming setup, including the home and visiting networks.

### [Setup](/roaming-setup/setup.md)

Set up your environment with the prerequisites needed for running the Open5GS roaming system.

### [Running](/roaming-setup/running.md)

Start and manage the containers that make up the roaming environment.

### [Testing](/roaming-setup/testing.md)

Verify that your roaming setup is working correctly through various tests.

### [Packet Capture](/roaming-setup/packet-capture.md)

Capture and analyze network traffic to understand the communication between components.

### [Troubleshooting](/roaming-setup/troubleshooting.md)

Resolve common issues that may arise during the setup and operation of your roaming environment.

## Getting Started

We recommend starting with the [Architecture](/roaming-setup/architecture.md) documentation to understand the overall system, and then proceeding through the other sections in the order listed above.
