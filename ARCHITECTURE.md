# Architecture — Secure Multi-Tier VPC

Secure network foundation with public and private subnets across two Availability Zones.

```mermaid
flowchart TB
    IGW[Internet Gateway] --- PUBRT[Public Route Table - 0.0.0.0/0 to IGW]
    subgraph VPC[VPC 10.0.0.0/16]
      PA[Public Subnet A 10.0.1.0/24]
      PB[Public Subnet B 10.0.2.0/24]
      NAT[NAT Gateway plus EIP]
      RA[Private Subnet A 10.0.11.0/24 - App / DB]
      RB[Private Subnet B 10.0.12.0/24 - App / DB]
    end
    PUBRT --- PA
    PUBRT --- PB
    NAT --> IGW
    PRIVRT[Private Route Table - 0.0.0.0/0 to NAT] --- RA
    PRIVRT --- RB
    RA -->|outbound only| NAT
    RB -->|outbound only| NAT
```

## How it works

- Public subnets host internet-facing resources and route to the Internet Gateway.
- Private subnets host the application and database tiers with no direct inbound internet access.
- The NAT Gateway allows private instances outbound internet access only (for updates).
- Network ACLs provide stateless subnet-level filtering and Security Groups provide stateful instance-level filtering.
