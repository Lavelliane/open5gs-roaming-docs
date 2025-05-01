# Configuration

This section explains how to configure the Open5GS roaming environment.

## Configuration Files

The configuration files for all components are located in the `configs/roaming` directory. These files define the behavior of each network function in the system.

### Home Network Configuration Files

- `h-nrf.yaml`: Configuration for the Home Network Repository Function
- `h-ausf.yaml`: Configuration for the Home Authentication Server Function
- `h-udm.yaml`: Configuration for the Home Unified Data Management
- `h-udr.yaml`: Configuration for the Home Unified Data Repository
- `h-sepp.yaml`: Configuration for the Home Security Edge Protection Proxy

### Visiting Network Configuration Files

- `v-nrf.yaml`: Configuration for the Visiting Network Repository Function
- `v-ausf.yaml`: Configuration for the Visiting Authentication Server Function
- `v-nssf.yaml`: Configuration for the Visiting Network Slice Selection Function
- `v-bsf.yaml`: Configuration for the Visiting Binding Support Function
- `v-pcf.yaml`: Configuration for the Visiting Policy Control Function
- `v-amf.yaml`: Configuration for the Visiting Access and Mobility Management Function
- `v-smf.yaml`: Configuration for the Visiting Session Management Function
- `v-upf.yaml`: Configuration for the Visiting User Plane Function
- `v-sepp.yaml`: Configuration for the Visiting Security Edge Protection Proxy

### PacketRusher Configuration

- `packetrusher.yaml`: Configuration for the PacketRusher gNB and UE simulator

## Understanding the Configuration

### PLMN Configuration

The Home Network (H-PLMN) is configured with:

- MCC: 001
- MNC: 01

The Visiting Network (V-PLMN) is configured with:

- MCC: 999
- MNC: 70

### Network Slicing Configuration

The setup includes a network slice with:

- SST (Slice/Service Type): 01
- SD (Slice Differentiator): 000001

### SEPP Configuration and TLS Setup

The Security Edge Protection Proxy (SEPP) components secure inter-PLMN communication using TLS. Both Home SEPP (h-sepp) and Visiting SEPP (v-sepp) use similar configuration structures with some key differences.

#### TLS Certificate Generation and Management

Both SEPPs automatically generate TLS certificates during initialization using the `entrypoint.sh` script. This process includes:

1. Generating a CA certificate and private key
2. Creating a SEPP certificate signed by the CA
3. Building a certificate chain
4. Exchanging CA certificates between SEPPs
5. Creating a CA bundle for verification

The certificates are stored in the following locations:

