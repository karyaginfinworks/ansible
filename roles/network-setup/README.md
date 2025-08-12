# Network Setup Role

## 📋 Description

This role provides comprehensive network interface management functionality for Ubuntu systems using netplan. It automates the process of renaming network interfaces to ensure consistent naming across deployments, which is essential for automation and infrastructure management.

The role intelligently detects the current network configuration, performs safe interface renaming when needed, and validates connectivity after changes to ensure network stability.

## 🎯 Features

- **Intelligent Interface Detection**: Automatically identifies active network interfaces
- **Safe Renaming Process**: Only renames interfaces when necessary
- **Netplan Integration**: Uses Ubuntu's native netplan for configuration
- **Connectivity Validation**: Tests network connectivity after changes
- **Rollback Safety**: Preserves original configuration with backup
- **Detailed Reporting**: Provides comprehensive status reports

## 🔧 Requirements

### System Requirements
- **OS**: Ubuntu 18.04+ with netplan support
- **Ansible**: 2.12+
- **Privileges**: Root or sudo access
- **Network**: Active network connection during execution

### Dependencies
- `netplan.io` package (usually pre-installed on Ubuntu)
- `iproute2` package for network utilities
- Active network interface for connectivity testing

## 📦 Installed Packages

The role automatically installs required packages:

```yaml
required_packages:
  - net-tools
  - iproute2

ubuntu_packages:
  - netplan.io
```

## ⚙️ Variables

### Required Variables

These variables must be defined in your inventory or playbook:

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `vm_if_name` | string | Target interface name | `"net0"` |

### Optional Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `connectivity_test_host` | string | `"8.8.8.8"` | Host to ping for connectivity testing |

### Example Configuration

```yaml
# In inventory.yml
all:
  hosts:
    server1:
      vm_if_name: "net0"
      connectivity_test_host: "1.1.1.1"
```

## 🚀 Usage

### Execute Role Independently

```bash
# Run only network setup
ansible-playbook -i inventory.yml all.yml --tags network

# With verbose output
ansible-playbook -i inventory.yml all.yml --tags network -v

# Dry run to check what would change
ansible-playbook -i inventory.yml all.yml --tags network --check
```

### Include in Playbook

```yaml
---
- name: Configure Network
  hosts: target_servers
  become: true
  roles:
    - role: network-setup
      vars:
        vm_if_name: "eth0"
        connectivity_test_host: "8.8.8.8"
```

## 🔄 Process Flow

The role follows a systematic approach to ensure safe network configuration:

### 1. Discovery Phase
- Detects current active network interface
- Identifies existing netplan configuration files
- Determines if renaming is necessary

### 2. Validation Phase
- Checks if target interface name is already in use
- Validates netplan configuration syntax
- Ensures network connectivity before changes

### 3. Configuration Phase
- Updates netplan configuration with new interface name
- Applies netplan changes
- Waits for network stabilization

### 4. Verification Phase
- Tests network connectivity
- Verifies interface rename success
- Generates detailed status report

## 📊 Task Breakdown

### Package Installation
```yaml
- name: "Network Setup | Install required packages"
  package:
    name: "{{ required_packages }}"
    state: present
```

### Interface Discovery
```yaml
- name: "Network Setup | Get current active network interface"
  shell: "ip route | grep default | awk '{print $5}' | head -1"
  register: current_interface
```

### Netplan Configuration
```yaml
- name: "Network Setup | Update set-name field to desired interface name"
  replace:
    path: "{{ target_netplan_file }}"
    regexp: 'set-name:\s*"?[^"\n]*"?'
    replace: 'set-name: "{{ vm_if_name }}"'
```

### Connectivity Testing
```yaml
- name: "Network Setup | Test network connectivity"
  command: "ping -c 3 -W 5 {{ connectivity_test_host }}"
  register: connectivity_test
```

## 🛡️ Safety Features

### Backup and Recovery
- Automatic backup of original netplan configuration
- Rollback capability in case of configuration errors
- Preservation of existing network settings

### Validation Checks
- Pre-flight validation of target interface name
- Syntax checking of netplan configuration
- Connectivity verification before and after changes

### Error Handling
- Graceful handling of missing netplan files
- Safe failure modes that preserve connectivity
- Detailed error reporting for troubleshooting

## 📋 Example Netplan Configuration

### Before Renaming
```yaml
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: true
```

### After Renaming
```yaml
network:
  version: 2
  ethernets:
    ens3:
      set-name: "net0"
      dhcp4: true
```


## 🚨 Troubleshooting

### Common Issues

#### Interface Not Found
**Problem**: Current interface cannot be detected
```
TASK [network-setup : Get current active network interface] ***
fatal: [server]: FAILED! => {"msg": "No default route found"}
```

**Solution**:
- Verify network connectivity: `ip route show`
- Check interface status: `ip link show`
- Ensure DHCP or static configuration is active

#### Netplan Apply Fails
**Problem**: Netplan configuration cannot be applied
```
TASK [network-setup : Apply netplan configuration] ***
fatal: [server]: FAILED! => {"rc": 1, "msg": "netplan apply failed"}
```

**Solution**:
- Check netplan syntax: `netplan try`
- Validate YAML format in netplan files
- Review systemd-networkd logs: `journalctl -u systemd-networkd`

#### Connectivity Test Fails
**Problem**: Network connectivity lost after rename
```
TASK [network-setup : Test network connectivity] ***
fatal: [server]: FAILED! => {"rc": 1, "msg": "ping failed"}
```

**Solution**:
- Check interface status: `ip addr show`
- Verify routing table: `ip route show`
- Restart networking: `systemctl restart systemd-networkd`

#### Permission Denied
**Problem**: Insufficient privileges for network configuration
```
TASK [network-setup : Update netplan configuration] ***
fatal: [server]: FAILED! => {"msg": "Permission denied"}
```

**Solution**:
- Ensure sudo access: `ansible all -m shell -a "sudo whoami"`
- Check file permissions: `ls -la /etc/netplan/`
- Verify ansible_become is set to true

### Debug Commands

```bash
# Check current network configuration
ip addr show
ip route show
netplan get

# Validate netplan configuration
netplan try --timeout 30

# Check systemd-networkd status
systemctl status systemd-networkd
journalctl -u systemd-networkd -f

# Test connectivity manually
ping -c 3 8.8.8.8
```



## 📚 Additional Resources

- [Netplan Documentation](https://netplan.io/reference/)
- [Ubuntu Network Configuration](https://ubuntu.com/server/docs/network-configuration)
- [systemd-networkd Manual](https://www.freedesktop.org/software/systemd/man/systemd-networkd.html)
- [Network Interface Naming](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)

## 🏷️ Tags

Available tags for selective execution:

- `network-setup`: All network setup tasks
- `install`: Package installation only
- `discovery`: Interface discovery tasks
- `check`: Validation and checking tasks
- `info`: Information display tasks
- `skip`: Tasks that can be skipped
- `apply`: Configuration application tasks
- `connectivity`: Connectivity testing tasks
- `verify`: Verification tasks
- `report`: Reporting tasks

### Tag Usage Examples

```bash
# Install packages only
ansible-playbook -i inventory.yml all.yml --tags "network-setup,install"

# Discovery and validation only
ansible-playbook -i inventory.yml all.yml --tags "network-setup,discovery,check"

# Skip connectivity testing
ansible-playbook -i inventory.yml all.yml --tags network-setup --skip-tags connectivity
