# Running the System

This section guides you through starting and operating the Open5GS roaming environment.

## Starting the Containers

### Navigate to the Project Directory

First, ensure you're in the project directory:

```bash
cd open5gs-roaming
```

### Launch the Roaming Setup

To start the entire roaming setup using Docker Compose:

```bash
cd compose-files/roaming
docker-compose up -d
```

This command starts all the containers defined in the `docker-compose.yaml` file in detached mode.

### Verifying Container Status

Check that all containers are running properly:

```bash
docker-compose ps
```

You should see all containers in the "Up" state. If any container is in "Exit" or "Restarting" state, there may be configuration issues or dependency problems.

## Adding a Subscriber

For the UE to connect to the network, you need to add a subscriber to the database using the WebUI.

### Accessing the WebUI

The WebUI is accessible at `http://localhost:9999` in your browser. Log in with the default credentials:

- Username: `admin`
- Password: `1423`

### Adding a Subscriber

1. In the WebUI, go to "Subscriber" in the left menu
2. Click the "+" button to add a new subscriber
3. Enter the following information:

   - IMSI: `001010000000001` (must match the PacketRusher configuration)
   - Subscriber Key (K): `7F176C500D47CF2090CB6D91F4A73479`
   - OPc: `3D45770E83C7BBB6900F3653FDA6330F`
   - For "Slice Configuration":
     - SST: `1`
     - SD: `000001`
     - Session-AMBR Downlink: `1 Gbps`
     - Session-AMBR Uplink: `1 Gbps`
     - Default 5QI: `9`
   - For "Data Network Name (DNN)":
     - DNN: `internet`
     - IPv4: Selected
     - IPv6: Optional

4. Click "Save" to add the subscriber

## Monitoring Network Functions

### Viewing Container Logs

To view logs from all containers:

```bash
docker-compose logs
```

To view logs for a specific container (e.g., v-amf):

```bash
docker-compose logs v-amf
```

To continuously follow the logs:

```bash
docker-compose logs -f
```

### Checking Component Status

#### Verify SEPP TLS Setup

The SEPP components generate TLS certificates during initialization. To verify the certificates:

```bash
docker exec -it h-sepp ls -la /etc/open5gs/default/tls
docker exec -it v-sepp ls -la /etc/open5gs/default/tls
```

You should see the certificate files (`.crt` and `.key` files) in the output.

#### Check NRF Registrations

To check which network functions are registered with the NRFs:

```bash
# Home NRF
docker exec -it h-nrf curl -s http://nrf.5gc.mnc001.mcc001.3gppnetwork.org:7777/nnrf-nfm/v1/nf-instances | jq

# Visiting NRF
docker exec -it v-nrf curl -s http://nrf.5gc.mnc070.mcc999.3gppnetwork.org:7777/nnrf-nfm/v1/nf-instances | jq
```

## Running the UE Test

After adding the subscriber and ensuring all containers are running properly, you can test the UE connectivity.

### Starting the UE

The PacketRusher container automatically starts the UE simulation. You can verify it's working by checking the logs:

```bash
docker-compose logs packetrusher
```

Look for messages indicating successful registration and PDU session establishment.

### Verifying Data Connectivity

To verify that the UE has data connectivity:

```bash
docker exec -it packetrusher /bin/sh
ping -I uesimtun0 8.8.8.8
```

If the ping is successful, the UE has correctly connected to the 5G network and has internet access.

## Stopping the System

To stop all containers while preserving their data:

```bash
docker-compose down
```

To completely remove all containers, networks, and volumes:

```bash
docker-compose down -v
```

## Next Steps

After successfully running the system, proceed to the [Testing and Verification](testing.html) page to learn about testing procedures and verifying the roaming functionality.
