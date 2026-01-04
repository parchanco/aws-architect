# 27 - VPC (Virtual Private Cloud)

## VPC Basics

Red privada virtual en AWS.

### Componentes

```
VPC: Network container (10.0.0.0/16)
├─ Subnets: Segments (10.0.1.0/24, 10.0.2.0/24)
├─ Route Tables: Traffic routing
├─ Internet Gateway: Internet access
├─ NAT Gateway: Private → Internet
└─ Security Groups / NACLs: Firewalls
```

### CIDR Blocks

```
IP Range notation:
10.0.0.0/16
│       │
│       └─ Bits for network (16)
└───────── IP address

10.0.0.0/16 = 10.0.0.0 - 10.0.255.255 (65,536 IPs)
10.0.0.0/24 = 10.0.0.0 - 10.0.0.255 (256 IPs)

AWS reserves 5 IPs per subnet:
10.0.0.0: Network
10.0.0.1: VPC router
10.0.0.2: DNS
10.0.0.3: Future
10.0.0.255: Broadcast (not used but reserved)

Usable: 256 - 5 = 251 IPs
```

### Default VPC

```
Cada región tiene default VPC:
- CIDR: 172.31.0.0/16
- Subnets: 1 per AZ (public)
- Internet Gateway: ✓
- DNS: Enabled

→ EC2 gets public IP by default
```

## Subnets

Segment de VPC.

### Types

**Public Subnet**:
```
Route table: 0.0.0.0/0 → Internet Gateway
Resources get public IPs
Use: Load balancers, bastions
```

**Private Subnet**:
```
No route to Internet Gateway
No public IPs
Use: App servers, databases
```

### Example Layout

```
VPC: 10.0.0.0/16

Public Subnets:
├─ 10.0.1.0/24 (AZ-a) → ALB
├─ 10.0.2.0/24 (AZ-b) → ALB
└─ 10.0.3.0/24 (AZ-c) → ALB

Private App Subnets:
├─ 10.0.11.0/24 (AZ-a) → EC2 Odoo
├─ 10.0.12.0/24 (AZ-b) → EC2 Odoo
└─ 10.0.13.0/24 (AZ-c) → EC2 Odoo

Private DB Subnets:
├─ 10.0.21.0/24 (AZ-a) → RDS
├─ 10.0.22.0/24 (AZ-b) → RDS
└─ 10.0.23.0/24 (AZ-c) → RDS (standby)
```

## Route Tables

Determina routing de tráfico.

### Main Route Table

```
Default para subnets sin explicit association

Target             Destination
local              10.0.0.0/16
```

### Custom Route Tables

**Public Route Table**:
```
Target             Destination
local              10.0.0.0/16
igw-xxx            0.0.0.0/0

Associated: Public subnets
```

**Private Route Table**:
```
Target             Destination
local              10.0.0.0/16
nat-xxx            0.0.0.0/0

Associated: Private subnets
```

## Internet Gateway (IGW)

Public Internet access.

```
Characteristics:
- 1 per VPC
- Horizontally scaled
- Redundant
- No bandwidth constraints

Attach to VPC:
aws ec2 attach-internet-gateway --vpc-id vpc-xxx --internet-gateway-id igw-xxx
```

## NAT Gateway

Private subnets → Internet (outbound only).

### NAT Gateway

```
Managed by AWS
Highly available (within AZ)
Bandwidth: 5-45 Gbps
Cost: $0.045/hour + $0.045/GB

⚠️ Single AZ
→ Deploy 1 per AZ for HA
```

**Setup**:
```bash
# Create NAT Gateway en public subnet
aws ec2 create-nat-gateway   --subnet-id subnet-public-a   --allocation-id eipalloc-xxx

# Update private route table
Destination: 0.0.0.0/0
Target: nat-gateway-xxx
```

**High Availability**:
```
Public Subnet AZ-a → NAT-a
Public Subnet AZ-b → NAT-b

Private Subnet AZ-a → Route → NAT-a
Private Subnet AZ-b → Route → NAT-b

If NAT-a fails:
→ AZ-a instances lose internet
→ AZ-b instances unaffected
```

