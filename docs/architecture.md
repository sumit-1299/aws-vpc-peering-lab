# AWS VPC Peering Architecture

## 1. Purpose

This document describes the logical network design used in the AWS VPC Peering Lab. The lab connects private workloads in two VPCs using a VPC Peering Connection.

## 2. Network Topology

### VPC A — `10.10.0.0/16`

| Component | Address | Role |
|---|---|---|
| Public subnet | `10.10.1.0/24` | Hosts the bastion |
| Private subnet | `10.10.2.0/24` | Hosts App-A |
| Bastion | `10.10.1.10` | Controlled SSH entry point |
| App-A | `10.10.2.10` | Private application host |
| Internet Gateway | Attached to VPC A | Internet routing for the public subnet |

### VPC B — `10.20.0.0/16`

| Component | Address | Role |
|---|---|---|
| Private subnet | `10.20.1.0/24` | Hosts App-B |
| App-B | `10.20.1.10` | Private application host |

The CIDR ranges of VPC A and VPC B are non-overlapping.

## 3. VPC Peering

The VPC Peering Connection links the two VPCs using their private address ranges. Both VPCs are in the same AWS Region for this lab.

A peering connection alone does not enable traffic. The route tables and security groups must also permit the required communication.

## 4. Route Tables

| Route table | Required route | Target |
|---|---|---|
| VPC A public | `0.0.0.0/0` | Internet Gateway |
| VPC A private | `10.20.0.0/16` | VPC Peering Connection |
| VPC B private | `10.10.0.0/16` | VPC Peering Connection |

Each VPC also has its default local route for its own CIDR.

## 5. Traffic Flow

### Administrator to bastion

The administrator connects to the bastion's public IPv4 address using SSH. The bastion security group should allow TCP port 22 only from the administrator's authorized public IP.

### Bastion to App-B

The administrator opens an SSH session to the bastion, then connects from the bastion to `10.20.1.10` using SSH. App-B does not need a public IPv4 address for this private connection.

### App-A to App-B

App-A (`10.10.2.10`) and App-B (`10.20.1.10`) communicate over the VPC Peering Connection. The source and destination security groups must allow the selected protocol, and the reciprocal private routes must be present.

## 6. Security Controls

- Restrict SSH inbound sources to the required administrator or bastion addresses.
- Allow ICMP only where required for connectivity testing.
- Keep application instances private.
- Never place private SSH keys or AWS credentials in the repository.

## 7. Limitations

VPC peering is not transitive. A workload in one VPC cannot use a peered VPC as a general transit point to reach a third VPC. This lab documents direct communication between two VPCs only.
