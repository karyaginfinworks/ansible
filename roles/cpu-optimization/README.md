# CPU Optimization Role

## 📋 Description

Advanced CPU performance optimization role designed for enterprise servers running high-performance applications. This role automatically detects CPU architecture (Intel/AMD) and applies vendor-specific optimizations to maximize computational performance while maintaining system stability.

The role configures CPU governors, frequency scaling, power management features, and system-level optimizations to achieve optimal performance for compute-intensive workloads.

## 🎯 Features

- **Multi-Vendor Support**: Automatic Intel and AMD CPU detection and optimization
- **Performance Governors**: Configures CPU frequency scaling for maximum performance
- **Turbo Boost/SMT**: Enables Intel Turbo Boost and AMD SMT technologies
- **Power Management**: Optimizes C-states and power management settings
- **I/O Scheduling**: Configures optimal disk I/O schedulers
- **GRUB Integration**: Applies kernel-level performance parameters
- **Verification**: Comprehensive validation of applied optimizations

## 🔧 Requirements

### System Requirements
- **OS**: Ubuntu 18.04+, RHEL/CentOS 7+
- **Ansible**: 2.12+
- **Privileges**: Root or sudo access
- **CPU**: Intel or AMD processor with frequency scaling support

### Hardware Support
- **Intel**: Core, Xeon, and Atom processors with P-State support
- **AMD**: Ryzen, EPYC, and Threadripper processors with boost support
- **Virtualization**: Limited functionality in virtual environments

## 📦 Installed Packages

The role automatically installs required packages based on CPU vendor:

### Common Packages
```yaml
required_packages:
  - cpufrequtils
  - linux-tools-common
  - linux-tools-generic
```

### Intel-Specific Packages
```yaml
intel_packages:
  - intel-microcode
```

### AMD-Specific Packages
```yaml
amd_packages:
  - amd64-microcode
```

## ⚙️ Variables

### Performance Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `cpu_governor` | string | `"performance"` | CPU frequency governor |
| `cpu_governor_fallback` | string | `"ondemand"` | Fallback governor if performance unavailable |
| `enable_turbo_boost` | boolean | `true` | Enable Intel Turbo Boost/AMD Boost |
| `disable_c_states` | boolean | `false` | Disable CPU C-states for lower latency |

### Frequency Management

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `cpu_min_freq` | string | `"auto"` | Minimum CPU frequency (Hz or "auto") |
| `cpu_max_freq` | string | `"auto"` | Maximum CPU frequency (Hz or "auto") |

### System Optimization

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `scheduler_policy` | string | `"mq-deadline"` | I/O scheduler policy |
| `enable_numa_balancing` | boolean | `true` | Enable NUMA memory balancing |

### GRUB Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `grub_config_path` | string | `"/etc/default/grub"` | Path to GRUB configuration |
| `grub_cmdline_params` | list | See below | Kernel parameters for performance |



## 🚀 Usage

### Execute Role Independently

```bash
# Run CPU optimization only
ansible-playbook -i inventory.yml all.yml --tags cpu

# With verbose output for debugging
ansible-playbook -i inventory.yml all.yml --tags cpu -v

# Dry run to preview changes
ansible-playbook -i inventory.yml all.yml --tags cpu --check
```

### Include in Playbook

```yaml
---
- name: Optimize CPU Performance
  hosts: compute_servers
  become: true
  roles:
    - role: cpu-optimization
      vars:
        cpu_governor: "performance"
        enable_turbo_boost: true
        disable_c_states: true
```



## 🔄 Process Flow

### 1. Discovery Phase
- Detects CPU vendor (Intel/AMD)
- Gathers CPU information and capabilities
- Checks available frequency governors
- Identifies current performance settings

### 2. Package Installation
- Installs vendor-specific packages
- Updates microcode if available
- Installs performance monitoring tools

### 3. Governor Configuration
- Sets optimal CPU frequency governor
- Configures frequency scaling parameters
- Applies vendor-specific optimizations

### 4. Power Management
- Configures Intel P-State or AMD P-State
- Enables/disables Turbo Boost/SMT
- Optimizes C-state configuration

### 5. System Optimization
- Configures I/O schedulers
- Updates GRUB kernel parameters
- Applies system-level performance settings

### 6. Verification
- Validates applied configurations
- Checks CPU frequencies and governors
- Generates performance report

## 📊 Intel-Specific Optimizations

### P-State Driver Configuration
```yaml
# Intel P-State paths
intel_pstate_path: "/sys/devices/system/cpu/intel_pstate"
intel_turbo_path: "/sys/devices/system/cpu/intel_pstate/no_turbo"
intel_max_perf_path: "/sys/devices/system/cpu/intel_pstate/max_perf_pct"
intel_min_perf_path: "/sys/devices/system/cpu/intel_pstate/min_perf_pct"
```

