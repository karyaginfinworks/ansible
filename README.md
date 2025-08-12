# Ansible Server Setup Project

## 📋 Overview

Comprehensive Ansible project for automating enterprise server setup and optimization. This playbook provides a complete solution for configuring critical server infrastructure with focus on security, performance, and reliability. The project consists of four specialized roles that work together to deliver a fully optimized server environment.

### Key Features

- **Network Configuration**: Automated network interface management with netplan
- **CPU Optimization**: Performance tuning for Intel and AMD processors
- **Disk Encryption**: LUKS-based disk encryption with automatic mounting
- **System Analysis**: Comprehensive system information gathering and reporting

### Architecture

The playbook follows a modular architecture with four independent roles:

```
ansible-server-setup/
├── all.yml                 # Main playbook orchestrating all roles
├── inventory.yml           # Host definitions and variables
├── ansible.cfg             # Ansible configuration settings
├── group_vars/            # Global variables
└── roles/                 # Individual role implementations
    ├── network-setup/     # Network interface configuration
    ├── cpu-optimization/  # CPU performance tuning
    ├── disk-encryption/   # LUKS disk encryption
    └── system-info/       # System information gathering
```

## 🔧 Prerequisites

### System Requirements
- **Ansible**: 2.12+ (recommended 2.18+)
- **Python**: 3.8+
- **Target OS**: Ubuntu 18.04+ (primary support)
- **SSH Access**: Key-based authentication to target servers
- **Privileges**: Sudo access on target servers


## 🚀 Installation

### 1. Clone Repository
```bash
git clone <repository-url>
cd ansible-server-setup
```

### 2. Install Ansible (if not installed)
```bash
# Ubuntu/Debian
apt update && apt install ansible

# RHEL/CentOS
yum install ansible

# macOS
brew install ansible
```

### 3. Install Required Collections
```bash
ansible-galaxy collection install \
    ansible.posix \
    community.general \
    community.crypto \
    community.docker \
    ansible.utils \
    community.network \
    --force
```

### 4. Verify Installation
```bash
ansible --version
ansible-galaxy collection list
```

## ⚙️ Configuration

### Inventory Configuration

Edit `inventory.yml` to define your target servers:

```yaml
all:
  hosts:
    target_server:
      ansible_host: YOUR_SERVER_IP
      ansible_user: YOUR_USERNAME
      ansible_ssh_private_key_file: /path/to/your/private/key
      
      # Role-specific variables
      disk_to_encrypt: /dev/vdb              # Disk for encryption
      encryption_passphrase: "secure_pass"   # LUKS passphrase
      vm_if_name: "net0"                     # Target interface name
      
  vars:
    ansible_python_interpreter: /usr/bin/python3
```


### Role-Specific Variables

Each role can be customized through variables defined in:
- `roles/<role_name>/defaults/main.yml` - Default values
- `roles/<role_name>/vars/main.yml` - Role-specific variables
- `inventory.yml` - Host-specific overrides

## 🎯 Usage

### Full Server Setup
Deploy all components with a single command:
```bash
ansible-playbook -i inventory.yml all.yml
```

### Verbose Output
Enable detailed logging for troubleshooting:
```bash
ansible-playbook -i inventory.yml all.yml -v
```

### Role-Specific Execution
Execute individual roles using tags:

```bash
# Network configuration only
ansible-playbook -i inventory.yml all.yml --tags network

# CPU optimization only
ansible-playbook -i inventory.yml all.yml --tags cpu

# Disk encryption only
ansible-playbook -i inventory.yml all.yml --tags disk

# System information gathering only
ansible-playbook -i inventory.yml all.yml --tags info
```

### Multiple Tags
Combine multiple roles:
```bash
# Network and CPU optimization
ansible-playbook -i inventory.yml all.yml --tags network,cpu

# Everything except disk encryption
ansible-playbook -i inventory.yml all.yml --skip-tags disk
```

### Dry Run
Test configuration without making changes:
```bash
ansible-playbook -i inventory.yml all.yml --check
```

