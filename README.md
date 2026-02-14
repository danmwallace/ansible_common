# Ansible Role: ansible_common

Baseline system configuration for homelab servers and workstations. This role handles initial server setup including hostname configuration, user management, Docker installation, and base package installation.

## Description

This role provides a common baseline configuration that should be applied to all systems before deploying more specialized roles. It supports both Debian-based (Ubuntu, Debian) and RedHat-based (Fedora) systems, including Fedora atomic variants (IoT/CoreOS).

## Requirements

- Ansible 2.15+
- Target OS: Ubuntu 20.04+, Debian 11+, Fedora 38+
- Sudo/root access on target systems

## Role Variables

### System Configuration

```yaml
# Timezone setting
common_timezone: "America/New_York"

# DNS resolver
common_dns_resolver: "1.1.1.2"
```

### Package Configuration

```yaml
# Packages for Debian/Ubuntu systems
common_packages_debian:
  - apt-transport-https
  - ca-certificates
  - curl
  - git
  - python3
  - python3-venv

# Packages for Fedora systems
common_packages_fedora:
  - ca-certificates
  - certbot
  - curl
  - git
  - python3
  - python3-virtualenv
```

### Docker Configuration

```yaml
# Docker Compose download URI
# Update to latest version from: https://github.com/docker/compose/releases
common_docker_compose_uri: "https://github.com/docker/compose/releases/download/v2.29.7/docker-compose-linux-x86_64"
```

### User Configuration

```yaml
# Users to create on the system
# Override in group_vars with actual users
common_users:
  - name: username
    ssh_key: "ssh-ed25519 AAAAC3N..."
```

### Certificate Paths

```yaml
# SSL certificate paths
common_cert_path: "/etc/letsencrypt/live/{{ host }}"
common_cockpit_cert_dir: "/etc/cockpit/ws-certs.d"
```

## Dependencies

None.

## Example Playbook

### Basic Usage

```yaml
---
- name: Configure baseline system
  hosts: all
  become: true
  roles:
    - ansible_common
```

### With Group Variables (Recommended)

```yaml
# inventories/production/group_vars/all/vars.yml
common_timezone: "America/Los_Angeles"
common_dns_resolver: "8.8.8.8"
common_packages_debian:
  - apt-transport-https
  - ca-certificates
  - curl
  - git
  - htop
  - vim

common_users:
  - name: adminuser
    ssh_key: "ssh-ed25519 AAAAC3N..."
  - name: deploy
    ssh_key: "ssh-ed25519 AAAAC3N..."

# playbooks/baseline.yml
- name: Configure baseline
  hosts: all
  become: true
  roles:
    - ansible_common
```

## What This Role Does

1. **Hostname Configuration**: Sets system hostname from inventory
2. **Package Installation**: Installs OS-specific base packages
3. **QEMU Guest Agent**: Installs guest agent on QEMU VMs
4. **Timezone Configuration**: Sets system timezone
5. **Docker Installation**:
   - Adds Docker repository (Ubuntu/Debian)
   - Installs Docker CE
   - Downloads and installs Docker Compose
   - Creates docker group
6. **User Management**:
   - Creates specified users
   - Adds users to sudo/wheel group
   - Configures SSH authorized keys
   - Sets up passwordless sudo for ansible user

## OS-Specific Behavior

### Debian/Ubuntu
- Uses `apt` package manager
- Installs from Docker's official repository
- Adds users to `sudo` group

### Fedora Standard
- Uses `dnf` package manager
- Installs Docker from Docker's repository
- Adds users to `wheel` group

### Fedora Atomic (IoT/CoreOS)
- Uses `rpm-ostree` for package management
- Installs Docker as layered package
- Handles atomic OS constraints

## Multi-OS Support

The role automatically detects the OS using Ansible facts:

```yaml
# Example conditional from tasks
when: ansible_facts['os_family'] == "Debian"
when: ansible_facts['os_family'] == "RedHat" and ansible_facts['distribution'] == "Fedora"
```

## Tags

This role supports the following tags:

- No role-specific tags (part of base configuration)

## Testing

```bash
# Syntax check
ansible-playbook --syntax-check playbooks/common.yml

# Check mode (dry-run)
ansible-playbook --check -i inventories/lab/inventory playbooks/common.yml

# Run against specific hosts
ansible-playbook -i inventories/lab/inventory playbooks/common.yml --limit test-server
```

## Security Considerations

- Never include actual user credentials in `defaults/main.yml`
- Store user definitions in inventory `group_vars`
- SSH keys should be unique per user
- Ansible user gets passwordless sudo for automation

## Common Issues

### Docker Installation Failures

If Docker installation fails, verify:
- Target system has internet access
- Docker repository is accessible
- No conflicting Docker versions installed

### User Creation Failures

If user creation fails:
- Verify SSH keys are in correct format
- Check that usernames are valid
- Ensure sudo/wheel group exists

## License

MIT

## Author

Dan Wallace

## See Also

- [Role Development Guide](../../ansible-homelab-cfg/Documentation/Role-Development.md)
- [Variable Hierarchy](../../ansible-homelab-cfg/Documentation/Variable-Hierarchy.md)
