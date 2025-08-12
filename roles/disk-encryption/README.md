# Disk Encryption Role

## 📋 Description

Enterprise-grade disk encryption role that implements LUKS (Linux Unified Key Setup) encryption for secure data storage on servers. This role provides automated setup of encrypted block devices with secure key management, automatic mounting, and comprehensive validation to ensure data protection compliance.

The role handles the complete encryption lifecycle from initial setup through operational management, ensuring encrypted storage is seamlessly integrated into the server infrastructure.

## 🎯 Features

- **LUKS Encryption**: Industry-standard AES encryption with configurable parameters
- **Automatic Mounting**: Seamless integration with system boot process
- **Key Management**: Secure passphrase handling and key file support
- **Filesystem Creation**: Automated filesystem creation on encrypted devices
- **Recovery Support**: Backup and recovery procedures for encrypted data
- **Validation**: Comprehensive verification of encryption status and functionality

## 🔧 Requirements

### System Requirements
- **OS**: Ubuntu 18.04+, RHEL/CentOS 7+
- **Ansible**: 2.12+
- **Privileges**: Root or sudo access
- **Storage**: Unencrypted block device for encryption

### Hardware Requirements
- **CPU**: AES-NI support recommended for performance
- **Memory**: Sufficient RAM for encryption operations
- **Storage**: Target block device with adequate space

### Dependencies
- `cryptsetup` package for LUKS operations
- Block device available for encryption

## 📦 Installed Packages

The role automatically installs required packages:

```yaml
required_packages:
  - cryptsetup
  - cryptsetup-bin
```

## ⚙️ Variables

### Required Variables

These variables must be defined in your inventory:

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `disk_to_encrypt` | string | Block device path to encrypt | `"/dev/vdb"` |
| `encryption_passphrase` | string | LUKS encryption passphrase | `"secure_password_123"` |

### Encryption Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `luks_cipher` | string | `"aes-xts-plain64"` | Encryption cipher algorithm |
| `luks_key_size` | integer | `512` | Key size in bits |
| `luks_hash` | string | `"sha256"` | Hash algorithm for key derivation |
| `luks_iter_time` | integer | `2000` | Key derivation iteration time (ms) |

### Device and Mounting

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `encrypted_device_name` | string | `"encrypted_disk"` | Name for the encrypted device mapper |
| `mount_point` | string | `"/mnt/encrypted"` | Mount point for encrypted filesystem |
| `filesystem_type` | string | `"ext4"` | Filesystem type to create |
| `mount_options` | string | `"defaults,noatime"` | Mount options for filesystem |

### Example Configuration

```yaml
# In inventory.yml
all:
  hosts:
    server1:
      disk_to_encrypt: "/dev/vdb"
      encryption_passphrase: "MySecurePassword123!"
      encrypted_device_name: "data_disk"
      mount_point: "/opt/encrypted_data"
      filesystem_type: "ext4"
```

## 🚀 Usage

### Execute Role Independently

```bash
# Run disk encryption only
ansible-playbook -i inventory.yml all.yml --tags disk

# With verbose output for debugging
ansible-playbook -i inventory.yml all.yml --tags disk -v

# Dry run to preview changes
ansible-playbook -i inventory.yml all.yml --tags disk --check
```

### Include in Playbook

```yaml
---
- name: Setup Disk Encryption
  hosts: storage_servers
  become: true
  roles:
    - role: disk-encryption
      vars:
        disk_to_encrypt: "/dev/sdb"
        encryption_passphrase: "{{ vault_encryption_password }}"
        mount_point: "/opt/secure_data"
```


## 🔄 Process Flow

### 1. Pre-flight Checks
- Validates target device exists and is accessible
- Checks for existing LUKS encryption
- Identifies processes using the device
- Verifies sufficient disk space

### 2. Device Preparation
- Unmounts any existing filesystems
- Terminates processes using the device
- Ensures device is ready for encryption

### 3. LUKS Encryption Setup
- Creates LUKS header with specified parameters
- Initializes encryption with provided passphrase
- Opens encrypted device for filesystem creation

### 4. Filesystem Creation
- Creates specified filesystem on encrypted device
- Configures optimal filesystem parameters
- Prepares mount point directory

### 5. System Integration
- Adds entry to `/etc/crypttab` for automatic unlocking
- Configures `/etc/fstab` for automatic mounting
- Updates initramfs for boot-time support

### 6. Verification
- Validates encryption status and parameters
- Tests mounting and unmounting operations
- Generates encryption status report

## 📊 Encryption Parameters

### Cipher Algorithms

