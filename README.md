# Network Automation Lab with Ansible + CI/CD

## 📌 Overview
This project demonstrates end-to-end network automation using Ansible, integrated with CI/CD pipelines and secure access via Tailscale.

The lab simulates a real-world enterprise workflow:
Git → CI/CD → Secure Access → Automation → Network Devices

---

## 🧱 Lab Environment

- EVE-NG (virtual lab)
- Cisco IOS switches (sw01, sw02, sw03)
- Ansible for automation
- GitHub Actions for CI/CD
- Tailscale for secure private connectivity

---

## 📂 Project Structure
network-ansible/
├── inventories/
│ └── lab/
│ ├── hosts.yml
│ ├── group_vars/
│ └── host_vars/
├── playbooks/
├── roles/
│ ├── cisco_base/
│ └── access_switch/
├── .github/workflows/
└── ansible.cfg


---

## ⚙️ Features Implemented

### 🔹 Day-0 Configuration
- Hostname
- Management IP (SVI)
- Default Gateway
- DNS
- NTP
- Syslog
- SSH configuration
- Local user creation

---

### 🔹 Access Switch Configuration
- VLAN creation (MGMT, USERS, VOICE)
- Management SVI setup
- Default gateway configuration
- Interface configuration

---

### 🔹 Security Automation
- Disable unused interfaces
- Add descriptions
- Shutdown unused ports

---

### 🔹 Operational Workflow (MOP Style)

#### ✅ Pre-check
- Validate interface configuration
- Check VLAN table
- Verify interface status

#### 🚀 Deploy
- Create new VLANs
- Assign interfaces
- Apply configuration changes

#### 🔍 Post-check
- Verify VLAN assignment
- Confirm interface configuration
- Validate expected state

---

## 🔄 CI/CD Pipeline

Implemented using GitHub Actions.

### Workflow:
1. Code pushed to GitHub
2. CI pipeline triggered
3. Runner connects via Tailscale
4. SSH into automation VM
5. Execute:
   - Pre-check
   - Deploy
   - Post-check

---

## 🔐 Secure Access (Tailscale)

- Private mesh VPN
- No public exposure required
- GitHub runner joins tailnet using auth key
- SSH access to VM via private IP (100.x.x.x)

---

## 🧠 Key Learnings

- Infrastructure as Code (IaC) for networking
- Ansible roles & inventory design
- Idempotent configuration management
- Real-world deployment workflows (MOP)
- CI/CD integration with network automation
- Secure access without exposing infrastructure

---

## 🚀 Future Improvements

- Replace password auth with SSH keys
- Add approval step before deployment
- Implement rollback automation
- Separate environments (lab / staging / prod)
- Add monitoring and alerting

---

## 👤 Author
Built as a hands-on lab to simulate real enterprise network automation workflows.