### Turbo Boost Management
- Enables Intel Turbo Boost technology
- Configures maximum performance percentage
- Optimizes thermal management

### Hyper-Threading Optimization
- Maintains Hyper-Threading enabled for most workloads
- Provides options for HT-sensitive applications

## 📊 AMD-Specific Optimizations

### P-State Driver Configuration
```yaml
# AMD P-State paths
amd_pstate_path: "/sys/devices/system/cpu/amd_pstate"
amd_boost_path: "/sys/devices/system/cpu/cpufreq/boost"
```

### SMT (Simultaneous Multi-Threading)
- Enables AMD SMT for improved throughput
- Configures boost frequencies
- Optimizes scheduler policies

### Energy Efficiency
- Disables energy-aware scheduling when needed
- Configures performance-focused power management


## 🚨 Troubleshooting

### Common Issues

#### CPU Governor Not Applied
**Problem**: Performance governor not set
```
TASK [cpu-optimization : Set CPU governor] ***
fatal: [server]: FAILED! => {"msg": "Permission denied"}
```

**Solutions**:
- Check cpufreq support: `ls /sys/devices/system/cpu/cpu0/cpufreq/`
- Verify running on physical hardware (VMs have limited support)
- Ensure proper kernel modules: `modprobe cpufreq_performance`

#### Intel P-State Not Available
**Problem**: Intel P-State driver not loaded
```
TASK [cpu-optimization : Configure Intel P-State] ***
skipping: [server] => {"msg": "Intel P-State not available"}
```

**Solutions**:
- Check CPU support: `grep -i pstate /proc/cpuinfo`
- Verify kernel parameters: `cat /proc/cmdline | grep intel_pstate`
- Load P-State driver: `modprobe intel_pstate`

#### Turbo Boost Not Working
**Problem**: Turbo frequencies not achieved
```
Current CPU frequency lower than expected
```

**Solutions**:
- Check thermal throttling: `sensors` or `cat /proc/cpuinfo`
- Verify turbo is enabled: `cat /sys/devices/system/cpu/intel_pstate/no_turbo`
- Check power limits: `turbostat` (if available)

#### GRUB Update Fails
**Problem**: GRUB configuration update fails
```
TASK [cpu-optimization : Update GRUB] ***
fatal: [server]: FAILED! => {"msg": "update-grub failed"}
```

**Solutions**:
- Check GRUB installation: `grub-install --version`
- Verify GRUB configuration syntax: `grub-mkconfig -o /dev/null`
- Manual update: `update-grub` or `grub2-mkconfig -o /boot/grub2/grub.cfg`

## 📚 Additional Resources

- [Intel P-State Driver Documentation](https://www.kernel.org/doc/html/latest/admin-guide/pm/intel_pstate.html)
- [AMD P-State Driver Documentation](https://www.kernel.org/doc/html/latest/admin-guide/pm/amd-pstate.html)
- [CPU Frequency Scaling](https://wiki.archlinux.org/title/CPU_frequency_scaling)
- [Linux Performance Tuning](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/monitoring_and_managing_system_status_and_performance/)

## 🏷️ Tags

Available tags for selective execution:

- `cpu-optimization`: All CPU optimization tasks
- `install`: Package installation
- `info`: CPU information gathering
- `detect`: CPU vendor detection
- `governor`: CPU governor configuration
- `intel`: Intel-specific optimizations
- `amd`: AMD-specific optimizations
- `turbo`: Turbo Boost/SMT configuration
- `freq`: Frequency scaling configuration
- `scheduler`: I/O scheduler configuration
- `grub`: GRUB configuration updates
- `performance`: Performance mode settings
- `verify`: Verification and validation

### Tag Usage Examples

```bash
# Install packages and detect CPU only
ansible-playbook -i inventory.yml all.yml --tags "cpu-optimization,install,detect"

# Apply Intel optimizations only
ansible-playbook -i inventory.yml all.yml --tags "cpu-optimization,intel"

# Configure governors without GRUB changes
ansible-playbook -i inventory.yml all.yml --tags "cpu-optimization,governor" --skip-tags grub

# Verification only
ansible-playbook -i inventory.yml all.yml --tags "cpu-optimization,verify"
```

## ⚠️ Important Notes

### Virtual Environments
- Limited functionality in VMs due to hypervisor restrictions
- Some optimizations may not apply in containerized environments
- Cloud instances may have vendor-specific limitations

### Reboot Requirements
- GRUB parameter changes require system reboot
- Some optimizations take effect immediately
- Plan maintenance windows for GRUB updates
