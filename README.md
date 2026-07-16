# Secure Multi-Tier VPC Architecture on AWS

> Public & Private Subnets · Route Tables · Internet Gateway · NAT Gateway · NACLs · Security Groups

A reference design for a **secure, multi-tier network** on AWS. Public-facing
resources (load balancers / bastion) live in public subnets, while application
and database tiers live in private subnets with **no direct internet access**.
Outbound internet for private instances is provided through a NAT Gateway.

---

## Skills Demonstrated
- **VPC** design and CIDR planning
- **Public & Private Subnets** across multiple Availability Zones
- **Route Tables** (public route via IGW, private route via NAT)
- **Internet Gateway (IGW)** for public inbound/outbound
- **NAT Gateway** for private-subnet outbound only
- **Network ACLs (NACLs)** — stateless subnet-level firewall
- **Security Groups** — stateful instance-level firewall

---

## Architecture

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

> Full diagram details: [ARCHITECTURE.md](ARCHITECTURE.md)

---

## Security Layers
| Layer | Type | Scope | Behavior |
|-------|------|-------|----------|
| NACL  | Stateless | Subnet | Explicit allow/deny in & out |
| Security Group | Stateful | Instance/ENI | Return traffic auto-allowed |

**Best practice:** NACLs act as a coarse subnet guardrail; Security Groups do the
fine-grained, per-tier access control (e.g. App tier -> DB tier on 3306 only).

---

## Repository Structure
```
.
├── cloudformation/
│   └── vpc.yaml          # VPC, subnets, IGW, NAT, route tables, NACLs, SGs
├── docs/
│   └── network-plan.md   # CIDR allocation & routing reference
└── README.md
```

---

## Deploy
```bash
aws cloudformation create-stack \\
  --stack-name secure-vpc \\
  --template-body file://cloudformation/vpc.yaml
```

## Cleanup
```bash
aws cloudformation delete-stack --stack-name secure-vpc
```

---

## Author
**Dipak Kuthe** — DevOps / Cloud Engineer
