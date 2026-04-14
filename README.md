## Lab Environment

- EVE-NG (virtual lab)
- Cisco IOS switches (sw01, sw02, sw03)
- Ansible for automation
- GitHub Actions for CI/CD
- Tailscale for secure private connectivity

---

## Project Structure
network-ansible/
├── inventories/
│ └── lab/
│ ├── hosts.yml
│ ├── group_vars/
│ │ ├── all.yml
│ │ ├── cisco_ios.yml
│ │ └── access_switches.yml
│ └── host_vars/
│ ├── sw01.yml
│ ├── sw02.yml
│ └── sw03.yml
├── playbooks/
├── roles/
│ ├── cisco_base/
│ └── access_switch/
├── .github/
│ └── workflows/
├── ansible.cfg
└── README.md


---

## Capabilities

### Base Device Provisioning
- Hostname and system identity
- Management interface (SVI)
- Default gateway configuration
- DNS, NTP, and syslog integration
- SSH enablement and secure access
- Local user provisioning

### Access Layer Configuration
- VLAN provisioning (MGMT, USERS, VOICE)
- Interface configuration
- Management SVI setup

### Security Hardening
- Identification and shutdown of unused interfaces
- Standardized interface descriptions

### Operational Workflow
- Pre-deployment validation
- Configuration deployment
- Post-deployment verification
