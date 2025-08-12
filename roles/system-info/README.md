# System Information Role

## 📋 Description

Comprehensive system information gathering and analysis role designed for enterprise infrastructure monitoring and optimization. This role collects detailed hardware, software, and performance data from target servers, generating comprehensive reports for infrastructure assessment, compliance auditing, and performance optimization.

The role provides intelligent analysis of CPU capabilities, memory configuration, storage systems, network interfaces, security features, and virtualization status to support informed decision-making in enterprise environments.

## 🎯 Features

- **Hardware Analysis**: Detailed CPU, memory, and storage information
- **Performance Monitoring**: CPU frequencies, governors, and optimization status
- **Security Assessment**: Vulnerability status and security mitigations
- **Network Discovery**: Interface configuration and network capabilities
- **Virtualization Detection**: Hypervisor and container environment identification
- **Vendor-Specific Analysis**: Intel Hyper-Threading and AMD SMT detection
- **Customizable Reporting**: Flexible output formats and detail levels

## 🔧 Requirements

### System Requirements
- **OS**: Ubuntu 18.04+, RHEL/CentOS 7+, Debian 9+
- **Ansible**: 2.12+
- **Privileges**: Root or sudo access for detailed hardware information
- **Python**: 3.6+ for advanced fact gathering

### Dependencies
- Standard Linux utilities (usually pre-installed)
- Hardware detection tools for detailed analysis
- Network utilities for interface information

## 📦 Installed Packages

The role automatically installs required packages for comprehensive analysis:

```yaml
required_packages:
  - dmidecode         # DMI/SMBIOS information
  - pciutils          # PCI device information
  - usbutils          # USB device information
```

## ⚙️ Variables

### Information Gathering Control

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `gather_cpu_info` | boolean | `true` | Collect CPU information and capabilities |
| `gather_memory_info` | boolean | `true` | Gather memory configuration and usage |
| `gather_disk_info` | boolean | `true` | Analyze storage devices and filesystems |
| `gather_network_info` | boolean | `true` | Collect network interface information |
| `gather_kernel_info` | boolean | `true` | Gather kernel and module information |
| `gather_virtualization_info` | boolean | `true` | Detect virtualization environment |

### CPU Analysis Settings

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `check_hyperthreading` | boolean | `true` | Analyze Intel Hyper-Threading status |
| `check_intel_features` | boolean | `true` | Check Intel-specific CPU features |
| `check_amd_smt` | boolean | `true` | Analyze AMD SMT (Simultaneous Multi-Threading) |
| `check_cpu_frequency` | boolean | `true` | Monitor CPU frequency information |
| `check_cpu_governor` | boolean | `true` | Check CPU frequency governor |

### Security Settings

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `check_security_features` | boolean | `true` | Analyze security mitigations and vulnerabilities |

### Hardware Analysis

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `check_hardware_info` | boolean | `true` | Collect detailed hardware information |
| `check_bios_info` | boolean | `true` | Gather BIOS/UEFI information |
| `check_pci_devices` | boolean | `false` | List PCI devices (can be verbose) |

### Report Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `detailed_report` | boolean | `true` | Generate detailed analysis reports |
| `display_summary` | boolean | `true` | Show system summary information |
| `display_detailed_info` | boolean | `true` | Display comprehensive system details |
| `save_to_file` | boolean | `false` | Save reports to files |
| `report_format` | string | `"text"` | Report format: text, json, yaml |
| `include_ansible_facts` | boolean | `false` | Include raw Ansible facts |

## 🚀 Usage

### Execute Role Independently

```bash
# Run system information gathering only
ansible-playbook -i inventory.yml all.yml --tags info

# With verbose output for debugging
ansible-playbook -i inventory.yml all.yml --tags info -v

# Quick summary only
ansible-playbook -i inventory.yml all.yml --tags "info,summary"
```

### Include in Playbook

```yaml
---
- name: Gather System Information
  hosts: all_servers
  become: true
  roles:
    - role: system-info
      vars:
        detailed_report: true
        check_security_features: true
        save_to_file: true
```


## 🔄 Process Flow

### 1. Package Installation
- Installs required system analysis tools
- Verifies tool availability and versions
- Prepares environment for information gathering

### 2. System Discovery
- Gathers comprehensive Ansible facts
- Detects hardware architecture and capabilities
- Identifies operating system and kernel version

### 3. CPU Analysis
- Detects CPU vendor (Intel/AMD)
- Analyzes CPU features and capabilities
- Checks frequency scaling and governor status
- Evaluates Hyper-Threading/SMT configuration

### 4. Hardware Assessment
- Collects memory configuration and usage
- Analyzes storage devices and filesystems
- Gathers network interface information
- Identifies PCI and USB devices (if enabled)

### 5. Security Evaluation
- Checks CPU vulnerability mitigations
- Analyzes security features and configurations
- Evaluates system hardening status

### 6. Report Generation
- Compiles comprehensive system report
- Formats output according to configuration
- Displays or saves results as specified

## 📊 CPU Analysis Features

### Intel CPU Analysis
```yaml
# Intel-specific checks
intel_commands:
  ht_check: "lscpu | grep -i 'thread(s) per core' | awk '{print $4}'"
  logical_cpus: "nproc"
  physical_cpus: "lscpu | grep '^CPU(s):' | awk '{print $2}'"
  threads_per_core: "lscpu | grep 'Thread(s) per core' | awk '{print $4}'"
```

### AMD CPU Analysis
```yaml
# AMD-specific checks
amd_commands:
  logical_cpus: "nproc"
  physical_cpus: "lscpu | grep '^CPU(s):' | awk '{print $2}'"
  smt_status: "cat /sys/devices/system/cpu/smt/active"
```

