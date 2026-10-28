# Implementation Guide

## Prerequisites

- An AWS account with permission to create VPC, EC2, route-table, security-group, and peering resources.
- Access to the AWS Management Console.
- An SSH key pair stored securely on the administrator's machine.
- An SSH client.

## Step 1 — Create the VPCs

Create VPC A with CIDR `10.10.0.0/16` and VPC B with CIDR `10.20.0.0/16`. Confirm the ranges do not overlap.

## Step 2 — Create the subnets

Create the following subnets:

- VPC A public: `10.10.1.0/24`
- VPC A private: `10.10.2.0/24`
- VPC B private: `10.20.1.0/24`

Use the intended Availability Zone and VPC for each subnet.

## Step 3 — Configure public routing

Attach an Internet Gateway to VPC A. Add `0.0.0.0/0` to the public route table with the Internet Gateway as target, and associate the table with the public subnet.

## Step 4 — Launch EC2 instances

Launch the bastion in VPC A's public subnet. Launch App-A in VPC A's private subnet and App-B in VPC B's private subnet. Use the intended private addresses and avoid assigning public IPs to the application instances.

## Step 5 — Create VPC peering

Create a peering request from VPC A to VPC B and accept it from the peer side. Confirm the connection status is Active.

## Step 6 — Add private routes

In VPC A's private route table, add destination `10.20.0.0/16` targeting the peering connection.

In VPC B's private route table, add destination `10.10.0.0/16` targeting the peering connection.

## Step 7 — Configure security groups

Allow SSH from the authorized administrator to the bastion. Allow SSH from the bastion's private address to the application instances. Add the required ICMP Echo Request rules for the cross-VPC ping tests.

## Step 8 — Test the lab

From the administrator's machine, connect to the bastion. From the bastion, connect to App-B by private IP. Run the connectivity tests described in `testing-and-troubleshooting.md`.

## Important

AWS console labels and options can change. Verify the selected VPC, subnet, route-table association, target, and security group before saving each change.