### Limit to Specific Hosts
Target specific servers:
```bash
ansible-playbook -i inventory.yml all.yml --limit target_server
```

## 📁 Project Structure

```
ansible-server-setup/
├── README.md                           # This documentation
├── all.yml                            # Main playbook
├── inventory.yml                      # Host inventory and variables
├── ansible.cfg                        # Ansible configuration
├── group_vars/
│   └── all.yml                       # Global variables
├── templates/
│   └── deployment_report.j2          # Deployment report template
└── roles/
    ├── network-setup/                 # Network interface management
    │   ├── README.md                 # Role-specific documentation
    │   ├── defaults/main.yml         # Default variables
    │   ├── vars/main.yml             # Role variables
    │   ├── tasks/main.yml            # Main tasks
    │   ├── handlers/main.yml         # Event handlers
    │   └── meta/main.yml             # Role metadata
    ├── cpu-optimization/              # CPU performance tuning
    │   ├── README.md
    │   ├── defaults/main.yml
    │   ├── vars/main.yml
    │   ├── tasks/main.yml
    │   ├── handlers/main.yml
    │   ├── templates/
    │   │   ├── cpufreq.j2           # Intel optimization script
    │   │   └── amd-cpufreq.j2       # AMD optimization script
    │   └── meta/main.yml
    ├── disk-encryption/               # LUKS disk encryption
    │   ├── README.md
    │   ├── defaults/main.yml
    │   ├── vars/main.yml
    │   ├── tasks/main.yml
    │   ├── handlers/main.yml
    │   ├── templates/
    │   │   └── crypttab.j2          # Crypttab configuration
    │   └── meta/main.yml
    └── system-info/                   # System information gathering
        ├── README.md
        ├── defaults/main.yml
        ├── vars/main.yml
        ├── tasks/main.yml
        ├── handlers/main.yml
        ├── templates/
        │   └── system_report.j2      # System report template
        └── meta/main.yml
```

## 🔒 Security Considerations

### SSH Key Management
- Use dedicated SSH keys for automation
- Store private keys securely with appropriate permissions (600)
- Consider using SSH agent forwarding for enhanced security

### LUKS Encryption
- Use strong passphrases (minimum 12 characters)
- Store encryption keys separately from the playbook
- Consider using key files instead of passphrases for automation

### Network Security
- Ensure SSH access is properly configured
- Use firewall rules to restrict access
- Consider VPN access for remote management

## 🔧 Troubleshooting

### Common Issues

#### SSH Connection Problems
```bash
# Test SSH connectivity
ansible all -i inventory.yml -m ping

# Debug SSH issues
ansible-playbook -i inventory.yml all.yml -vvv
```

#### Permission Denied Errors
- Verify sudo access: `ansible all -i inventory.yml -m shell -a "sudo whoami"`
- Check SSH key permissions: `chmod 600 /path/to/private/key`

#### Package Installation Failures
- Update package cache manually: `apt update`
- Check internet connectivity on target servers
- Verify repository configurations

#### Disk Encryption Issues
- Ensure target disk exists and is not mounted
- Verify sufficient disk space
- Check for existing LUKS headers: `cryptsetup isLuks /dev/device`

#### CPU Optimization Not Applied
- Check if running in virtual environment (limited CPU control)
- Verify cpufreq support: `ls /sys/devices/system/cpu/cpu0/cpufreq/`
- Review kernel parameters in `/proc/cmdline`

### Debug Mode
Enable maximum verbosity for detailed troubleshooting:
```bash
ansible-playbook -i inventory.yml all.yml -vvvv
```

### Log Analysis
Check system logs on target servers:
```bash
# System logs
journalctl -f

# Ansible logs (if configured)
tail -f /var/log/ansible.log
```


### Testing
```bash
# Syntax check
ansible-playbook --syntax-check all.yml

# Dry run
ansible-playbook -i inventory.yml all.yml --check


```

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [LUKS Encryption Guide](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/home)
- [CPU Performance Tuning](https://wiki.archlinux.org/title/CPU_frequency_scaling)
