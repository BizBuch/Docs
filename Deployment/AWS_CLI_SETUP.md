# AWS CLI Setup Guide

**Version:** 1.0.0 | **Updated:** 2026-02-21 | **Scope:** Complete AWS CLI installation, configuration, and usage guide for all platforms.

## What is AWS CLI?

The AWS Command Line Interface (CLI) is a unified tool to manage AWS services from your terminal. Instead of clicking through the AWS Console, you can automate tasks and manage resources using commands.

**Why use AWS CLI?**
- Automate repetitive tasks
- Manage AWS resources from scripts
- Faster than using the console
- Essential for DevOps workflows
- Required for VPN setup automation

---

## Prerequisites

Before installing AWS CLI, ensure you have:
- A computer with Ubuntu/Debian, macOS, or Windows
- Internet connection
- Administrative/sudo access (for some installations)
- Python 3.8 or later (recommended, but not always required)

---

## Installation Guide

Choose your operating system below.

### Ubuntu/Debian

#### Option 1: Using APT (Recommended)

```bash
# Update package manager
sudo apt-get update

# Install AWS CLI
sudo apt-get install awscli

# Verify installation
aws --version
```

**Pros:** Simple, automatic updates
**Cons:** May not have the latest version

#### Option 2: Using Python pip (Latest Version)

```bash
# Install Python and pip (if not already installed)
sudo apt-get install python3 python3-pip

# Install AWS CLI using pip
pip3 install awscli --upgrade --user

# Add to PATH (add this line to ~/.bashrc or ~/.zshrc)
export PATH=$PATH:~/.local/bin

# Reload your shell
source ~/.bashrc  # or source ~/.zshrc

# Verify installation
aws --version
```

**Pros:** Always latest version, more control
**Cons:** Manual PATH management needed

#### Option 3: Using a Virtual Environment (Best Practice)

```bash
# Create a virtual environment
python3 -m venv ~/aws-cli-env

# Activate the environment
source ~/aws-cli-env/bin/activate

# Install AWS CLI
pip install awscli --upgrade

# Verify installation
aws --version

# Deactivate when done (optional)
deactivate
```

**Pros:** Isolated, doesn't affect system Python
**Cons:** Need to activate environment each time

---

### macOS

#### Option 1: Using Homebrew (Recommended)

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install AWS CLI
brew install awscli

# Verify installation
aws --version

# Update AWS CLI (when new versions are available)
brew upgrade awscli
```

**Pros:** Simple, with automatic updates
**Cons:** Requires Homebrew

#### Option 2: Using Python pip

```bash
# Install Python 3 (if not already installed)
brew install python3

# Install AWS CLI
pip3 install awscli --upgrade --user

# Add to PATH (add to ~/.zshrc or ~/.bash_profile)
export PATH=$PATH:~/.local/bin

# Reload your shell
source ~/.zshrc  # or source ~/.bash_profile

# Verify installation
aws --version
```

**Pros:** Works without Homebrew
**Cons:** Manual PATH management

#### Option 3: Using Direct Installer

1. Download the AWS CLI installer:
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-macos.zip" -o "awscliv2.zip"
   ```

2. Unzip and install:
   ```bash
   unzip awscliv2.zip
   sudo ./aws/install
   ```

3. Verify installation:
   ```bash
   aws --version
   ```

**Pros:** Official installer, most reliable
**Cons:** Manual download required

---

### Windows

#### Option 1: Using Chocolatey (Recommended)

Prerequisites: Install Chocolatey first if you don't have it.

```powershell
# Open PowerShell as Administrator
# Install Chocolatey (if not already installed)
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install AWS CLI
choco install awscli

# Verify installation
aws --version
```

**Pros:** Simple, automatic updates
**Cons:** Requires Chocolatey and admin access

#### Option 2: Using MSI Installer (Easiest)

1. Download the AWS CLI MSI installer:
   - Visit: https://awscli.amazonaws.com/AWSCLIV2.msi
   - Or use PowerShell:
     ```powershell
     curl "https://awscli.amazonaws.com/AWSCLIV2.msi" -o "AWSCLIV2.msi"
     ```

2. Install the MSI:
   - Double-click `AWSCLIV2.msi`
   - Follow the installation wizard
   - Click "Finish" when complete

