
<div align="center">

# 🛡️ Wazuh SIEM Home Lab

### Real-Time Threat Detection & Security Monitoring Platform

[![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)](https://github.com/yassine674/wazuh-siem-homelab)
[![Platform](https://img.shields.io/badge/Platform-Ubuntu_Server-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![SIEM](https://img.shields.io/badge/SIEM-Wazuh_4.7-005571?style=for-the-badge)](https://wazuh.com/)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)

*Complete enterprise-grade SIEM infrastructure for defensive security operations*

[📖 Documentation](#documentation) • [🚀 Quick Start](#quick-start) • [🎯 Features](#features) • [📸 Screenshots](#screenshots)

</div>

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Technical Stack](#technical-stack)
- [Installation Guide](#installation-guide)
- [Configuration](#configuration)
- [Use Cases](#use-cases)
- [Screenshots](#screenshots)
- [Learning Outcomes](#learning-outcomes)
- [Resources](#resources)
- [Contributing](#contributing)
- [License](#license)

---

## 🎯 Overview

This project demonstrates the **complete deployment and configuration of a production-ready SIEM environment** using Wazuh, an open-source XDR platform. The lab simulates real-world SOC operations with multi-source log collection, automated threat detection, and real-time security monitoring.

### Project Objectives

- ✅ Deploy enterprise SIEM infrastructure from scratch
- ✅ Configure multi-platform agent deployment (Windows/Linux)
- ✅ Implement File Integrity Monitoring (FIM)
- ✅ Create custom detection rules for threat hunting
- ✅ Build real-time security dashboards
- ✅ Simulate incident response workflows

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Wazuh Manager (Ubuntu)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Indexer    │  │    Server    │  │  Dashboard   │      │
│  │ (OpenSearch) │  │  (Analysis)  │  │   (Kibana)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │ Log forwarding (Port 1514)
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼────────┐   ┌────────▼───────┐   ┌────────▼───────┐
│ Windows Agent  │   │  Linux Agent   │   │  Kali Agent    │
│   (Endpoint)   │   │   (Ubuntu)     │   │  (AttackBox)   │
└────────────────┘   └────────────────┘   └────────────────┘
```

**Components:**
- **Wazuh Manager**: Central analysis engine and dashboard
- **Agents**: Lightweight collectors on monitored endpoints
- **OpenSearch**: Log indexing and storage backend
- **Dashboard**: Real-time visualization interface

---

## ✨ Features

### 🔍 Detection Capabilities
- **File Integrity Monitoring (FIM)**: Track unauthorized file modifications
- **Rootkit Detection**: Identify hidden processes and malware
- **Vulnerability Scanning**: Automated CVE detection
- **Log Analysis**: Multi-source correlation (Syslog, Windows Events, Auth logs)
- **Active Response**: Automated threat mitigation actions

### 📊 Monitoring & Alerting
- Real-time security event dashboards
- Custom alert rules with severity classification
- Email/webhook notifications for critical events
- Compliance reporting (PCI-DSS, HIPAA, GDPR)

### 🛠️ Advanced Configuration
- Custom decoder and rule creation
- CDB lists for threat intelligence integration
- Agent grouping and centralized policy management
- API integration for automation workflows

---

## 🔧 Technical Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **Operating System** | Ubuntu Server | 22.04 LTS |
| **SIEM Platform** | Wazuh | 4.7.0 |
| **Indexer** | OpenSearch | 2.10 |
| **Dashboard** | Wazuh Dashboard | 4.7.0 |
| **Agent OS** | Windows 10/11, Linux | Latest |
| **Hypervisor** | VirtualBox / VMware | Latest |

---

## 🚀 Installation Guide

### Prerequisites

- **Hardware**: 4 GB RAM minimum (8 GB recommended)
- **OS**: Ubuntu Server 22.04 LTS
- **Network**: Static IP or bridged networking
- **Storage**: 50 GB free disk space

### Step 1: Wazuh Manager Installation

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Download Wazuh installation script
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# Run all-in-one installation
sudo bash wazuh-install.sh -a
```

### Step 2: Access Dashboard

```bash
# Get admin credentials
sudo tar -xvf wazuh-install-files.tar
```

Navigate to: `https://<SERVER_IP>`

**Default credentials:**
- Username: `admin`
- Password: (extracted from tar file)

### Step 3: Deploy Windows Agent

```powershell
# Download installer
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.0-1.msi -OutFile wazuh-agent.msi

# Install with manager IP
msiexec /i wazuh-agent.msi WAZUH_MANAGER="<MANAGER_IP>" /q

# Start service
NET START WazuhSvc
```

### Step 4: Verify Agent Connection

```bash
# On Wazuh Manager
sudo /var/ossec/bin/agent_control -l
```

---

## ⚙️ Configuration

### Custom FIM Rule for Critical Directories

**Edit**: `/var/ossec/etc/ossec.conf`

```xml
<syscheck>
  <directories check_all="yes" realtime="yes">/home/*/Documents</directories>
  <directories check_all="yes" realtime="yes">/etc</directories>
  <ignore>/etc/mtab</ignore>
</syscheck>
```

**Restart Wazuh:**
```bash
sudo systemctl restart wazuh-manager
```

### Custom Alert Rule (Suspicious PowerShell)

**Create**: `/var/ossec/etc/rules/local_rules.xml`

```xml
<group name="windows,powershell">
  <rule id="100001" level="12">
    <if_sid>60009</if_sid>
    <match>Invoke-Mimikatz|Invoke-Expression</match>
    <description>Suspicious PowerShell command detected</description>
    <mitre>
      <id>T1059.001</id>
    </mitre>
  </rule>
</group>
```

---

## 🎯 Use Cases

### 1. Detecting Unauthorized File Modifications

**Scenario**: Monitor `/etc/passwd` for unauthorized changes

**Alert Triggered:**
```json
{
  "rule": {
    "level": 7,
    "description": "Integrity checksum changed"
  },
  "syscheck": {
    "path": "/etc/passwd",
    "event": "modified"
  }
}
```

### 2. Brute-Force Attack Detection

**Scenario**: Multiple failed SSH login attempts

**Custom Rule:**
```xml
<rule id="100002" level="10" frequency="5" timeframe="120">
  <if_matched_sid>5503</if_matched_sid>
  <description>SSH brute force attack detected</description>
</rule>
```

### 3. Malware Detection via Rootkit Check

**Command:**
```bash
sudo /var/ossec/bin/rootcheck -d /var/ossec
```

**Dashboard Alert**: Hidden processes flagged under Security Events

---

## 📸 Screenshots

### Dashboard Overview
<p align="center">
  <img src="assets/wazuh-dashboard.png" width="800" alt="Wazuh Dashboard">
</p>

### File Integrity Monitoring Alerts
<p align="center">
  <img src="assets/fim-alerts.png" width="800" alt="FIM Alerts">
</p>

### Agent Management
<p align="center">
  <img src="assets/agent-status.png" width="800" alt="Agent Status">
</p>

---

## 🎓 Learning Outcomes

By completing this lab, you will:

✅ **Understand SIEM architecture** and component interactions  
✅ **Master log collection** from heterogeneous sources  
✅ **Create detection rules** using XML and regex patterns  
✅ **Perform threat hunting** using dashboard queries  
✅ **Configure automated responses** to security events  
✅ **Document incidents** following SOC procedures  

---

## 📚 Resources

### Official Documentation
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh Ruleset](https://github.com/wazuh/wazuh-ruleset)
- [OpenSearch Documentation](https://opensearch.org/docs/)

### Additional Learning
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [SOC Prime Threat Detection Marketplace](https://tdm.socprime.com/)

### Community
- [Wazuh Slack Community](https://wazuh.com/community/)
- [r/cybersecurity](https://reddit.com/r/cybersecurity)

---

## 🤝 Contributing

Contributions are welcome! Please feel free to:

- Report bugs or issues
- Suggest new detection rules
- Share dashboard improvements
- Submit pull requests

<div align="center">

**⭐ If you found this project helpful, please star the repository!**

</div>