### NAT Instance

```
EC2 instance as NAT
DIY, manual setup
Cheaper pero less reliable
Debe disable source/destination check
```

**vs NAT Gateway**:
```
NAT Gateway:
✅ Managed, HA
✅ Higher bandwidth
❌ More expensive

NAT Instance:
✅ Cheaper
✅ Use as bastion
❌ Manual management
❌ Single point of failure
```

## Security Groups

Stateful firewall para resources.

### Características

```
Stateful: Return traffic auto-allowed
Level: Instance (ENI)
Allow rules only
Default: Deny all inbound, allow all outbound
Can reference other SGs
```

### Example

```
SG-ALB:
Inbound:
- Type: HTTP, Port: 80, Source: 0.0.0.0/0
- Type: HTTPS, Port: 443, Source: 0.0.0.0/0

SG-Odoo:
Inbound:
- Type: Custom TCP, Port: 8069, Source: SG-ALB
- Type: Custom TCP, Port: 8072, Source: SG-ALB
- Type: SSH, Port: 22, Source: 203.0.113.0/24 (admin IP)

SG-RDS:
Inbound:
- Type: PostgreSQL, Port: 5432, Source: SG-Odoo
```

## Network ACLs (NACLs)

Stateless firewall para subnets.

### Características

```
Stateless: Must allow both inbound + outbound
Level: Subnet
Allow + Deny rules
Numbered rules (evaluated in order)
Default: Allow all
```

### Example

```
Rule # Type     Protocol Port  Source         Allow/Deny
100    HTTP     TCP      80    0.0.0.0/0      ALLOW
110    HTTPS    TCP      443   0.0.0.0/0      ALLOW
120    SSH      TCP      22    203.0.113.0/24 ALLOW
*      All      All      All   0.0.0.0/0      DENY

Ephemeral ports:
200    Custom   TCP      1024- 0.0.0.0/0      ALLOW
                         65535
```

### NACL vs Security Group

```
NACL:
- Subnet level
- Stateless
- Allow + Deny
- Numbered rules (order matters)
- First match wins

Security Group:
- Instance level
- Stateful
- Allow only
- All rules evaluated
```

## VPC Peering

Connect 2 VPCs.

```
VPC-A (10.0.0.0/16) ⟷ VPC-B (10.1.0.0/16)

Requirements:
- No overlapping CIDRs
- Not transitive (A-B, B-C ≠ A-C)
- Can be cross-region, cross-account
```

**Setup**:
```bash
# Create peering connection
aws ec2 create-vpc-peering-connection   --vpc-id vpc-a   --peer-vpc-id vpc-b

# Accept
aws ec2 accept-vpc-peering-connection   --vpc-peering-connection-id pcx-xxx

# Update route tables
VPC-A: 10.1.0.0/16 → pcx-xxx
VPC-B: 10.0.0.0/16 → pcx-xxx
```

## VPC Endpoints

Access AWS services sin Internet Gateway.

### Interface Endpoint (PrivateLink)

```
ENI en subnet
Private IP
Powered by PrivateLink
Cost: $0.01/hour + $0.01/GB

Supports:
- Most AWS services
- 3rd party services
```

### Gateway Endpoint

```
Target en route table
Free
Only: S3, DynamoDB

Route:
pl-xxx (S3) → vpce-xxx
```

**Example S3 Gateway Endpoint**:
```bash
# Create
aws ec2 create-vpc-endpoint   --vpc-id vpc-xxx   --service-name com.amazonaws.eu-west-1.s3   --route-table-ids rtb-xxx

# Route table auto-updated:
Destination        Target
pl-xxx (S3)        vpce-xxx
```

## VPC Flow Logs

Capture IP traffic.

### Levels

```
VPC level: All ENIs in VPC
Subnet level: All ENIs in subnet
ENI level: Specific ENI
```

### Destinations

```
CloudWatch Logs
S3
Kinesis Data Firehose
```

