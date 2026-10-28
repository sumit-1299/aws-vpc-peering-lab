# Testing and Troubleshooting

## Recorded test results

| Test | Expected result | Recorded result |
|---|---|---|
| SSH from administrator to bastion | Shell prompt on bastion | Successful |
| SSH from bastion to App-B (`10.20.1.10`) | Shell prompt on App-B | Successful |
| Ping from App-B to App-A (`10.10.2.10`) | ICMP replies | 4 received, 0% packet loss |

These results reflect the lab session and the saved evidence. Re-run the tests if the resources or security rules change.

## SSH to the bastion

From the administrator's machine, use the correct key path and the bastion's current public IPv4 address:

```bash
chmod 400 /path/to/ewb-lab.pem
ssh -i /path/to/ewb-lab.pem ec2-user@BASTION_PUBLIC_IP
```

Replace `BASTION_PUBLIC_IP` with the instance's current public IPv4 address. Stop/start operations can change an automatically assigned public IP.

A connection timeout usually indicates a network path or access-rule problem. Check that the instance is running, status checks pass, the public subnet has an Internet Gateway route, and the bastion security group allows SSH from the administrator's current public IP.

## SSH from the bastion to App-B

If the private key is available to the bastion session through a securely configured SSH agent, connect using App-B's private address:

```bash
ssh ec2-user@10.20.1.10
```

Do not copy a private key into a public repository. Prefer secure agent forwarding or an appropriate managed access method.

## Test cross-VPC ICMP

From App-B:

```bash
ping -c 4 10.10.2.10
```

A successful test should show ICMP replies from App-A.

## If ping fails

Check the following in order:

1. Confirm the VPC Peering Connection is Active.
2. Confirm VPC A's private route table has `10.20.0.0/16` targeting the peering connection.
3. Confirm VPC B's private route table has `10.10.0.0/16` targeting the peering connection.
4. Confirm the correct private route tables are associated with the application subnets.
5. Confirm the destination security group permits ICMP Echo Request from the source instance.
6. Check network ACL rules if they have been customized.
7. Verify the destination IP address and instance state.

A successful ping proves ICMP works for the tested path; it does not prove SSH or every other protocol is allowed.

## If SSH from bastion to App-B fails

Check the App-B security group for TCP port 22 from the bastion's private IP. Verify the SSH service is running, the username matches the AMI, the key or SSH agent is valid, and the route tables allow the traffic.

## Security reminder

Do not solve troubleshooting issues by opening SSH to `0.0.0.0/0` or by committing private keys. Keep access rules narrowly scoped.
