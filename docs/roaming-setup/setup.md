# Environment Setup

This section covers how to set up the environment needed for running the Open5GS roaming setup.

## System Requirements

- Ubuntu 22.04 LTS (recommended)
- Minimum 8GB RAM
- Minimum 4 CPU cores
- 50GB of free disk space
- Virtualization capability if running on a non-Linux OS

## Setting Up Ubuntu 22.04 VM (if needed)

If you're not running Ubuntu 22.04 natively, you'll need to set up a virtual machine:

### Option 1: Using VirtualBox

1. Download and install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. Download the [Ubuntu 22.04 LTS ISO](https://releases.ubuntu.com/22.04/)
3. Create a new virtual machine in VirtualBox:
   - Type: Linux
   - Version: Ubuntu (64-bit)
   - Memory: 8GB or more
   - Create a virtual hard disk (VDI) of at least 50GB
4. Start the VM and install Ubuntu 22.04
5. Once installed, update the system:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

### Option 2: Using VMware

1. Download and install [VMware Workstation Player](https://www.vmware.com/products/workstation-player.html) (Windows/Linux) or [VMware Fusion](https://www.vmware.com/products/fusion.html) (macOS)
2. Download the [Ubuntu 22.04 LTS ISO](https://releases.ubuntu.com/22.04/)
3. Create a new virtual machine in VMware:
   - Select the Ubuntu 22.04 ISO
   - Allocate at least 8GB RAM
   - Allocate at least 4 CPU cores
   - Create a disk of at least 50GB
4. Complete the Ubuntu installation
5. Update the system:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

## Required Software Components

Before proceeding with the Open5GS roaming setup, you need to install the following software:

1. Docker and Docker Compose
2. Git
3. GTP5G Kernel Module
4. Wireshark (for packet capture analysis)

## Installing Dependencies Automatically

The repository includes an installation script that will set up all the required dependencies.

1. Clone the repository:

   ```bash
   git clone https://github.com/roastedbeans/open5gs-roaming.git
   cd open5gs-roaming
   ```

2. Make the installation script executable and run it:
   ```bash
   chmod +x install-dep.sh
   ./install-dep.sh
   ```

The script will:

- Install Docker and configure it for non-root usage
- Install Git
- Install the GTP5G kernel module for 5G user plane functionality

## Manual Installation (if needed)

If you prefer to install the components manually, follow these steps:

### Docker Installation

```bash
# Remove old Docker versions
sudo apt-get remove -y docker docker-engine docker.io containerd runc

# Update package index and install dependencies
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Set up the Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add current user to docker group
sudo groupadd docker
sudo usermod -aG docker $USER

# Log out and log back in, or run:
newgrp docker
```

### Git Installation

```bash
sudo apt-get update
sudo apt-get install -y git
```

### GTP5G Kernel Module Installation

```bash
# Install build dependencies
sudo apt-get install -y build-essential linux-headers-$(uname -r) git

# Clone and build GTP5G module
cd /usr/src
sudo git clone https://github.com/free5gc/gtp5g.git
cd gtp5g
sudo make clean
sudo make
sudo make install

# Load the module
sudo modprobe gtp5g

# Make it load at boot
echo "gtp5g" | sudo tee /etc/modules-load.d/gtp5g.conf
```

### Wireshark Installation

```bash
sudo apt-get update
sudo apt-get install -y wireshark
```

When asked about non-root users capturing packets, select "Yes" to allow it.

## Next Steps

After completing the environment setup, proceed to the [System Architecture](/roaming-setup/architecture) page to understand the components of the Open5GS roaming setup.
