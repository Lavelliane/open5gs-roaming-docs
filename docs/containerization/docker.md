# Open5GS Roaming Configuration with Docker

Welcome to your comprehensive guide for deploying Open5GS with roaming capabilities using Docker! This guide documents the automated setup process from the [vagrant-open5gs-roaming](https://github.com/Lavelliane/vagrant-open5gs-roaming) repository that makes deploying a complete 5G core network with roaming support a breeze.

## 📋 About This Project

This project provides a Vagrant-based environment for deploying Open5GS with roaming capabilities between two networks. The automation script handles the deployment of a complete 5G core network with inter-PLMN roaming support using Docker containers.

### Key Features
- Fully automated deployment of Open5GS 5G core network
- Pre-configured roaming between home network (MCC: 001, MNC: 01) and visiting network (MCC: 999, MNC: 70)
- Includes test subscribers for both networks
- PacketRusher integration for UE simulation and testing
- MongoDB database for subscriber management
- Helper scripts for adding and managing subscribers

## 🚀 What The Setup Script Does

Our setup script (`setup-open5gs.sh`) automates the entire deployment process of Open5GS with roaming configuration, handling everything from network configuration to subscriber provisioning. Let's explore what it does for you:

### 1. System Configuration
- Updates the `/etc/hosts` file with proper hostname mapping
- Configures the network environment for proper communication

### 2. Environment Setup
- Sets MongoDB to version 4.4 for optimal compatibility
- Automatically detects your VM's IP address
- Updates Docker configuration with the correct network settings

### 3. Deployment Process
- Builds all necessary containers using Docker Buildx
- Starts MongoDB first to ensure database availability
- Adds test subscribers to the database including:
  - PacketRusher test UE (IMSI: 001011234567891)
  - Additional test subscribers for home and visiting networks
- Launches all Open5GS services with proper roaming configuration

### 4. Helpful Utilities
- Creates an `add-subscriber.sh` script for easily adding more subscribers
- Provides verification and debugging commands
- Includes complete usage documentation

## 🔧 Usage Instructions

### Starting the Environment
In the Vagrant VM, run the setup script:
```bash
/vagrant/setup-open5gs.sh
```
If you are not using Vagrant, run as you would on your preferred directory.
```bash
./setup-open5gs.sh
```

### Managing Subscribers
To add additional subscribers after deployment:
```bash
/home/vagrant/add-subscriber.sh <imsi> <key> <opc>
```

Example:
```bash
/home/vagrant/add-subscriber.sh 001010000000002 465B5CE8B199B49FAA5F0A2EE238A6BC E8ED289DEBA952E4283B54E88E6183CA
```

### Managing Services
- **View logs**: 
  ```bash
  docker compose -f compose-files/roaming/docker-compose.yaml --env-file=.env logs -f
  ```
- **Restart a service**: 
  ```bash
  docker compose -f compose-files/roaming/docker-compose.yaml --env-file=.env restart <service-name>
  ```
- **Stop all services**: 
  ```bash
  docker compose -f compose-files/roaming/docker-compose.yaml --env-file=.env down
  ```
- **Check UE registration**: 
  ```bash
  docker logs -f packetrusher
  ```

## 📋 Default Configuration

After deployment, you'll have access to:
- MongoDB on `localhost:27017`
- AMF N2 interface on `<VM_IP>:38412` (SCTP)
- UPF N3 interface on `<VM_IP>:2152` (UDP)

### Pre-configured Subscribers
1. PacketRusher test UE: IMSI 001011234567891
2. Home Network: IMSI 001010000000001
3. Visiting Network: IMSI 999700000000001

## 🔍 Troubleshooting

If you encounter any issues:
1. Check if all containers are running with `docker ps`
2. Examine container logs for specific services
3. Verify network configuration and IP addressing
4. Ensure MongoDB is properly initialized

## 🛠️ Advanced Configuration

For advanced users who need to customize the deployment, you can:
- Modify the `.env` file to change service parameters
- Edit the MongoDB initialization scripts to add different subscribers
- Adjust the docker-compose file for different network or service configurations

## 📚 Additional Resources

- [GitHub Repository](https://github.com/Lavelliane/vagrant-open5gs-roaming) - Source code and additional scripts
- [Open5GS Documentation](https://open5gs.org/open5gs/docs/) - Official documentation for Open5GS
- [Vagrant Documentation](https://www.vagrantup.com/docs) - Information about Vagrant VM management

Happy deploying! 🎉
