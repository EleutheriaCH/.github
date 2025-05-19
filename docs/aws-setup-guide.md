# AWS CLI and SSH Proxy Configuration Guide

## Overview

This guide will help you set up AWS CLI and the SSH proxy command for accessing AWS resources using ssh, scp, rsync commands. 
The setup is compatible with both macOS and Linux systems.

## Prerequisites

- macOS or Linux operating system
- Terminal access with administrative privileges
- AWS Access Key ID and Secret Access Key (to be provided separately)

## Step 1: Install AWS CLI

### Option A: Using Package Managers (Recommended)

#### On macOS (using Homebrew)

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install AWS CLI
brew install awscli
```

#### On Linux (using package managers)

**Ubuntu/Debian:**
```bash
# Update package manager
sudo apt update

# Install AWS CLI
sudo apt install awscli -y
```

**CentOS/RHEL/Amazon Linux:**
```bash
# Install AWS CLI
sudo yum install awscli -y
# or for newer versions
sudo dnf install awscli -y
```

### Option B: Using Python pip (Universal method)

```bash
# Install pip if not already installed
# On macOS
brew install python

# On Ubuntu/Debian
sudo apt install python3-pip -y

# On CentOS/RHEL
sudo yum install python3-pip -y

# Install AWS CLI via pip
pip3 install awscli --upgrade --user
```

### Verify AWS CLI Installation

```bash
aws --version
```

Expected output should show AWS CLI version information.

## Step 2: Configure AWS CLI

### Basic Configuration

Run the following command and enter your credentials when prompted:

```bash
aws configure
```

You will be prompted to enter:
- **AWS Access Key ID**: [Enter the provided Access Key ID]
- **AWS Secret Access Key**: [Enter the provided Secret Access Key]
- **Default region name**: [Enter our operating region: `eu-west-2`]
- **Default output format**: [Enter `json` (recommended)]

### Alternative: Manual Configuration

If you prefer to configure manually, create the AWS configuration files:

```bash
# Create AWS config directory
mkdir -p ~/.aws

# Create credentials file
cat > ~/.aws/credentials << EOF
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY
EOF

# Create config file
cat > ~/.aws/config << EOF
[default]
region = eu-west-2
output = json
EOF
```

**Note**: Replace `YOUR_ACCESS_KEY_ID` and `YOUR_SECRET_ACCESS_KEY` with the actual values provided to you.

### Verify AWS Configuration

```bash
aws sts get-caller-identity
```

This should return information about your AWS account if configured correctly.

## Step 3: Install AWS CLI Session Manager Plugin

### MacOS (brew)

```bash
brew install session-manager-plugin
```

### MacOS (using the signed installer)

```bash
# x86_64
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/mac/session-manager-plugin.pkg" -o "session-manager-plugin.pkg"
# Apple Silicon
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/mac_arm64/session-manager-plugin.pkg" -o "session-manager-plugin.pkg"

sudo installer -pkg session-manager-plugin.pkg -target /
sudo ln -s /usr/local/sessionmanagerplugin/bin/session-manager-plugin /usr/local/bin/session-manager-plugin
```

### Install the Session Manager plugin on Debian/Ubuntu

```bash
# x86_64
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_64bit/session-manager-plugin.deb" -o "session-manager-plugin.deb"
# x86
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_32bit/session-manager-plugin.deb" -o "session-manager-plugin.deb"
# arm64
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_arm64/session-manager-plugin.deb" -o "session-manager-plugin.deb"

sudo dpkg -i session-manager-plugin.deb
```

## Step 4: Install AWS SSM SSH Proxy Command

### Download and Install

```bash
# Download the latest release
curl "https://raw.githubusercontent.com/qoomon/aws-ssm-ssh-proxy-command/refs/heads/main/aws-ssm-ssh-proxy-command.sh" -o ~/.ssh/aws-ssm-ssh-proxy-command.sh

### Ensure it is executable
chmod +x ~/.ssh/aws-ssm-ssh-proxy-command.sh
```

## Step 5: Configure SSH for AWS SSM

### Update SSH Configuration

Add the following configuration to your SSH config file:

```bash
# Edit SSH config file
nano ~/.ssh/config
```

Add this configuration block:

```
# SSH over Session Manager
host i-* mi-*
  IdentityFile ~/.ssh/id_rsa
  ProxyCommand ~/.ssh/aws-ssm-ec2-proxy-command.sh %h %r %p ~/.ssh/id_rsa.pub
  StrictHostKeyChecking no
```

### Additional notes

Ensure AWS CLI environemnt variables are set properly if you use different profile than default
    export AWS_PROFILE=yourprofile


## Step 6: Test the Configuration
(Replace "i-1234567890abcdef0" with actual instance ID)

### Test Commands

```bash

ssh ec2-user@i-1234567890abcdef0

scp -r ./my-local-dir ec2-user@i-1234567890abcdef0:/var/www/html

rsync -av ./my-local-dir/ ec2-user@i-1234567890abcdef0:/var/www/html
```

## Step 6: Security Best Practices

### Set Proper File Permissions

```bash
# Secure AWS credentials
chmod 600 ~/.aws/credentials
chmod 600 ~/.aws/config

# Secure SSH config
chmod 600 ~/.ssh/config

# Secure SSH keys (if you have them)
chmod 600 ~/.ssh/*.pem
```

### Environment Variables (Optional)

You can also set AWS credentials as environment variables:

```bash
# Add to your shell profile (.bashrc, .zshrc, etc.)
export AWS_ACCESS_KEY_ID="your-access-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-access-key"
export AWS_DEFAULT_REGION="eu-west-2"
```

## Troubleshooting

### Common Issues and Solutions

#### 1. "aws: command not found"
- Ensure AWS CLI is in your PATH
- Try adding `/usr/local/bin` to your PATH: `export PATH="/usr/local/bin:$PATH"`

#### 2. "Access Denied" errors
- Verify your AWS credentials are correctly configured
- Ensure your IAM user has the necessary permissions for SSM

#### 3. SSH connection timeout
- Ensure the target instance has SSM agent installed and running
- Verify the instance has the necessary IAM role for SSM


## Additional Resources

- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
- [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [AWS SSM SSH Proxy Command GitHub](https://github.com/qoomon/aws-ssm-ssh-proxy-command)



**Last Updated**: 2025-05-19
**Version**: 1.0
