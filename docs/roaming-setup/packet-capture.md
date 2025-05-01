# Packet Capture

This section explains how to capture and analyze network traffic in the Open5GS roaming environment.

## Built-in Packet Capture

The roaming setup includes a built-in packet capture functionality through the `tshark` container. This container continuously captures packets on the `br-ogs` bridge network interface and saves them to PCAP files.

### Capture Files

The packet captures are stored in the `captures` directory within the Docker volume. They are organized as follows:

- `open5gs_all.pcap`: All traffic on the `br-ogs` interface
- `sepp_sbi.pcap`: SBI (Service-Based Interface) traffic for the SEPP components
- `n32_interfaces.pcap`: Traffic on the N32 interfaces
- `n1_n2_interfaces.pcap`: Traffic on the N1 and N2 interfaces (SCTP)
- `n3_interface.pcap`: Traffic on the N3 interface (UDP port 2152, GTP-U)

### Accessing Capture Files

To access the capture files:

```bash
# Create a directory to store the captured files
mkdir -p captures

# Copy the captures from the Docker volume to the local filesystem
docker cp tshark:/captures/ ./

# List the capture files
ls -la captures
```

## Analyzing Captures with Wireshark

You can analyze the capture files using Wireshark. If you're using a remote server, you may need to transfer the files to your local machine first.

### Transferring Files to Your Local Machine

If you're using a remote server, use `scp` to transfer the files:

```bash
# From your local machine
scp -r username@remote-server:/path/to/open5gs-roaming/captures ./
```

### Opening Captures in Wireshark

Open Wireshark on your local machine and open the desired capture file.

#### SCTP (N1/N2) Traffic Analysis

For analyzing the SCTP traffic (N1/N2 interfaces):

1. Open `n1_n2_interfaces.pcap` in Wireshark
2. Apply a filter for NGAP messages: `ngap`
3. Look for messages such as:
   - Registration Request/Accept
   - PDU Session Establishment Request/Accept
   - UE Context Setup Request/Response

#### GTP-U (N3) Traffic Analysis

For analyzing the GTP-U traffic (N3 interface):

1. Open `n3_interface.pcap` in Wireshark
2. Apply a filter for GTP messages: `gtp`
3. Look for GTP-U packets carrying user data

#### SEPP SBI Traffic Analysis

For analyzing the SEPP SBI traffic:

1. Open `sepp_sbi.pcap` in Wireshark
2. Apply a filter for HTTP/2 messages: `http2`
3. Look for HTTP/2 messages between NRF and SEPP components

#### N32 Interface Traffic Analysis

For analyzing the N32 interface traffic:

1. Open `n32_interfaces.pcap` in Wireshark
2. Apply a filter for HTTP/2 messages: `http2`
3. Look for HTTP/2 messages between SEPPs

## Capturing Live Traffic

You can also capture live traffic directly using `tshark` within the container:

```bash
# Capture all traffic for 30 seconds
docker exec -it tshark tshark -i br-ogs -a duration:30

# Capture traffic with a specific filter
docker exec -it tshark tshark -i br-ogs -f "sctp" -a duration:30

# Capture traffic to a new file
docker exec -it tshark tshark -i br-ogs -w /captures/custom_capture.pcap -a duration:30
```

## Analyzing TLS-encrypted Traffic

The SEPP components use TLS to secure their communication. To analyze the encrypted traffic, you need the TLS keys.

### Extracting TLS Keys

The TLS keys are stored in the SEPP containers. To extract them:

```bash
# Extract H-SEPP certificates
docker cp h-sepp:/etc/open5gs/default/tls/ ./h-sepp-tls/

# Extract V-SEPP certificates
docker cp v-sepp:/etc/open5gs/default/tls/ ./v-sepp-tls/
```

### Configuring Wireshark for TLS Decryption

To configure Wireshark to decrypt the TLS traffic:

1. Open Wireshark
2. Go to Edit > Preferences > Protocols > TLS
3. Add the RSA keys by clicking on the "RSA Keys List" button
4. For each key, add:
   - IP address: SEPP IP address
   - Port: SEPP port (7777, 7778, or 7779)
   - Protocol: http
   - Key File: Path to the private key file
5. Click OK to save the settings

## Key Interfaces to Analyze

When analyzing the packet captures, focus on these key interfaces:

1. **N1 Interface**: Between UE and AMF

   - Look for NAS (Non-Access Stratum) messages
   - Registration, Authentication, Security Mode procedures

2. **N2 Interface**: Between gNB and AMF

   - Look for NGAP (NG Application Protocol) messages
   - UE Context Setup, PDU Session Resource Setup

3. **N3 Interface**: Between gNB and UPF

   - Look for GTP-U (GPRS Tunneling Protocol User Plane) messages
   - User data tunneling

4. **N32 Interface**: Between SEPPs
   - Look for SEPP message exchange (N32-c, N32-f)
   - Security parameters exchange, JOSE-protected messages

### Sample Wireshark Data Screenshot

To better understand the packet flow and the types of messages exchanged in the Open5GS roaming setup, you can capture and analyze the network traffic using Wireshark. Below is a sample screenshot of Wireshark displaying captured data:

![Sample Wireshark Data](/images/packet_capture.png)

In this screenshot, you can observe various protocol messages such as NAS, NGAP, and GTP-U. These messages are crucial for understanding the communication between different network components. Pay attention to the message types and their sequence to gain insights into the network operations.

## Next Steps

After learning about packet capture, proceed to the [Troubleshooting](/roaming-setup/troubleshooting) page for tips on resolving common issues in the Open5GS roaming environment.
