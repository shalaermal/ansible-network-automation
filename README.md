# 🚀 Networking Ansible Lab – End-to-End Automation with CI/CD

## 📌 Overview

This project demonstrates a **complete network automation workflow** using **Ansible** and **GitHub Actions CI/CD**, applied to a Cisco-based lab environment (EVE-NG).

It covers:

* 🔧 Configuration management (routers & switches)
* 🔁 Full lifecycle automation (Precheck → Deploy → Postcheck)
* 💾 Automated backups
* 🔄 Manual rollback (safe recovery)
* ⚙️ CI/CD pipeline with staged execution

---

## 🧠 Key Concepts

This project is structured around **real-world network automation practices**:

* Separation between **MOP (change-specific)** and **general automation**
* Use of **Ansible roles** for modular design
* **Inventory-driven configuration**
* **Fail-safe rollback strategy**
* CI/CD with **controlled execution flow**

---

## 🏗️ Project Structure

```
├── ansible.cfg
├── backups
│   ├── router_P1.cfg
│   ├── router_P2.cfg
│   ├── router_PE1.cfg
│   ├── router_PE2.cfg
│   ├── sw01.cfg
│   └── sw02.cfg
├── cisco-ios-5.0.0.tar.gz
├── cisco-ios.tar.gz
├── inventories
│   └── lab
│       ├── group_vars
│       │   ├── access_switches.yml
│       │   ├── all.yml
│       │   └── cisco_ios.yml
│       ├── hosts.yml
│       └── host_vars
│           ├── router_P1.yml
│           ├── router_P2.yml
│           ├── router_PE1.yml
│           ├── router_PE2.yml
│           ├── sw01.yml
│           ├── sw02.yml
│           ├── sw03.yml
│           └── sw04.yml
├── logs
│   └── ansible.log
├── playbooks
│   ├── access_switch_base.yml
│   ├── backup_config.yml
│   ├── cisco_base.yml
│   ├── mop
│   │   ├── deploy.yml
│   │   ├── postcheck.yml
│   │   ├── precheck.yml
│   │   ├── rollback.yml
│   │   └── vlan_mop.yml
│   ├── ping.yml
│   ├── precheck.yml
│   ├── rollback.yml
│   ├── show_version.yml
│   ├── site.yml
│   └── vlan
├── README.md
├── requirements.yml
└── roles
    ├── access_switch
    │   └── tasks
    │       └── main.yml
    ├── cisco_base
    │   └── tasks
    │       └── main.yml
    ├── router_interfaces
    │   └── tasks
    │       └── main.yml
    └── vlan_lifecycle
        └── tasks
            ├── deploy.yml
            ├── postcheck.yml
            ├── precheck.yml
            └── rollback.yml
```

---

## ⚙️ Technologies Used

* 🟢 Ansible (network automation)
* 🟣 Cisco IOS modules (`cisco.ios`)
* 🔵 GitHub Actions (CI/CD)
* 🟡 Tailscale (secure connectivity to lab)
* 🖥️ EVE-NG (network simulation)

---

## 🔌 Inventory Design

Devices are grouped logically:

```yaml
access_switches:
routers:
```

Each device is configured via:

* `host_vars/` → interface & device-specific config
* `group_vars/` → shared settings (connection, credentials)

---

## 🔄 Automation Workflow

### 🔹 General Workflow (CI/CD)

```
precheck → deploy → postcheck
```

* **Precheck**

  * Connectivity validation
  * Backup of current configs

* **Deploy**

  * VLAN configuration (switches)
  * Interface configuration (routers)

* **Postcheck**

  * Validation of applied configuration

---

## 💾 Backup Strategy

Backups are automatically created using:

```bash
ansible-playbook playbooks/precheck.yml
```

Stored in:

```
backups/
```

Includes:

* Router configs
* Switch configs

---

## 🔁 Rollback Strategy

### 🔹 Manual Rollback (Recommended)

Rollback is executed manually to ensure control:

```bash
ansible-playbook playbooks/rollback.yml
```

OR via GitHub Actions:

```
Actions → Manual Rollback → Run workflow
```

### ⚠️ Important

* Rollback restores **full device configuration**
* Overwrites current running config
* Should be used carefully

---

## 🚦 CI/CD Pipeline

Located in:

```
.github/workflows/network-ci.yml
```

### Pipeline Stages:

1. ✅ Precheck
2. 🚀 Deploy
3. 🔍 Postcheck

Rollback is **NOT automatic** (by design).

---

## 🧪 Testing Strategy

To test failure scenarios:

* Introduce incorrect configuration (e.g. wrong IP)
* Push changes to `main`
* Observe CI/CD failure
* Trigger rollback manually

---

## 🧱 Roles Overview

| Role                | Purpose                                  |
| ------------------- | ---------------------------------------- |
| `cisco_base`        | Base configuration (SSH, users, logging) |
| `access_switch`     | Switch configuration                     |
| `router_interfaces` | Interface IP configuration               |
| `vlan_lifecycle`    | VLAN deploy/validate/rollback            |

---

## 📌 Best Practices Implemented

* ✔️ Infrastructure as Code (IaC)
* ✔️ Idempotent configurations
* ✔️ Separation of concerns
* ✔️ Safe rollback design
* ✔️ CI/CD validation before changes

---

## 🚀 Future Improvements

* 🔄 Smart (partial) rollback
* ⏱️ Scheduled backups (cron-based)
* 🔍 Config diff tracking
* 🔐 Secret management improvements
* 📊 Monitoring integration

---

## 👨‍💻 Author

Built as a hands-on **network automation lab** to simulate real production workflows.

---

## ⚠️ Disclaimer

This project is intended for **lab and learning purposes**.
Use with caution in production environments.

---

