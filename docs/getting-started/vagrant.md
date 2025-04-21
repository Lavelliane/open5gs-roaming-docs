# VM Setup
This tutorial uses Vagrant & VirtualBox. You don't have to use vagrant, as long as it is on a Linux environment like Ubuntu or Debian, everything should work fine.

Vagrant provides an easy way to create and manage virtual development environments. This guide will help you set up the Open5GS Roaming environment using Vagrant with Ubuntu 22.04 (Jammy Jellyfish).

## Why Vagrant?

Vagrant offers several advantages for our Open5GS Roaming project:

- **Consistency**: Everyone uses the same environment, eliminating "it works on my machine" problems
- **Isolation**: The development environment is separate from your host system
- **Reproducibility**: New team members can get up and running quickly
- **Portability**: Works across different operating systems (Windows, macOS, Linux)

## Prerequisites

Before you begin, make sure you have the following installed:

- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (or another supported provider)

## Ubuntu 22.04 (Jammy Jellyfish)

We've chosen Ubuntu 22.04 LTS (Jammy Jellyfish) as our base operating system for several reasons:

- Long-term support until April 2027
- Excellent compatibility with Docker and MicroK8s
- Up-to-date packages for networking tools
- Widespread community support
- Stability and performance

### Vagrant File & Other scripts for Host Machine

## Setting Up Your Vagrant Environment

Follow these steps to set up your Open5GS Roaming environment:

1. Clone the repository:
   ```bash
   git clone https://github.com/Lavelliane/vagrant-open5gs-roaming.git
   cd vagrant-open5gs-roaming
   ```

2. Start the Vagrant environment:
   ```bash
   vagrant up
   ```
   This will create and provision your virtual machine according to the configuration in the Vagrantfile.

3. SSH into your Vagrant machine:
   ```bash
   vagrant ssh
   ```

4. Make the utility scripts executable:
   ```bash
   # If using Vagrant:
   chmod +x /vagrant/*.sh
   
   # If not using Vagrant (running directly on your machine):
   chmod +x *.sh
   ```

5. Fix DNS resolution for Docker by running:
   ```bash
   # If using Vagrant:
   /vagrant/fix-dns.sh
   
   # If not using Vagrant (running directly on your machine):
   ./fix-dns.sh
   ```
   This script configures proper DNS resolution required for Docker to function correctly.

6. Verify your `/etc/hosts` configuration:
   ```bash
   sudo nano /etc/hosts
   ```
   
   Your `/etc/hosts` file should include these entries:
   ```
   127.0.0.1       localhost
   # The following lines are desirable for IPv6 capable hosts
   ::1             ip6-localhost ip6-loopback
   fe00::0         ip6-localnet
   ff00::0         ip6-mcastprefix
   ff02::1         ip6-allnodes
   ff02::2         ip6-allrouters
   127.0.1.1       open5gs-roaming.open5gs.virtualbox.org  open5gs-roaming
   127.0.1.1       ubuntu-jammy     ubuntu-jammy
   ```
   
   If needed, add the missing entries and save the file.

## Using Your Vagrant Environment

Once set up, you can:

- Access your environment anytime with `vagrant ssh`
- Pause the VM with `vagrant suspend`
- Stop the VM with `vagrant halt`
- Delete the VM with `vagrant destroy`

## Troubleshooting

- If you encounter network issues, verify your host machine's network connection
- For performance issues, adjust the memory and CPU settings in the Vagrantfile
- Check logs in case of startup failures: `vagrant up --debug`




