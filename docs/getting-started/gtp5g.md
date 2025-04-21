# Installing gtp5g: 5G GTP Kernel Module

gtp5g is a specialized Linux kernel module designed to handle network packets according to 5G PFCP specifications, working with Packet Detection Rules (PDR) and Forwarding Action Rules (FAR). This guide walks you through installing gtp5g on your system.

## Compatibility

The module is not compatible with all Linux kernel versions due to kernel evolution. For optimal performance, use one of these kernel versions:
- `5.0.0-23-generic`
- Any version above `5.4` (Ubuntu 20.04)
- RHEL 8

## Installation Steps

### 1. Get the Source Code

You can clone either the latest version or a specific version:

For the latest version:
```
git clone https://github.com/free5gc/gtp5g.git
```

For a specific version (e.g., v0.8.10):
```
git clone -b v0.8.10 https://github.com/free5gc/gtp5g.git
```

### 2. Install Dependencies

Install the required development packages:
```
sudo apt -y update
sudo apt -y install gcc g++ cmake autoconf libtool pkg-config libmnl-dev libyaml-dev
```

### 3. Compile the Module

Navigate to the source directory and compile:
```
cd gtp5g
make clean && make
```

### 4. Install the Kernel Module

This will install the module to your system and configure it to load automatically at boot:
```
sudo make install
```

The kernel module should now be installed and ready to use.
