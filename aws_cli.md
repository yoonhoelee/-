# AWS CLI Configuration Guide

## Overview
This guide demonstrates how to properly configure AWS CLI with multiple profiles on macOS.

## Prerequisites
- AWS CLI installed (`brew install awscli`)
- AWS Access Keys (from IAM console)
- Terminal access

## Configuration Steps

### 1. Default Profile Setup
```bash
aws configure
```
You will be prompted to enter:
```
AWS Access Key ID: [YOUR_ACCESS_KEY_ID]
AWS Secret Access Key: [YOUR_SECRET_ACCESS_KEY]
Default region name: ap-northeast-2
Default output format: json
```

### 2. Additional Profile Setup
```bash
aws configure --profile [PROFILE_NAME]
```
Enter the corresponding credentials when prompted:
```
AWS Access Key ID: [YOUR_PROFILE_ACCESS_KEY_ID]
AWS Secret Access Key: [YOUR_PROFILE_SECRET_ACCESS_KEY]
Default region name: ap-northeast-2
Default output format: json
```

## Generated Configuration Files

### ~/.aws/config
```ini
[default]
region = ap-northeast-2
output = json

[profile custom-profile]
region = ap-northeast-2
output = json
```

### ~/.aws/credentials
```ini
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY

[custom-profile]
aws_access_key_id = YOUR_PROFILE_ACCESS_KEY_ID
aws_secret_access_key = YOUR_PROFILE_SECRET_ACCESS_KEY
```

## Verifying Configuration

Test default profile:
```bash
aws sts get-caller-identity
```

Test specific profile:
```bash
aws sts get-caller-identity --profile [PROFILE_NAME]
```

## Security Best Practices

1. Never commit AWS credentials to version control
2. Use appropriate file permissions:
   ```bash
   chmod 600 ~/.aws/credentials
   chmod 600 ~/.aws/config
   ```
3. Regularly rotate your access keys
4. Use IAM roles when possible instead of access keys
5. Never share your access keys

## Useful Commands

List all configured profiles:
```bash
aws configure list-profiles
```

View current configuration:
```bash
aws configure list
```

View specific profile configuration:
```bash
aws configure list --profile [PROFILE_NAME]
```

## Common Issues
1. Permission denied errors: Check file permissions
2. Invalid credentials: Verify access keys are correct
3. Region errors: Confirm region is properly set
4. Profile not found: Ensure profile name matches in both config files
