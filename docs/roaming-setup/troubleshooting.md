# Troubleshooting

This section provides solutions for common issues you might encounter with the Open5GS roaming setup.

## Container Issues

### Container Fails to Start

**Issue**: One or more containers fail to start or keep restarting.

**Solution**:

1. Check the container logs:

```bash
docker-compose logs <container_name>
```

2. Check dependencies:

```bash
docker-compose ps
```

Ensure that all the services on which the failing container depends are running correctly.

3. Check resource availability:

```bash
docker stats
```

Ensure that your system has enough CPU, memory, and disk space.

### MongoDB Connection Issues

**Issue**: Containers fail to connect to the MongoDB database.

**Solution**:

1. Check if the MongoDB container is running:

```bash
docker-compose ps db
```

2. Check MongoDB logs:

```bash
docker-compose logs db
```

3. Try to connect to MongoDB manually:

```bash
docker exec -it db mongo
```

4. Ensure the database is properly initialized:

```bash
docker exec -it db mongo --eval "db.stats()"
```

## Network Issues

### Network Function Registration Failures

**Issue**: Network functions fail to register with NRF.

**Solution**:

1. Check NRF logs:

```bash
docker-compose logs h-nrf
docker-compose logs v-nrf
```

2. Verify network connectivity:

```bash
docker exec -it v-amf ping -c 3 nrf.5gc.mnc070.mcc999.3gppnetwork.org
```

3. Check if NRF is accessible via its API:

```bash
docker exec -it v-amf curl -s http://nrf.5gc.mnc070.mcc999.3gppnetwork.org:7777/nnrf-nfm/v1/nf-instances
```

### SEPP TLS Certificate Issues

**Issue**: SEPP components fail to establish secure connections due to certificate issues.

**Solution**:

1. Check SEPP logs for certificate errors:

```bash
docker-compose logs h-sepp | grep "certificate"
docker-compose logs v-sepp | grep "certificate"
```

2. Regenerate the certificates:

```bash
docker-compose down
docker volume rm open5gs_sepp_certs open5gs_sepp_ca
docker-compose up -d
```

3. Verify certificates were created:

```bash
docker exec -it h-sepp ls -la /etc/open5gs/default/tls
docker exec -it v-sepp ls -la /etc/open5gs/default/tls
```

### IP Routing Issues

**Issue**: Components cannot communicate with each other.

**Solution**:

1. Check network configuration:

```bash
docker network inspect open5gs
```

2. Verify DNS resolution:

```bash
docker exec -it v-amf nslookup nrf.5gc.mnc070.mcc999.3gppnetwork.org
docker exec -it v-amf nslookup sepp.5gc.mnc001.mcc001.3gppnetwork.org
```

3. Check routing table:

```bash
docker exec -it v-amf ip route
```

## UE Connection Issues

### UE Registration Failure

**Issue**: The UE fails to register with the network.

**Solution**:

1. Check PacketRusher logs:

```bash
docker-compose logs packetrusher
```

2. Verify AMF configuration:

```bash
docker-compose logs v-amf
```

3. Ensure the subscriber exists in the database:

```bash
docker exec -it db mongo --eval "db = db.getSiblingDB('open5gs'); db.subscribers.find().pretty()"
```

4. Verify the subscriber information matches the UE configuration:
   - Check IMSI
   - Check authentication keys (K, OPC)
   - Check allowed slices (SST, SD)
   - Check DNN configuration

### PDU Session Establishment Failure

**Issue**: The UE fails to establish a PDU session.

**Solution**:

1. Check SMF logs:

```bash
docker-compose logs v-smf
```

2. Check UPF logs:

```bash
docker-compose logs v-upf
```

3. Verify GTP kernel module is loaded:

```bash
docker exec -it v-upf lsmod | grep gtp5g
```

4. Check UE DNN configuration matches the subscriber profile.

## WebUI Issues

### Cannot Access WebUI

**Issue**: Cannot access the WebUI at http://localhost:9999.

**Solution**:

1. Check if the WebUI container is running:

```bash
docker-compose ps webui
```

2. Check WebUI logs:

```bash
docker-compose logs webui
```

3. Ensure port 9999 is correctly mapped:

```bash
docker-compose port webui 9999
```

4. If using a remote server, ensure you have port forwarding set up correctly.

