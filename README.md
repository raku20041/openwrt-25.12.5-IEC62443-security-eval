# OpenWrt 25.12.5 Security Evaluation & Hardening Report

This repository contains a hands-on cybersecurity evaluation and security hardening project on **OpenWrt 25.12.5 (Dave's Guitar)** firmware, structured in compliance with **IEC 62443-4-2 Security Level 2 (SL-2)** standards and **ISO/IEC 17025** laboratory documentation principles.

---

## 📋 Evaluation Overview

| Parameter | Details |
| :--- | :--- |
| **Target of Evaluation (TOE)** | OpenWrt 25.12.5 (`r33051-f5dae5ece4`) |
| **Codename** | Dave's Guitar |
| **Package Manager** | Alpine Package Keeper (`apk`) |
| **Standard / Profile** | IEC 62443-4-2 Security Level 2 (SL-2) |
| **Attacker Host** | Kali Linux (`192.168.1.100`) |
| **Target Host** | OpenWrt Gateway (`192.168.1.1`) |
| **Overall Status** | **100% RE-TEST PASS** |

---

## 🛠️ Security Toolchain

* **Network Scanning & Enumeration**: Nmap v7.95
* **Traffic Analysis & Inspection**: Wireshark v4.2.0
* **Firmware Analysis & Static Audit**: Binwalk v2.3.3, SquashFS-tools
* **Access Control & CLI**: OpenSSH CLI, Curl

---

## 📊 Compliance Matrix

| Requirement | Description | Initial Result | Re-test Result | Mitigation & Hardening Summary |
| :---: | :--- | :---: | :---: | :--- |
| **CR 1.1** | Human User Identification & Authentication | ❌ **FAIL** | ✅ **PASS** | Disabled unauthenticated logins; enforced strong password policies via UCI. |
| **CR 3.1** | Communication Integrity & Confidentiality | ❌ **FAIL** | ✅ **PASS** | Disabled plaintext HTTP; installed `luci-ssl` via `apk` and enforced TLS 1.3 encryption. |
| **CR 7.1** | Denial of Service & Attack Surface Reduction | ✅ **PASS** | ✅ **PASS** | Verified minimum attack surface via full TCP port scan (Port 22/443 only). |
| **FIRM-01** | Firmware Static Analysis & Hardening Audit | ✅ **PASS** | ✅ **PASS** | Unpacked SquashFS rootfs via Binwalk; verified no hardcoded keys/secrets. |

---

## 🔍 Technical Deep-Dive Cases

### 1. [CR 1.1] Identification and Authentication (Human User Auth)

* **Finding**: The default `root` account had no password configured, allowing unauthenticated SSH and LuCI Web access.
* **Initial Evaluation (Fail)**:
  ```bash
  ssh root@192.168.1.1
  # Granted root shell access without password prompt