| Cipher | Security | Performance | Use Case |
|--------|----------|-------------|----------|
| `aes-xts-plain64` | High | Good | General purpose (recommended) |
| `aes-cbc-essiv:sha256` | High | Moderate | Legacy compatibility |
| `serpent-xts-plain64` | Very High | Lower | Maximum security |

### Key Sizes

| Key Size | Security Level | Performance Impact |
|----------|----------------|-------------------|
| 128 bits | Good | Minimal |
| 256 bits | High | Low |
| 512 bits | Very High | Moderate |

### Hash Algorithms

| Algorithm | Security | Speed | Recommendation |
|-----------|----------|-------|----------------|
| `sha256` | High | Fast | General use |
| `sha512` | Very High | Moderate | High security |
| `ripemd160` | Good | Fast | Legacy systems |


## 🚨 Troubleshooting

### Common Issues

#### Device Busy Error
**Problem**: Cannot encrypt device due to active usage
```
TASK [disk-encryption : Create LUKS encryption] ***
fatal: [server]: FAILED! => {"msg": "Device or resource busy"}
```

**Solutions**:
- Check mounted filesystems: `mount | grep /dev/vdb`
- Identify processes using device: `lsof /dev/vdb`
- Force unmount: `umount -f /dev/vdb`
- Kill processes: `fuser -k /dev/vdb`

#### Passphrase Authentication Failed
**Problem**: Cannot unlock LUKS device
```
TASK [disk-encryption : Open LUKS device] ***
fatal: [server]: FAILED! => {"msg": "No key available with this passphrase"}
```

**Solutions**:
- Verify passphrase correctness
- Check LUKS header integrity: `cryptsetup luksDump /dev/vdb`
- Try alternative key slots: `cryptsetup luksOpen --key-slot 1 /dev/vdb`
- Use recovery procedures if available

#### Mount Point Creation Failed
**Problem**: Cannot create mount point directory
```
TASK [disk-encryption : Create mount point] ***
fatal: [server]: FAILED! => {"msg": "Permission denied"}
```

**Solutions**:
- Check parent directory permissions
- Verify sufficient disk space: `df -h`
- Ensure proper sudo privileges
- Check SELinux context if applicable

#### Initramfs Update Failed
**Problem**: Cannot update initramfs for boot support
```
TASK [disk-encryption : Update initramfs] ***
fatal: [server]: FAILED! => {"msg": "update-initramfs failed"}
```

**Solutions**:
- Check available disk space in `/boot`
- Verify initramfs tools installation
- Manual update: `update-initramfs -u`
- Check for kernel module dependencies

## 📚 Additional Resources

- [LUKS Documentation](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/home)
- [Linux Disk Encryption](https://wiki.archlinux.org/title/Dm-crypt)
- [Cryptsetup Manual](https://man7.org/linux/man-pages/man8/cryptsetup.8.html)
- [NIST Encryption Guidelines](https://csrc.nist.gov/publications/detail/sp/800-111/final)

## 🏷️ Tags

Available tags for selective execution:

- `disk-encryption`: All disk encryption tasks
- `preflight`: Pre-flight validation checks
- `install`: Package installation
- `check`: Device and encryption status checks
- `unmount`: Device unmounting operations
- `format`: LUKS formatting operations
- `open`: Device opening operations
- `filesystem`: Filesystem creation
- `mount`: Mounting operations
- `crypttab`: Crypttab configuration
- `fstab`: Fstab configuration
- `verify`: Verification and validation
- `info`: Information display
- `error`: Error handling tasks

### Tag Usage Examples

```bash
# Install packages and run checks only
ansible-playbook -i inventory.yml all.yml --tags "disk-encryption,install,check"

# Format and setup encryption only
ansible-playbook -i inventory.yml all.yml --tags "disk-encryption,format,filesystem"

# Mount and configure system integration
ansible-playbook -i inventory.yml all.yml --tags "disk-encryption,mount,crypttab,fstab"

# Verification only
ansible-playbook -i inventory.yml all.yml --tags "disk-encryption,verify"
```

## ⚠️ Important Warnings

### Data Loss Prevention
- **Backup Data**: Always backup important data before encryption
- **Test Procedures**: Test encryption and recovery in non-production environments
- **Header Backup**: Always backup LUKS headers before making changes
- **Passphrase Security**: Store passphrases securely and have recovery procedures

### Operational Considerations
- **Boot Dependencies**: Encrypted root filesystems require special configuration
- **Network Dependencies**: Network-mounted encrypted devices need careful planning
- **Recovery Planning**: Ensure recovery procedures are documented and tested
- **Key Management**: Implement proper key lifecycle management
