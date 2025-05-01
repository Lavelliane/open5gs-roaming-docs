# Installation

This section guides you through installing the Open5GS roaming environment.

## Prerequisites

Ensure you have completed the [Environment Setup](setup.html) and have:

- Ubuntu 22.04 with all updates
- Docker and Docker Compose installed
- Git installed
- GTP5G kernel module installed

## Cloning the Repository

If you haven't already, clone the repository:

```bash
git clone https://github.com/roastedbeans/open5gs-roaming.git
cd open5gs-roaming
```

## Building the Docker Images

There are two methods available for building the Docker images: using `make` or using `docker bake`.

### Option 1: Using Docker Bake

Docker Bake is a high-level build command that uses BuildKit to build multiple images efficiently.

```bash
# Build all components at once
docker buildx bake -f docker-bake.hcl

# Build specific components
docker buildx bake -f docker-bake.hcl base
docker buildx bake -f docker-bake.hcl nrf ausf udm
```

This method provides better caching and parallel builds, which can significantly reduce build time.

### Option 2: Using Make

The project also provides a Makefile for a more traditional build approach.

#### Build All Components

To build all Open5GS components at once:

```bash
make all
```

This will build the following images:

- base-open5gs (the base image for all components)
- amf, ausf, bsf, nrf, nssf, pcf, scp, sepp, smf, udm, udr, upf (network functions)
- webui (Web user interface)

#### Build Components Individually

You can also build components individually if needed:

```bash
# Build base image first
make base-open5gs

# Then build individual components
make amf
make ausf
make udm
# etc.
```

### Building Time

Building all images for the first time may take 15-30 minutes depending on your system's resources and internet connection speed. Using Docker Bake (Option 1) is generally faster due to its improved parallelization and caching.

## Verifying the Build

After building, verify that all images were created successfully:

```bash
docker images
```

You should see a list of images with tags matching the Open5GS version specified in the Makefile or docker-bake.hcl file.

The output should include:

- base-open5gs
- amf, ausf, bsf, nrf, nssf, pcf, scp, sepp, smf, udm, udr, upf
- webui

## Network Configuration

The roaming setup creates a Docker bridge network named `open5gs` with subnet `10.33.33.0/24`. This is automatically configured when you start the containers.

## Docker Volumes

The following Docker volumes are created to persist data:

- `db_data`: MongoDB database data
- `db_config`: MongoDB configuration data
- `sepp_certs`: TLS certificates for SEPP components
- `sepp_ca`: Certificate Authority for SEPP components
- `captures`: Network packet captures from tshark

## Next Steps

After successfully building all the required Docker images, proceed to the [Configuration](configuration.html) page to configure the Open5GS roaming environment.
