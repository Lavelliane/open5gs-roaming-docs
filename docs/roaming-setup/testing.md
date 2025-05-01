# Testing and Verification

This section guides you through testing the Open5GS roaming environment and verifying its functionality.

## Basic Operation Testing

### Verifying Network Function Registration

Check that all network functions are registered with their respective NRFs (Network Repository Functions):

```bash
# Home NRF
docker exec -it h-nrf curl -s http://nrf.5gc.mnc001.mcc001.3gppnetwork.org:7777/nnrf-nfm/v1/nf-instances | jq

# Visiting NRF
docker exec -it v-nrf curl -s http://nrf.5gc.mnc070.mcc999.3gppnetwork.org:7777/nnrf-nfm/v1/nf-instances | jq
```

You should see all the expected network functions registered in each NRF.

### Verifying SEPP Connectivity

The SEPP (Security Edge Protection Proxy) components are responsible for inter-PLMN security. Check their connectivity:

```bash
# Check H-SEPP logs
docker-compose logs h-sepp | grep "connection established"

# Check V-SEPP logs
docker-compose logs v-sepp | grep "connection established"
```

You should see messages indicating successful connection establishment between the SEPPs.

### Verifying UE Registration

Check the PacketRusher logs to verify UE registration:

```bash
docker-compose logs packetrusher | grep "Registration"
```

You should see messages indicating successful registration, such as "Registration accept received" and "Registration complete".

### Verifying PDU Session Establishment

Check the PacketRusher logs to verify PDU session establishment:

```bash
docker-compose logs packetrusher | grep "PDU Session"
```

You should see messages indicating successful PDU session establishment, such as "PDU Session Establishment Accept received".

## Data Connectivity Testing

### Testing Internet Connectivity

Test internet connectivity from the UE:

```bash
docker exec -it packetrusher /bin/sh
ping -I uesimtun0 8.8.8.8
```

A successful ping indicates that the UE has connectivity to the internet through the 5G core network.

### Testing DNS Resolution

Test DNS resolution from the UE:

```bash
docker exec -it packetrusher /bin/sh
nslookup -i uesimtun0 google.com
```

This should resolve the domain name to IP addresses, indicating that DNS is working correctly.

### Testing Bandwidth

Test the bandwidth of the connection:

```bash
docker exec -it packetrusher /bin/sh
wget -O /dev/null http://speedtest.ftp.otenet.gr/files/test100k.db
```

This will download a small test file and show the download speed.

## Roaming Functionality Testing

### Verifying SEPP TLS Security

Check the TLS certificate details for both SEPPs:

```bash
docker exec -it h-sepp ls -la /etc/open5gs/default/tls
docker exec -it v-sepp ls -la /etc/open5gs/default/tls
```

You should see the certificate files and private keys for each SEPP.

### Verifying Inter-PLMN Communication

Check the SEPP logs to see the inter-PLMN messages being exchanged:

```bash
docker-compose logs h-sepp | grep "N32-f"
docker-compose logs v-sepp | grep "N32-f"
```

You should see messages related to N32-f message forwarding between the PLMNs.

### Verifying UE Identity Protection

Check the AMF logs to see messages related to SUPI (Subscription Permanent Identifier) concealment:

```bash
docker-compose logs v-amf | grep "SUCI"
```

You should see messages indicating the AMF is handling SUCI (Subscription Concealed Identifier) for the UE.

## Troubleshooting Tests

### Checking Component Health

If you're experiencing issues, check the health of each container:

```bash
for container in $(docker-compose ps -q); do
  echo -n "Container: "
  docker inspect --format='{{.Name}}' $container
  echo -n "Status: "
  docker inspect --format='{{.State.Status}}' $container
  echo -n "Health: "
  docker inspect --format='{{if .State.Health}}{{.State.Health.Status}}{{else}}N/A{{end}}' $container
  echo "------------------------------"
done
```

This will display the status and health of each container.

### Checking Network Connectivity

Test network connectivity between components:

```bash
# Test connectivity from V-AMF to H-SEPP
docker exec -it v-amf ping -c 3 sepp.5gc.mnc001.mcc001.3gppnetwork.org

# Test connectivity from H-SEPP to V-SEPP
docker exec -it h-sepp ping -c 3 sepp.5gc.mnc070.mcc999.3gppnetwork.org
```

Successful pings indicate that the network connections are working correctly.

### Checking Database

Check the MongoDB database connection:

```bash
docker exec -it db mongo --eval "rs.status()"
```

This should show the database status, indicating that it's working correctly.

## Next Steps

After successfully testing the system, proceed to the [Packet Capture](/roaming-setup/packet-capture) page to learn about capturing and analyzing network traffic in the Open5GS roaming environment.