### Log Format

```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status

Example:
2 123456789012 eni-abc123de 10.0.1.5 10.0.2.10 45678 8069 6 10 5000 1672531200 1672531260 ACCEPT OK
```

### Use Cases

```
✓ Troubleshoot connectivity
✓ Security analysis
✓ Monitor traffic
✓ Detect anomalies
✓ Compliance
```

## Transit Gateway

Hub para conectar VPCs.

```
            Transit Gateway
                  │
    ┌─────────────┼─────────────┐
    │             │             │
  VPC-A         VPC-B         VPC-C
  VPC-D         VPC-E    On-premise (VPN)
```

**vs VPC Peering**:
```
VPC Peering: N×(N-1)/2 connections
Transit Gateway: N connections

10 VPCs:
Peering: 45 connections
TGW: 10 connections
```

**Features**:
```
Hub-and-spoke topology
Cross-region peering
VPN connections
Direct Connect
Multicast support
Route tables
```

## Site-to-Site VPN

On-premise → VPC connection.

### Components

```
On-premise:
- Customer Gateway (physical device)

AWS:
- Virtual Private Gateway (VGW)
- VPN Connection (2 tunnels)
```

### Setup

```bash
# Create Customer Gateway
aws ec2 create-customer-gateway   --type ipsec.1   --public-ip 203.0.113.1   --bgp-asn 65000

# Create VPN Gateway
aws ec2 create-vpn-gateway --type ipsec.1

# Attach to VPC
aws ec2 attach-vpn-gateway --vpc-id vpc-xxx --vpn-gateway-id vgw-xxx

# Create VPN Connection
aws ec2 create-vpn-connection   --type ipsec.1   --customer-gateway-id cgw-xxx   --vpn-gateway-id vgw-xxx
```

### CloudHub

Multiple sites → VPC.

```
Site-A ─┐
Site-B ─┼→ VPN → VGW → VPC
Site-C ─┘

Also: Site-A ↔ Site-B (via VGW)
```

## Direct Connect

Dedicated network connection.

```
On-premise → Direct Connect Location → AWS

Benefits:
✓ More bandwidth (1 Gbps, 10 Gbps, 100 Gbps)
✓ Lower costs (vs internet)
✓ Consistent network performance
✓ Private connectivity
```

### Connection Types

**Dedicated**:
```
1, 10, 100 Gbps
Physical fiber
Directly to you
```

**Hosted**:
```
50 Mbps - 10 Gbps
Via partner
Flexible capacity
```

### Virtual Interfaces (VIF)

**Private VIF**:
```
Access VPC private resources
```

**Public VIF**:
```
Access AWS public services (S3, DynamoDB)
```

### Direct Connect Gateway

```
Multiple VPCs in different regions via single DX

On-premise → DX → DX Gateway
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    VPC eu-west-1  VPC us-east-1  VPC ap-southeast-1
```

## VPC Best Practices - Odoo

```
Architecture:

VPC: 10.0.0.0/16

Public Subnets (3 AZs):
10.0.{1,2,3}.0/24
└─ ALB, NAT Gateways

Private App Subnets (3 AZs):
10.0.{11,12,13}.0/24
└─ Odoo EC2 instances

Private DB Subnets (3 AZs):
10.0.{21,22,23}.0/24
└─ RDS Multi-AZ

Security:
✅ Private subnets para app y DB
✅ Public subnet solo para ALB
✅ NAT Gateway para outbound (updates, APIs)
✅ VPC Endpoints para S3 (reduce costs)
✅ Security Groups restrictivos
✅ VPC Flow Logs enabled
✅ NACLs como defensa adicional

Connectivity:
✅ Site-to-Site VPN para office access
✅ Direct Connect si high bandwidth needed
✅ Transit Gateway si múltiples VPCs
```

---

**Certificación**: VPC es uno de los topics más importantes. Domina subnets, routing, NAT, SG, NACLs.

**Siguiente**: [28 - Disaster Recovery](28-disaster-recovery.md)
