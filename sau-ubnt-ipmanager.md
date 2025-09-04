# sau-ubnt-ipmanager

A simple netplan network configuration management tool for Ubuntu 22.04 LTS systems.
Was created for Ubuntu 22.04 but seems to work fine on Ubuntu 24 as well,
probably future netplan based editions also.
It was created to battle the annoying cloud-init default stuff that install does, just a quicker way to disable cloud-init
networking and switch from dhcp to static or vice versa, for any system with a single interface. 

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
  - [Command Options](#command-options)
  - [Exit Codes](#exit-codes)
- [Examples](#examples)
  - [DHCP Configuration](#dhcp-configuration)
  - [Static IP Configuration](#static-ip-configuration)
  - [Cloud-init Cleanup](#cloud-init-cleanup)
  - [Advanced Scenarios](#advanced-scenarios)
- [Safety Features](#safety-features)
- [File Management](#file-management)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Overview

`sau-ubnt-ipmanager` is a command-line tool designed to safely manage netplan network configurations on Ubuntu systems. It provides a controlled way to switch between DHCP and static IP configurations while ensuring system safety through backup mechanisms and validation checks.

### Key Characteristics
- **Non-destructive by default**: Changes are staged but not applied automatically
- **Netplan-exclusive**: Only works with Netplan-managed systems
- **Backup-oriented**: All changes are backed up with timestamps
- **SSH-aware**: Warns users when applying changes over SSH connections
- **Cloud-init compatible**: Can clean up cloud-init network configurations

### My most common uses
- To set/keep dhcp after install, remove cloud init:
  - `sau-ubnt-ipmanager --dhcp --clean-cloud-init`
  - `sau-ubnt-ipmanager -d -c`
- To set static and remove cloud init network config:
  - `sau-ubnt-ipmanager --static 10.38.73.249/25 --gateway 10.38.73.129 --nameservers 10.5.100.9,10.5.100.10 --clean-cloud-init`
  - `sau-ubnt-ipmanager -s 10.38.73.249/25 -g 10.38.73.129 -n 10.5.100.9,10.5.100.10 -c`
- When you already removed cloud init and just need to change static ip:
  - `sau-ubnt-ipmanager --static 10.38.73.102/25 --gateway 10.38.73.129 --nameservers 10.5.100.9,10.5.100.10`
  - `sau-ubnt-ipmanager -s 10.38.73.102/25 -g 10.38.73.129 -n 10.5.100.9,10.5.100.10`

## Features

### Core Features
- ✅ **Netplan Management**: Verifies and manages Netplan configurations
- ✅ **DHCP Configuration**: Sets up DHCP with MAC-based identification
- ✅ **Static IP Configuration**: Configures static IP with full network parameters
- ✅ **Automatic Backup**: Creates timestamped backups of all modified files
- ✅ **SSH Detection**: Warns when applying changes over SSH
- ✅ **Cloud-init Cleanup**: Removes cloud-init network interference
- ✅ **Validation**: Comprehensive input validation for IPs, CIDR, and network parameters
- ✅ **Error Handling**: Detailed error messages to stderr with specific exit codes

### Safety Features
- Requires root privileges
- Validates all network parameters before applying
- Creates backups before any file modification
- Non-apply mode by default (requires explicit flag to apply)
- SSH session protection with user confirmation

## Requirements

### System Requirements
- Ubuntu 22.04 LTS
- Netplan installed and configured
- Root/sudo privileges
- Basic networking tools (`ip`, `netplan`)

### Network Requirements
- At least one network interface
- Valid network configuration parameters for static IP setup

## Installation

### Method 1: Direct Download
```bash
# Download the script
wget https://your-repo/sau-ubnt-ipmanager
# OR
curl -O https://your-repo/sau-ubnt-ipmanager

# Make executable
chmod +x sau-ubnt-ipmanager

# Move to system path (optional)
sudo mv sau-ubnt-ipmanager /usr/local/bin/
```

### Method 2: Manual Creation
```bash
# Create the script file
sudo nano /usr/local/bin/sau-ubnt-ipmanager

# Paste the script content

# Make executable
sudo chmod +x /usr/local/bin/sau-ubnt-ipmanager
```

## Usage

### Basic Syntax
```bash
sudo sau-ubnt-ipmanager [OPTIONS]
```

### Command Options

| Option | Long Form | Description | Required |
|--------|-----------|-------------|----------|
| `-d` | `--dhcp` | Configure DHCP mode | Choice: -d or -s |
| `-s IP/CIDR` | `--static IP/CIDR` | Configure static IP | Choice: -d or -s |
| `-g GATEWAY` | `--gateway GATEWAY` | Set gateway IP | With -s only |
| `-n DNS1,DNS2` | `--nameservers DNS1,DNS2` | Set DNS servers (comma-separated) | With -s only |
| `-i IFACE` | `--interface IFACE` | Specify network interface | Optional (auto-detected) |
| `-c` | `--clean-cloud-init` | Remove cloud-init network config | Optional |
| `-a` | `--apply` | Apply configuration immediately | Optional |
| `-h` | `--help` | Show help message | - |

### Exit Codes

| Code | Description | Meaning |
|------|-------------|---------|
| `0` | Success | Operation completed successfully |
| `1` | General Error | Unspecified error occurred |
| `2` | Not Netplan Managed | System is not using Netplan |
| `3` | Invalid Arguments | Incorrect or missing parameters |
| `4` | Permission Denied | Not running as root |
| `5` | SSH Warning Abort | User aborted due to SSH connection |

## Examples

### DHCP Configuration

#### Basic DHCP Setup
```bash
# Configure DHCP (changes staged, not applied)
sudo sau-ubnt-ipmanager --dhcp

# Configure DHCP and apply immediately
sudo sau-ubnt-ipmanager --dhcp --apply
```

#### DHCP with Specific Interface
```bash
# Use specific interface
sudo sau-ubnt-ipmanager -d -i eth0

# Auto-detect interface (default behavior)
sudo sau-ubnt-ipmanager -d
```

### Static IP Configuration

#### Basic Static IP Setup
```bash
# Configure static IP (staged)
sudo sau-ubnt-ipmanager \
  --static 192.168.1.100/24 \
  --gateway 192.168.1.1 \
  --nameservers 8.8.8.8,8.8.4.4

# Short form
sudo sau-ubnt-ipmanager \
  -s 192.168.1.100/24 \
  -g 192.168.1.1 \
  -n 8.8.8.8,8.8.4.4
```

#### Static IP with Multiple DNS Servers
```bash
# Multiple nameservers
sudo sau-ubnt-ipmanager \
  -s 10.0.0.50/24 \
  -g 10.0.0.1 \
  -n 1.1.1.1,1.0.0.1,8.8.8.8

# With specific interface
sudo sau-ubnt-ipmanager \
  -s 172.16.0.10/16 \
  -g 172.16.0.1 \
  -n 172.16.0.2,172.16.0.3 \
  -i enp0s3
```

### Cloud-init Cleanup

#### Remove Cloud-init Network Management
```bash
# Just remove cloud-init configs
sudo sau-ubnt-ipmanager --clean-cloud-init --dhcp

# Clean and set static IP
sudo sau-ubnt-ipmanager \
  --clean-cloud-init \
  --static 192.168.1.50/24 \
  --gateway 192.168.1.1 \
  --nameservers 192.168.1.1
```

### Advanced Scenarios

#### Complete System Reconfiguration
```bash
# Remove all existing configs, set static IP, and apply
sudo sau-ubnt-ipmanager \
  --clean-cloud-init \
  --static 192.168.100.10/24 \
  --gateway 192.168.100.1 \
  --nameservers 8.8.8.8,8.8.4.4 \
  --interface eth0 \
  --apply
```

#### Migration from Cloud-init to Local Management
```bash
# Step 1: Clean cloud-init and configure (staged)
sudo sau-ubnt-ipmanager -c -d

# Step 2: Verify configuration
cat /etc/netplan/01-network-config.yaml

# Step 3: Apply when ready
sudo netplan apply
```

#### Dry Run for Testing
```bash
# Configure without applying (default behavior)
sudo sau-ubnt-ipmanager \
  -s 192.168.1.100/24 \
  -g 192.168.1.1 \
  -n 8.8.8.8

# Review generated config
sudo cat /etc/netplan/01-network-config.yaml

# Apply manually when satisfied
sudo netplan apply
```

## Safety Features

### SSH Connection Protection
When using the `--apply` flag while connected via SSH:
1. Script detects SSH connection
2. Displays warning message
3. Requires explicit confirmation ("yes")
4. Exits with code 5 if user declines

Example SSH warning:
```
WARNING: You are connected via SSH!
Applying network configuration may disconnect your session.
Do you want to continue? (yes/no):
```

### Backup System
All modified or removed files are backed up to `/etc/netplan/backups/` with timestamps:
```
/etc/netplan/backups/
├── 50-cloud-init.yaml.20240115_143022
├── 70-netplan-set.yaml.20240115_143022
└── 01-network-config.yaml.20240115_143022
```

### Validation Checks
- **IP Address**: Validates format and octet ranges (0-255)
- **CIDR Notation**: Validates IP and prefix length (0-32)
- **Gateway**: Ensures valid IP format
- **DNS Servers**: Validates each server in the comma-separated list
- **Interface**: Verifies interface exists on the system

## File Management

### Files Created
- `/etc/netplan/01-network-config.yaml` - Primary configuration file
- `/etc/netplan/backups/` - Backup directory

### Files Modified
- `/etc/cloud/cloud.cfg` - When using `--clean-cloud-init`

### Files Removed (with backup)
- `/etc/netplan/50-cloud-init.yaml` - Cloud-init generated config
- `/etc/netplan/70-netplan-set.yaml` - Netplan-set generated config
- Other YAML files in `/etc/netplan/` (except `01-network-config.yaml`)

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Netplan is not installed on this system"
**Solution**: Install netplan
```bash
sudo apt update
sudo apt install netplan.io
```

#### Issue: "This script must be run as root"
**Solution**: Use sudo
```bash
sudo sau-ubnt-ipmanager --dhcp
```

#### Issue: "Could not determine primary network interface"
**Solution**: Specify interface manually
```bash
sudo sau-ubnt-ipmanager -d -i eth0
```

#### Issue: "Invalid IP/CIDR notation"
**Solution**: Use correct format
```bash
# Correct
sudo sau-ubnt-ipmanager -s 192.168.1.100/24 ...

# Incorrect
sudo sau-ubnt-ipmanager -s 192.168.1.100 ...  # Missing CIDR
sudo sau-ubnt-ipmanager -s 192.168.1.100/33 ... # Invalid prefix
```

#### Issue: Network connectivity lost after applying
**Recovery Steps**:
1. Access system via console
2. Restore from backup:
```bash
# Find latest backup
ls -lt /etc/netplan/backups/

# Restore previous config
sudo cp /etc/netplan/backups/[backup-file] /etc/netplan/01-network-config.yaml

# Apply restored config
sudo netplan apply
```

### Verification Commands

#### Check Current Configuration
```bash
# View active netplan config
cat /etc/netplan/*.yaml

# Test configuration without applying
sudo netplan try

# View current network status
ip addr show
ip route show
```

#### Validate Configuration
```bash
# Generate and validate without applying
sudo netplan generate

# Check for syntax errors
sudo netplan --debug generate
```

## Best Practices

### 1. Always Test First
```bash
# Configure without applying
sudo sau-ubnt-ipmanager [options]

# Review the generated config
cat /etc/netplan/01-network-config.yaml

# Test with netplan try (automatic rollback)
sudo netplan try

# Apply permanently only when confirmed working
sudo netplan apply
```

### 2. Document Your Configuration
```bash
# Add comments to track changes
echo "# Changed on $(date) by $(whoami)" | \
  sudo tee -a /etc/netplan/01-network-config.yaml
```

### 3. Maintain Backups
```bash
# Manual backup before major changes
sudo cp -r /etc/netplan /etc/netplan.backup.$(date +%Y%m%d)

# Verify backups exist
ls -la /etc/netplan/backups/
```

### 4. Use Console Access for Critical Changes
- When possible, use local console access for network changes
- Have a recovery plan before applying changes remotely
- Know your IPMI/iDRAC/iLO credentials if available

### 5. Clean Cloud-init When Taking Local Control
```bash
# Prevent cloud-init from overwriting your configs
sudo sau-ubnt-ipmanager --clean-cloud-init --dhcp
```

### 6. Verify Network Requirements
Before configuring static IP:
- Confirm IP address is not in use: `ping -c 3 192.168.1.100`
- Verify gateway is reachable: `ping -c 3 192.168.1.1`
- Test DNS servers: `nslookup google.com 8.8.8.8`

## License

This tool is provided as-is for managing Ubuntu network configurations. Use at your own risk and always maintain proper backups.

## Support

For issues, questions, or contributions, please contact your system administrator or refer to the Ubuntu Netplan documentation at https://netplan.io/

---

*Last updated: Documentation reflects sau-ubnt-ipmanager version as of script creation date*
