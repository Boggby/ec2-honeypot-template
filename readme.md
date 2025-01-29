# EC2 Honeypot CloudFormation Stack

This project provides a CloudFormation template for deploying a free-tier eligible EC2 instance configured as a honeypot with customizable security groups.

The template creates an EC2 instance with two security groups: one for administrative access and another allowing unrestricted access. This setup is designed to attract potential attackers while providing a controlled environment for monitoring and analysis.

## Repository Structure

- `cloudformation/`: Contains CloudFormation templates
  - `ec2_honeypot_template.yaml`: Main CloudFormation template for the honeypot EC2 instance
- `readme.md`: This file, containing project documentation

## Usage Instructions

### Prerequisites

- AWS account with appropriate permissions
- AWS CLI installed and configured
- Basic understanding of AWS CloudFormation and EC2 services

### Deployment

1. Clone this repository to your local machine.

2. Navigate to the `cloudformation` directory:

   ```bash
   cd cloudformation
   ```

3. Deploy the CloudFormation stack using the AWS CLI:

   ```bash
   aws cloudformation create-stack --stack-name EC2Honeypot --template-body file://ec2_honeypot_template.yaml --parameters ParameterKey=AdminIpRange,ParameterValue=YOUR_IP_RANGE
   ```

   Replace `YOUR_IP_RANGE` with the IP range you want to allow SSH access from (e.g., `203.0.113.0/24`).

4. Monitor the stack creation process:

   ```bash
   aws cloudformation describe-stacks --stack-name EC2Honeypot
   ```

### Configuration

The `ec2_honeypot_template.yaml` template includes the following configurable parameter:

- `AdminIpRange`: The IP address range that can be used to SSH to the EC2 instance (default: 203.0.113.0/24, the default was set an RFC 5735 address so that if someone forgets to update it, they don't have to worry about it being accessed by anyone else. This value to needs to be updated to the IP range you want to allow SSH access from)

You can modify this parameter during stack creation or update it later using the AWS Management Console or CLI.

### Security Groups

The template creates two security groups:

1. `AdminSecurityGroup`: Allows SSH access (port 22) from the specified `AdminIpRange`.
2. `UnrestrictedSecurityGroup`: Allows unrestricted access (all protocols and ports) from any IP address (0.0.0.0/0).

### Accessing the EC2 Instance

After the stack is created, you can access the EC2 instance using the following steps:

1. Retrieve the instance's public IP address from the stack outputs:

   ```bash
   aws cloudformation describe-stacks --stack-name EC2Honeypot --query "Stacks[0].Outputs[?OutputKey=='PublicIP'].OutputValue" --output text
   ```

2. Download the generated key pair from the AWS Console. Go to EC2 > Key Pairs and locate the key pair with the name specified in the stack outputs.

3. Use the downloaded key pair to SSH into the instance:

   ```bash
   ssh -i /path/to/key.pem ubuntu@<instance-public-ip>
   ```

   Replace `/path/to/key.pem` with the path to your downloaded key pair file and `<instance-public-ip>` with the actual public IP address.

### Monitoring and Analysis

To effectively use this honeypot:

1. Set up logging and monitoring tools on the EC2 instance.
2. Regularly review the logs for suspicious activities.
3. Analyze attempted connections and attacks.
4. Update your security measures based on the gathered intelligence.

### Troubleshooting

1. Issue: Unable to SSH into the instance
   - Verify that your IP is within the `AdminIpRange`.
   - Check that you're using the correct key pair.
   - Ensure the instance is in the "running" state.

2. Issue: Stack creation fails
   - Review the stack events in the CloudFormation console for specific error messages.
   - Verify that you have the necessary permissions to create the resources.

3. Issue: Instance is not visible in the EC2 console
   - Confirm that you're in the correct AWS region.
   - Check the CloudFormation stack outputs for the instance ID.

For any persistent issues, enable CloudFormation debug logging:

```bash
aws cloudformation create-stack --stack-name EC2Honeypot --template-body file://ec2_honeypot_template.yaml --parameters ParameterKey=AdminIpRange,ParameterValue=YOUR_IP_RANGE --enable-termination-protection
```

This will provide more detailed logs during the stack creation process.

## Data Flow

The EC2 honeypot instance processes incoming network traffic as follows:

1. Incoming traffic reaches the EC2 instance.
2. The `UnrestrictedSecurityGroup` allows all traffic to pass through.
3. The operating system on the EC2 instance receives the traffic.
4. Logging and monitoring tools (if configured) record the traffic details.
5. For SSH connections:
   - Traffic is filtered by the `AdminSecurityGroup`.
   - Only connections from the specified `AdminIpRange` are allowed.

```
[Internet] -> [UnrestrictedSecurityGroup] -> [EC2 Instance] -> [Logging/Monitoring]
                                          |
[Admin IP] -> [AdminSecurityGroup] -------|
```

Note: Ensure proper monitoring and analysis tools are in place to capture and examine the incoming traffic effectively.

## Infrastructure

The CloudFormation template (`ec2_honeypot_template.yaml`) defines the following AWS resources:

- EC2::KeyPair
  - HoneypotKeyPair: Used for SSH access to the EC2 instance

- EC2::Instance
  - HoneypotInstance: t2.micro instance with Ubuntu Server 22.04 LTS AMI (selected based on the AWS region)

- EC2::SecurityGroup
  - AdminSecurityGroup: Allows SSH access from the specified IP range
  - UnrestrictedSecurityGroup: Allows unrestricted access from any IP

The template also includes outputs for easy access to important resource information such as the instance's public IP, DNS name, and key pair details.