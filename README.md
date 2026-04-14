## Lab Environment

- EVE-NG (virtual lab)
- Cisco IOS switches (sw01, sw02, sw03)
- Ansible for automation
- GitHub Actions for CI/CD
- Tailscale for secure private connectivity

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


## Features Implemented

### Day-0 Configuration
- Hostname
- Management IP (SVI)
- Default gateway
- DNS, NTP, syslog
- SSH configuration
- Local user

### Access Switch Configuration
- VLAN creation (MGMT, USERS, VOICE)
- SVI configuration
- Interface setup

### Security Automation
- Disable unused interfaces
- Interface descriptions
- Shutdown unused ports

### Operational Workflow
- Pre-check
- Deploy
- Post-check