3. Verify installation (open new Command Prompt or PowerShell):
   ```powershell
   aws --version
   ```

**Pros:** Simple wizard, no admin PowerShell needed
**Cons:** Manual installer download

#### Option 3: Using Python pip

1. Install Python 3.8 or later:
   - Download from: https://www.python.org/downloads/
   - Run installer, check **"Add Python to PATH"**
   - Click "Install Now"

2. Open Command Prompt as Administrator:
   ```powershell
   pip install awscli --upgrade
   ```

3. Verify installation:
   ```powershell
   aws --version
   ```

**Pros:** Works if you already have Python
**Cons:** Python dependency

---

## Configuration

### Step 1: Get AWS Credentials

You need an **Access Key ID** and **Secret Access Key** to configure the CLI.

**Get from AWS Console:**
1. Sign in to AWS: https://console.aws.amazon.com/
2. Click your **account name** (top right) → **Security Credentials**
3. Go to **Access Keys** section
4. Click **Create New Access Key**
5. Copy both:
   - **Access Key ID** (starts with `AKIA`)
   - **Secret Access Key** (long alphanumeric string)

⚠️ **Important:** Secret Access Key appears **only once**. Save it securely.

### Step 2: Run aws configure

Open your terminal/Command Prompt and run:

```bash
aws configure
```

You'll be prompted for 4 pieces of information:

```
AWS Access Key ID [None]: AKIA2ZWPK5XXXXXXXXXXX
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: json
```

**What each means:**

| Field | Description | Example |
|-------|-------------|---------|
| **Access Key ID** | Your AWS account identifier | `AKIA2ZWPK5XXXXXXXXXXX` |
| **Secret Access Key** | Your AWS password (keep secret!) | `wJalrXUtnFEMI/...` |
| **Default region** | AWS region for commands | `us-east-1`, `us-west-2` |
| **Output format** | How results display | `json`, `yaml`, `table`, `text` |

**Region examples:**
- `us-east-1` (N. Virginia)
- `us-west-2` (Oregon)
- `eu-west-1` (Ireland)
- `ap-south-1` (Mumbai)
- Full list: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html

### Step 3: Verify Configuration

Test your setup with:

```bash
aws sts get-caller-identity
```

**Expected output:**
```json
{
    "UserId": "AIDAI...",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}
```

If you get an error like "Unable to locate credentials", your configuration failed. Rerun `aws configure`.

---

## Configuration Files

AWS CLI stores credentials in two locations:

### Linux/macOS

**Credentials file:** `~/.aws/credentials`
```
[default]
aws_access_key_id = AKIA2ZWPK5XXXXXXXXXXX
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

**Config file:** `~/.aws/config`
```
[default]
region = us-east-1
output = json
```

### Windows

**Credentials file:** `C:\Users\YourUsername\.aws\credentials`
```
[default]
aws_access_key_id = AKIA2ZWPK5XXXXXXXXXXX
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

**Config file:** `C:\Users\YourUsername\.aws\config`
```
[default]
region = us-east-1
output = json
```

---

## Common AWS CLI Commands

### General Information

```bash
# Show AWS CLI version
aws --version

# Get account information
aws sts get-caller-identity

# Get current region
aws configure get region

# List all configured profiles
aws configure list
```

### RDS Commands (Example)

```bash
# List all RDS databases
aws rds describe-db-instances

# Get specific database info
aws rds describe-db-instances --db-instance-identifier bizbuch-db

# Get VPC of a database
aws rds describe-db-instances \
  --db-instance-identifier bizbuch-db \
  --query "DBInstances[0].DBSubnetGroup.VpcId"
```

### EC2 Commands (Example)

```bash
# List security groups
aws ec2 describe-security-groups

# List VPCs
aws ec2 describe-vpcs

# List subnets
aws ec2 describe-subnets
```

### ACM Commands (Certificate Management)

```bash
# List certificates
aws acm list-certificates

# Import a certificate
aws acm import-certificate \
  --certificate fileb://path/to/cert.crt \
  --private-key fileb://path/to/key.key
```

### Help

```bash
# Get help on any command
aws help
aws rds help
aws rds describe-db-instances help
```

---

## Using Multiple AWS Accounts

If you manage multiple AWS accounts, you can configure multiple profiles:

