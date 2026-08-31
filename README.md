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
| **FIRM-01** | Firmware Static Analysis & Hardening Audit | ✅ **PASS** | ✅ **PASS** | Unpacked SquashFS rootfs via Binwalk; verified no hardcoded password hashes in `/etc/shadow`. |

---

## 🔍 Technical Deep-Dive Cases

### 1. [CR 1.1] Identification and Authentication (Human User Auth)

* **Finding**: The default `root` account had no password configured, allowing unauthenticated SSH and LuCI Web access.
* **Initial Evaluation (Fail)**:
  ```bash
  ssh root@192.168.1.1
  # Granted root shell access without password prompt
  ```

  ![CR1.1 Initial Fail](images/CR1.1_01_Fail.png)

* **Remediation & Hardening**:
  ```bash
  # 1. Set root password
  passwd

  # 2. Enforce Dropbear SSH password authentication via UCI
  uci set dropbear.@dropbear[0].PasswordAuth='on'
  uci set dropbear.@dropbear[0].RootPasswordAuth='on'
  uci set dropbear.@dropbear[0].EmptyPasswd='0'
  uci commit dropbear
  /etc/init.d/dropbear restart
  ```

* **Re-test Result (Pass)**:
  Unauthenticated connection attempts were rejected: `Permission denied (publickey,password).`

  ![CR1.1 Retest Pass](images/CR1.1_02_Pass.png)

---

### 2. [CR 3.1] Communication Integrity (Web Traffic Encryption)

* **Finding**: LuCI Web interface transmitted admin credentials in plaintext over HTTP (Port 80), exposing credentials to passive eavesdropping.
* **Initial Evaluation (Fail)**:
  Captured HTTP POST packets in Wireshark revealing plaintext `luci_username` and `luci_password`.

  ![CR3.1 Initial Fail](images/CR3.1_01_Fail.png)

* **Remediation & Hardening (OpenWrt 25.12 `apk` syntax)**:
  ```bash
  # 1. Update package lists and install SSL modules
  apk update
  apk add luci-ssl uhttpd-mod-ubus

  # 2. Configure uHTTPd to force HTTP-to-HTTPS redirect
  uci set uhttpd.main.redirect_https='1'
  uci set uhttpd.main.tls_redirect='1'
  uci commit uhttpd
  /etc/init.d/uhttpd restart
  ```

* **Re-test Result (Pass)**:
  Port 80 requests redirected via `301 Moved Permanently` to Port 443; Wireshark captured encrypted TLS 1.3 traffic only.

  ![CR3.1 Retest Pass](images/CR3.1_02_Pass.png)

---

### 3. [CR 7.1] Attack Surface Reduction (Port Scanning)

* **Objective**: Ensure only required management interfaces are open.
* **Command**:
  ```bash
  nmap -sS -sV -p- -T4 192.168.1.1
  ```
* **Result (Pass)**:
  Only Port 22/tcp (Dropbear SSH) and Port 443/tcp (uHTTPd HTTPS) were open, adhering to the principle of least privilege.

  ![CR7.1 Port Scan Pass](images/CR7.1_01_PortScan.png)

---

### 4. [FIRM-01] Firmware Extraction & Static Analysis

* **Objective**: Inspect unpacked firmware filesystem for hardcoded password hashes or default credentials in `/etc/shadow`.
* **Command**:
  ```bash
  binwalk -e --rm openwrt-25.12.5-x86-64-generic-squashfs-combined.img.gz
  cd _openwrt-25.12.5-x86-64-generic-squashfs-combined.img.extracted/squashfs-root/
  cat etc/shadow
  ```
* **Result (Pass)**:
  Verified `root` shadow entry contains no hardcoded password hash (`root:::0:99999:7:::`), confirming the default firmware does not ship with pre-baked credentials.

  ![FIRM-01 Binwalk Analysis](images/FIRM01_01_Binwalk_Analysis.png)

---

## 📁 Repository Structure

```text
.
├── README.md                     # Security evaluation report (this file)
└── images/                       # Evidential screenshots
    ├── CR1.1_01_Fail.png
    ├── CR1.1_02_Pass.png
    ├── CR3.1_01_Fail.png
    ├── CR3.1_02_Pass.png
    ├── CR7.1_01_PortScan.png
    └── FIRM01_01_Binwalk_Analysis.png
```