### Cannot Add Subscribers

**Issue**: Unable to add subscribers through the WebUI.

**Solution**:

1. Check WebUI logs:

```bash
docker-compose logs webui
```

2. Check MongoDB connection:

```bash
docker exec -it webui curl -s http://db.open5gs.org:27017
```

3. Clear browser cache or try a different browser.

## Packet Capture Issues

### No Traffic Captured

**Issue**: No traffic appears in the packet capture files.

**Solution**:

1. Check tshark container:

```bash
docker-compose ps tshark
```

2. Check tshark logs:

```bash
docker-compose logs tshark
```

3. Ensure the capture directory exists:

```bash
docker exec -it tshark ls -la /captures
```

4. Try a manual capture:

```bash
docker exec -it tshark tshark -i br-ogs -a duration:30 -w /captures/manual_capture.pcap
```

### Cannot Open Capture Files

**Issue**: Unable to open or analyze capture files.

**Solution**:

1. Check file format:

```bash
docker exec -it tshark file /captures/open5gs_all.pcap
```

2. Try converting the capture file format:

```bash
docker exec -it tshark editcap -F libpcap /captures/open5gs_all.pcap /captures/open5gs_all_converted.pcap
```

3. Ensure you're using a recent version of Wireshark.

## Roaming-specific Issues

### Home-Visited Communication Failure

**Issue**: The Home network cannot communicate with the Visiting network.

**Solution**:

1. Check SEPP logs:

```bash
docker-compose logs h-sepp
docker-compose logs v-sepp
```

2. Verify SBI discovery in NRF:

```bash
docker exec -it h-nrf curl -s http://nrf.5gc.mnc001.mcc001.3gppnetwork.org:7777/nnrf-disc/v1/nf-instances
docker exec -it v-nrf curl -s http://nrf.5gc.mnc070.mcc999.3gppnetwork.org:7777/nnrf-disc/v1/nf-instances
```

3. Check N32 interface connectivity:

```bash
docker exec -it h-sepp curl -k https://sepp.5gc.mnc070.mcc999.3gppnetwork.org:8778
```

### Authentication Failure in Roaming

**Issue**: UE authentication fails in roaming scenario.

**Solution**:

1. Check UDM logs:

```bash
docker-compose logs h-udm
```

2. Check AUSF logs:

```bash
docker-compose logs h-ausf
docker-compose logs v-ausf
```

3. Verify the subscriber data in the UDR:

```bash
docker-compose logs h-udr
```

4. Ensure the correct authentication keys are used in both PacketRusher and the subscriber database.

## Common Commands for Troubleshooting

Here's a summary of useful commands for troubleshooting:

### Container Management

```bash
# View all containers
docker-compose ps

# Restart a specific container
docker-compose restart <container_name>

# Restart all containers
docker-compose down && docker-compose up -d

# View container logs
docker-compose logs <container_name>

# Follow container logs
docker-compose logs -f <container_name>
```

### Container Inspection

```bash
# Get a shell in a container
docker exec -it <container_name> /bin/bash

# Check container resource usage
docker stats

# Inspect container details
docker inspect <container_name>
```

### Network Inspection

```bash
# Inspect network
docker network inspect open5gs

# Check connectivity from a container
docker exec -it <container_name> ping -c 3 <target>

# Check open ports
docker exec -it <container_name> netstat -tuln
```

### Database Management

```bash
# Connect to MongoDB
docker exec -it db mongo

# Check subscribers
docker exec -it db mongo --eval "db = db.getSiblingDB('open5gs'); db.subscribers.find().pretty()"

# Add a test subscriber
# (This is a simplified example; use the WebUI for complete subscriber management)
docker exec -it db mongo --eval "db = db.getSiblingDB('open5gs'); db.subscribers.insert({imsi: '001010000000001', ...})"
```

## Next Steps

After troubleshooting any issues, you should have a fully functioning Open5GS roaming environment. For advanced topics and further exploration, consult the following resources:

- [Open5GS Official Documentation](https://open5gs.org/open5gs/docs/)
- [5G Core Network Specifications](https://www.3gpp.org/technologies/5g-system)
- [3GPP Roaming Specifications](https://www.gsma.com/newsroom/wp-content/uploads//NG.113-v5.0.pdf)