### CPU Frequency Monitoring
- Current CPU frequencies across all cores
- Available frequency governors
- Frequency scaling capabilities
- Turbo Boost/SMT status

## 🔍 Security Analysis

### Vulnerability Assessment
The role checks for common CPU vulnerabilities:

- **Spectre v1**: Bounds check bypass
- **Spectre v2**: Branch target injection
- **Meltdown**: Rogue data cache load
- **L1TF**: L1 Terminal Fault
- **MDS**: Microarchitectural Data Sampling


## 📋 Report Examples

### System Summary Report
```
=== SYSTEM SUMMARY ===
Hostname: production-server-01
OS: Ubuntu 20.04.3 LTS
Kernel: 5.4.0-91-generic
Architecture: x86_64
CPU: Intel(R) Xeon(R) CPU E5-2686 v4 @ 2.30GHz
CPU Cores: 8
CPU VCPUs: 16
Memory: 32768 MB
Virtualization: kvm
```

### CPU Analysis Report
```
=== CPU ANALYSIS ===
Vendor: Intel
Model: Xeon E5-2686 v4
Cores: 8 physical, 16 logical
Hyper-Threading: ENABLED
Current Governor: performance
Turbo Boost: ENABLED
Base Frequency: 2.30 GHz
Max Frequency: 3.00 GHz
```

### Security Assessment Report
```
=== SECURITY ASSESSMENT ===
Spectre v1: Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Spectre v2: Mitigation: Full generic retpoline, IBPB: conditional, IBRS_FW, STIBP: conditional, RSB filling
Meltdown: Mitigation: PTI
L1TF: Mitigation: PTE Inversion; VMX: conditional cache flushes, SMT vulnerable
```

## 🚨 Troubleshooting

### Common Issues

#### Missing Hardware Information
**Problem**: Limited hardware details available
```
TASK [system-info : Get hardware information] ***
skipping: [server] => {"msg": "dmidecode not available"}
```

**Solutions**:
- Install dmidecode: `apt install dmidecode`
- Check virtualization limitations
- Verify sudo privileges for hardware access
- Use alternative tools: `lshw`, `hwinfo`

#### CPU Frequency Information Unavailable
**Problem**: Cannot read CPU frequency scaling information
```
TASK [system-info : Get CPU governor] ***
fatal: [server]: FAILED! => {"msg": "No such file or directory"}
```

**Solutions**:
- Check if running in virtual environment
- Verify cpufreq support: `ls /sys/devices/system/cpu/cpu0/cpufreq/`
- Load cpufreq modules: `modprobe cpufreq_stats`
- Check kernel configuration

#### Security Information Missing
**Problem**: Vulnerability information not available
```
TASK [system-info : Check CPU vulnerabilities] ***
skipping: [server] => {"msg": "Vulnerability files not found"}
```

**Solutions**:
- Update kernel to recent version
- Check kernel configuration for vulnerability reporting
- Verify sysfs mount: `mount | grep sysfs`
- Use alternative security tools

#### Permission Denied Errors
**Problem**: Cannot access system information
```
TASK [system-info : Get BIOS information] ***
fatal: [server]: FAILED! => {"msg": "Permission denied"}
```

**Solutions**:
- Ensure sudo privileges: `ansible all -m shell -a "sudo dmidecode"`
- Check file permissions: `ls -la /dev/mem`
- Verify ansible_become configuration
- Use alternative information sources




## 📚 Additional Resources

- [Linux Hardware Information](https://www.cyberciti.biz/faq/linux-list-hardware-information/)
- [CPU Vulnerability Mitigations](https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/index.html)

## 🏷️ Tags

Available tags for selective execution:

- `system-info`: All system information tasks
- `install`: Package installation
- `facts`: Ansible fact gathering
- `cpu`: CPU information and analysis
- `memory`: Memory information
- `disk`: Storage information
- `network`: Network interface information
- `kernel`: Kernel and module information
- `hardware`: Hardware detection
- `bios`: BIOS/UEFI information
- `pci`: PCI device information
- `security`: Security assessment
- `intel`: Intel-specific analysis
- `amd`: AMD-specific analysis
- `hyperthreading`: Hyper-Threading analysis
- `smt`: SMT analysis
- `virtualization`: Virtualization detection
- `summary`: System summary display
- `report`: Report generation

### Tag Usage Examples

```bash
# CPU and memory information only
ansible-playbook -i inventory.yml all.yml --tags "system-info,cpu,memory"

# Security assessment only
ansible-playbook -i inventory.yml all.yml --tags "system-info,security"

# Hardware inventory
ansible-playbook -i inventory.yml all.yml --tags "system-info,hardware,bios,pci"

# Quick summary
ansible-playbook -i inventory.yml all.yml --tags "system-info,summary"

# Intel-specific analysis
ansible-playbook -i inventory.yml all.yml --tags "system-info,intel,hyperthreading"
```

## 📊 Output Formats

### Text Format (Default)
Human-readable text output with structured sections and clear formatting.

### JSON Format
```yaml
report_format: "json"
```
Machine-readable JSON output suitable for integration with other tools.

### YAML Format
```yaml
report_format: "yaml"
```
YAML output format for configuration management integration.

### File Output
```yaml
save_to_file: true
report_file_path: "/tmp/system_report_{{ inventory_hostname }}.txt"
```

## ⚠️ Important Notes

### Virtual Environment Limitations
- Some hardware information may be limited in VMs
- CPU frequency scaling might not be available
- Hardware-specific features may be virtualized
- Security features might be managed by hypervisor