```bash
# Configure a second profile
aws configure --profile production

# Use a specific profile in commands
aws s3 ls --profile production

# Set default profile
export AWS_PROFILE=production
```

**View profiles in `~/.aws/config`:**
```
[default]
region = us-east-1

[production]
region = us-west-2

[staging]
region = eu-west-1
```

---

## Environment Variables

You can also set AWS credentials using environment variables (useful for CI/CD):

```bash
# Linux/macOS
export AWS_ACCESS_KEY_ID=AKIA2ZWPK5XXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_DEFAULT_REGION=us-east-1

# Windows PowerShell
$env:AWS_ACCESS_KEY_ID="AKIA2ZWPK5XXXXXXXXXXX"
$env:AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
$env:AWS_DEFAULT_REGION="us-east-1"

# Windows Command Prompt
set AWS_ACCESS_KEY_ID=AKIA2ZWPK5XXXXXXXXXXX
set AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
set AWS_DEFAULT_REGION=us-east-1
```

---

## Security Best Practices

### ✅ DO:

- **Use IAM users** instead of root account credentials
- **Rotate access keys** every 90 days
- **Store secrets securely** (password manager, AWS Secrets Manager)
- **Use MFA** (Multi-Factor Authentication) on your account
- **Use temporary credentials** (STS) for scripts
- **Restrict IAM permissions** to minimum required
- **Monitor API calls** with CloudTrail

### ❌ DON'T:

- Share credentials with anyone
- Commit `.aws/credentials` to git or version control
- Hardcode credentials in scripts or source code
- Use root account credentials for daily work
- Share credentials via email or messaging apps
- Store credentials in plain text files
- Leave long-lived credentials unused

### Revoking Compromised Credentials

If you accidentally expose your access key:

1. Go to AWS Console → IAM → Users
2. Select your user → Security Credentials
3. Delete the compromised access key immediately
4. Create a new access key
5. Update your local `.aws/credentials` file

---

## Troubleshooting

### Issue: "Unable to locate credentials"

**Cause:** AWS CLI can't find your credentials

**Solution:**
```bash
# Check if credentials file exists
cat ~/.aws/credentials  # Linux/macOS
type C:\Users\YourUsername\.aws\credentials  # Windows

# Reconfigure AWS CLI
aws configure
```

### Issue: "Access Denied" or "UnauthorizedOperation"

**Cause:** Your IAM user lacks permissions

**Solution:**
1. Check your IAM user permissions in AWS Console
2. Ask administrator to grant necessary permissions
3. Verify you're using correct access key (not expired)

### Issue: "region is not specified" error

**Cause:** No default region configured

**Solution:**
```bash
# Check current region
aws configure get region

# Set region
aws configure set region us-east-1

# Or use in commands
aws s3 ls --region us-east-1
```

### Issue: Command works on macOS but not on Windows

**Cause:** Bash vs PowerShell syntax differences

**Solution:** Check command escaping and path separators:
- Linux/macOS: uses `/` for paths
- Windows: use `\` or `\\` for paths
- PowerShell: may need different quotes

### Issue: "aws: command not found"

**Cause:** AWS CLI not in system PATH

**Solution:**
```bash
# Find AWS CLI location
which aws  # Linux/macOS
where aws  # Windows PowerShell

# Add to PATH (Linux/macOS)
export PATH=$PATH:/path/to/aws/bin

# Add to PATH (Windows) - use System Environment Variables
```

---

## Upgrading AWS CLI

### Linux/macOS (using apt)

```bash
sudo apt-get update
sudo apt-get upgrade awscli
```

### macOS (using Homebrew)

```bash
brew upgrade awscli
```

### Using pip

```bash
pip3 install awscli --upgrade --user
```

### Windows (using Chocolatey)

```powershell
choco upgrade awscli
```

### Windows (using MSI)

Download the latest installer from: https://awscli.amazonaws.com/AWSCLIV2.msi

---

## Useful Resources

- [AWS CLI Official Documentation](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [AWS CLI GitHub](https://github.com/aws/aws-cli)
- [AWS CLI Command Completion](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-completion.html)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

## Related Guides

- [AWS Client VPN Setup](AWS_CLIENT_VPN_SETUP.md) - Uses AWS CLI for VPN endpoint management
- [AWS IAM Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)