- Private keys: `/etc/open5gs/default/tls/sepp1.key` (h-sepp) or `/etc/open5gs/default/tls/sepp2.key` (v-sepp)
- Certificate chains: `/etc/open5gs/default/tls/sepp1.chain.crt` (h-sepp) or `/etc/open5gs/default/tls/sepp2.chain.crt` (v-sepp)
- CA bundle: `/etc/open5gs/default/tls/ca-bundle.crt` (contains both SEPPs' CA certificates)

#### Home SEPP Configuration (h-sepp.yaml)

```yaml
sepp:
  default:
    tls:
      server:
        private_key: /etc/open5gs/default/tls/sepp1.key
        cert: /etc/open5gs/default/tls/sepp1.chain.crt
      client:
        cacert: /etc/open5gs/default/tls/ca-bundle.crt
        verify: true
  sbi:
    server:
      - address: sepp.5gc.mnc001.mcc001.3gppnetwork.org
        port: 80
    client:
      nrf:
        - uri: http://nrf.5gc.mnc001.mcc001.3gppnetwork.org:80
  n32:
    server:
      - sender: sepp1.localdomain
        scheme: https
        address: 0.0.0.0
        port: 7778
        n32f:
          scheme: https
          address: 0.0.0.0
          port: 7779
    client:
      sepp:
        - receiver: sepp2.localdomain
          uri: https://sepp2.localdomain:7778
          verify_client: true
          n32f:
            uri: https://sepp2.localdomain:7779
            verify_client: true
```

#### Visiting SEPP Configuration (v-sepp.yaml)

```yaml
sepp:
  default:
    tls:
      server:
        private_key: /etc/open5gs/default/tls/sepp2.key
        cert: /etc/open5gs/default/tls/sepp2.chain.crt
      client:
        cacert: /etc/open5gs/default/tls/ca-bundle.crt
        verify: true
  sbi:
    server:
      - address: sepp.5gc.mnc070.mcc999.3gppnetwork.org
        port: 80
    client:
      nrf:
        - uri: http://nrf.5gc.mnc070.mcc999.3gppnetwork.org:80
  n32:
    server:
      - sender: sepp2.localdomain
        scheme: https
        address: 0.0.0.0
        port: 7778
        n32f:
          scheme: https
          address: 0.0.0.0
          port: 7779
    client:
      sepp:
        - receiver: sepp1.localdomain
          uri: https://sepp1.localdomain:7778
          verify_client: true
          n32f:
            uri: https://sepp1.localdomain:7779
            verify_client: true
```

#### Key Points in SEPP Configuration

1. **TLS Certificates**:

   - Both SEPPs use their own private keys and certificate chains
   - Both verify peer certificates using a shared CA bundle

2. **SBI (Service-Based Interface)**:

   - h-sepp connects to the Home NRF at `nrf.5gc.mnc001.mcc001.3gppnetwork.org:80`
   - v-sepp connects to the Visiting NRF at `nrf.5gc.mnc070.mcc999.3gppnetwork.org:80`

3. **N32 Interfaces**:

   - **N32-c** (control plane): Both SEPPs listen on port 7778 for capability negotiation and security parameter exchange
   - **N32-f** (forwarding plane): Both SEPPs listen on port 7779 for actual message exchange between PLMNs
   - h-sepp connects to v-sepp at `sepp2.localdomain` and vice versa
   - Client verification is enabled for secure TLS communication

4. **Domain Names**:

   - h-sepp uses `sepp1.localdomain` as its sender identity
   - v-sepp uses `sepp2.localdomain` as its sender identity

5. **Port Mapping**:
   - Although both SEPPs use the same internal ports (7778 for N32-c, 7779 for N32-f), when accessing v-sepp from outside, ports 8778 and 8779 are used due to Docker port mapping

### UE and gNB Configuration

The PacketRusher configuration defines the User Equipment and gNB parameters:

```yaml
gnodeb:
  controlif:
    ip: 'gnb.packetrusher.org'
    port: 38412
  dataif:
    ip: 'gnb.packetrusher.org'
    port: 2152
  plmnlist:
    mcc: '999'
    mnc: '70'
    tac: '000001'
    gnbid: '000008'
  slicesupportlist:
    sst: '01'
    sd: '000001'

ue:
  hplmn:
    mcc: '001'
    mnc: '01'
  msin: '1234567891'
  key: '7F176C500D47CF2090CB6D91F4A73479'
  opc: '3D45770E83C7BBB6900F3653FDA6330F'
  dnn: 'internet'
  snssai:
    sst: 01
    sd: '000001'
  amf: '8000'
  sqn: '00000000'
  protectionScheme: 0
  integrity:
    nia0: false
    nia1: false
    nia2: true
    nia3: false
  ciphering:
    nea0: true
    nea1: false
    nea2: true
    nea3: false

amfif:
  - ip: 'amf.5gc.mnc070.mcc999.3gppnetwork.org'
    port: 38412
logs:
  level: 4
```

## Docker Compose Configuration

The Docker Compose configuration is in the `compose-files/roaming/docker-compose.yaml` file. This file defines:

- All the containers and their dependencies
- Network configuration
- Volume mounts
- Environment variables
- Port mappings
- Service-specific configurations

### SEPP Environment Variables

The SEPP containers have specific environment variables that control their behavior:

#### Home SEPP (h-sepp)

```yaml
environment:
  - SEPP_TYPE=sepp1
  - REGENERATE_CERTS=true
  - TLS_DIR=/etc/open5gs/default/tls
  - SEPP_FQDN=sepp1.localdomain
  - SEPP_PLMN_FQDN=sepp.5gc.mnc001.mcc001.3gppnetwork.org
```

#### Visiting SEPP (v-sepp)

```yaml
environment:
  - SEPP_TYPE=sepp2
  - REGENERATE_CERTS=true
  - TLS_DIR=/etc/open5gs/default/tls
  - SEPP_FQDN=sepp2.localdomain
  - SEPP_PLMN_FQDN=sepp.5gc.mnc070.mcc999.3gppnetwork.org
```

### SEPP Port Mapping

The SEPPs expose their services on different port mappings:

#### Home SEPP (h-sepp)

```yaml
ports:
  - '0.0.0.0:7777:7777/tcp' # SBI Server
  - '0.0.0.0:7778:7778/tcp' # N32c Server
  - '0.0.0.0:7779:7779/tcp' # N32f Server
```

#### Visiting SEPP (v-sepp)

```yaml
ports:
  - '0.0.0.0:8777:7777/tcp' # SBI Server
  - '0.0.0.0:8778:7778/tcp' # N32c Server
  - '0.0.0.0:8779:7779/tcp' # N32f Server
```

### Key Environment Variables

The Docker images use several environment variables defined at build time:

- `OPEN5GS_VERSION`: Version of Open5GS
- `UBUNTU_VERSION`: Ubuntu version for base images
- `NODE_VERSION`: Node.js version for WebUI
- `MONGODB_VERSION`: MongoDB version

## Next Steps

After understanding the configuration, proceed to the [Running the System](running.html) page to start the Open5GS roaming environment.
