# 🔐 Google Cloud Cybersecurity Capstone Project
### Incident Response & Vulnerability Remediation on Google Cloud Platform

<p align="left">
  <img src="https://img.shields.io/badge/Google%20Cloud-Security%20Command%20Center-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white"/>
  <img src="https://img.shields.io/badge/Compliance-PCI%20DSS%203.2.1-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Platform-Qwiklabs-orange?style=for-the-badge"/>
</p>

---

## 📋 Project Overview

This capstone project simulates a real-world **cloud security incident response** scenario at a fictional enterprise, **Cymbal Retail** — a global retail company operating 170 stores across 28 countries with $15 billion in annual revenue.

Following a **massive data breach** in which attackers gained unauthorized access to sensitive customer and credit card data, I was tasked as a junior cloud security analyst to:

- Identify and analyze active vulnerabilities using **Google Cloud Security Command Center (SCC)**
- Contain the breach by isolating compromised infrastructure
- Remediate misconfigurations across Compute, Storage, and Networking
- Verify compliance against **PCI DSS 3.2.1**

---

## 🧰 Tools & Technologies

| Category | Tools |
|---|---|
| Cloud Platform | Google Cloud Platform (GCP) |
| Security | Security Command Center, IAP, Shielded VMs |
| Compute | Compute Engine, VM Snapshots |
| Storage | Cloud Storage, IAM, Bucket Policies |
| Networking | VPC Firewall Rules, Flow Logs |
| Compliance | PCI DSS 3.2.1, GCP Security Health Analytics |

---

## 🗂️ Project Tasks

### Task 1 — Analyze the Data Breach & Gather Intelligence

**Objective:** Use GCP Security Command Center to map the attack surface and identify all active vulnerabilities.

**Approach:**
- Navigated to **Security > Risk Overview** and examined misconfigurations by resource type
- Reviewed the **PCI DSS 3.2.1 Compliance Report** to identify non-compliant controls
- Filtered findings by resource type (Storage Bucket, Compute Instance, Firewall) for targeted analysis

**Key Findings Identified:**

| Finding | Severity | Resource |
|---|---|---|
| Public Bucket ACL | 🔴 High | Cloud Storage Bucket |
| Public IP Address | 🔴 High | Compute Instance (cc-app-01) |
| Open SSH Port (0.0.0.0/0) | 🔴 High | Firewall |
| Open RDP Port (0.0.0.0/0) | 🔴 High | Firewall |
| Full API Access | 🟡 Medium | Compute Instance |
| Default Service Account Used | 🟡 Medium | Compute Instance |
| Compute Secure Boot Disabled | 🟡 Medium | Compute Instance |
| Firewall Rule Logging Disabled | 🟡 Medium | Firewall |
| Malware: Bad Domain | 🟢 Low | Compute Instance |
| Bucket Policy Only Disabled | 🟡 Medium | Cloud Storage Bucket |

**Non-compliant PCI DSS 3.2.1 Controls:**
- **1.2.1** — Firewall rule logging disabled
- **7.1.2** — Basic roles (Owner/Writer/Reader) in use
- **10.1** — Firewall rule logging disabled; VPC Flow Logs not enabled
- **10.2** — Audit logging not configured across all services

<details>
<summary>📸 Screenshots — Task 1</summary>

**Risk Overview — Misconfigurations by Resource Type**
![Risk Overview](assets/screenshots/01_risk_overview_scc.png)

**Security Findings — All Active Findings**
![All Findings](assets/screenshots/02_findings_all.png)

**PCI DSS 3.2.1 Compliance Report**
![PCI DSS Report](assets/screenshots/03_pci_dss_compliance.png)

**PCI DSS — Non-Compliant Controls Expanded**
![PCI DSS Controls](assets/screenshots/04_pci_dss_controls_detail.png)

**Findings Filtered — Cloud Storage Bucket**
![Bucket Findings](assets/screenshots/05_findings_storage_bucket.png)

**Findings Filtered — Compute Instance**
![Compute Findings](assets/screenshots/06_findings_compute_instance.png)

**Findings Filtered — Firewall (Last 30 Days)**
![Firewall Findings](assets/screenshots/07_findings_firewall.png)

</details>

---

### Task 2 — Fix Compute Engine Vulnerabilities

**Objective:** Isolate the compromised VM, restore from a clean snapshot, harden the new instance, and decommission the old one.

**Steps Executed:**

1. **Stopped the compromised VM** `cc-app-01` — preventing further malicious activity
2. **Created a new VM** `cc-app-02` from the pre-breach snapshot `cc-app01-snapshot`:
   - Machine type: `e2-medium` (Shared-core)
   - External IPv4 address: **None** (eliminates Public IP Address finding)
   - Service account: **Qwiklabs User Service Account** (removes Full API Access finding)
   - Network tag: `cc` (scopes firewall rules to this instance only)
3. **Enabled Secure Boot** on `cc-app-02` via the Edit instance page (addresses Compute Secure Boot Disabled)
4. **Deleted `cc-app-01`** — eliminating the source of the breach

**Security Improvements Achieved:**
- ✅ No public IP address on new VM
- ✅ Scoped service account (not default, no full API access)
- ✅ Secure Boot enabled
- ✅ Network-tagged for controlled firewall access
- ✅ Compromised VM permanently removed

<details>
<summary>📸 Screenshots — Task 2</summary>

**VM Instances — cc-app-01 (Compromised)**
![VM Instance](assets/screenshots/08_vm_instances_cc_app_01.png)

**Stop VM — Confirmation Dialog**
![Stop VM](assets/screenshots/10_stop_vm_confirmation.png)

**Create Instance — Snapshot-Based VM Configuration**
![Create Instance](assets/screenshots/11_create_instance_snapshot.png)

**Edit cc-app-02 — Enable Secure Boot**
![Edit VM Secure Boot](assets/screenshots/12_edit_cc_app_02_secure_boot.png)

**Start cc-app-02 — Confirmation**
![Start VM](assets/screenshots/13_start_cc_app_02.png)

**Delete cc-app-01 — Confirmation**
![Delete VM](assets/screenshots/14_delete_cc_app_01.png)

</details>

---

### Task 3 — Fix Cloud Storage Bucket Permissions

**Objective:** Remove all public access from the compromised storage bucket and enforce uniform access control.

**Context:** A file `myfile.csv` containing sensitive customer data was publicly readable in the bucket due to an ACL entry granting `allUsers` read access — a critical finding from the breach investigation.

**Steps Executed:**

1. Navigated to the exposed bucket `qwiklabs-gcp-01-a2453edddbf4_bucket`
2. Clicked **Prevent public access** and confirmed — overrides all ACL entries for `allUsers` and `allAuthenticatedUsers`
3. Switched access control from **Fine-grained (object-level ACLs)** to **Uniform (bucket-level)** — enforces a single consistent permission model
4. Removed the `allUsers: Storage Legacy Bucket Reader` IAM binding

**Security Improvements Achieved:**
- ✅ Public bucket ACL finding resolved
- ✅ Bucket policy (uniform access) enforced
- ✅ No anonymous/public read access to stored data
- ✅ Fine-grained object ACLs disabled — single control plane for access

<details>
<summary>📸 Screenshots — Task 3</summary>

**Bucket Contents — Exposed myfile.csv**
![Bucket Contents](assets/screenshots/15_bucket_contents_myfile_csv.png)

**Bucket Permissions — allUsers Binding Visible**
![Bucket Permissions](assets/screenshots/16_bucket_permissions_allUsers.png)

**Prevent Public Access — Confirmation Dialog**
![Prevent Public Access](assets/screenshots/17_prevent_public_access_confirm.png)

**Bucket — Public Access Prevented, Uniform Access Enabled**
![Bucket Secured](assets/screenshots/18_bucket_not_public_uniform.png)

</details>

---

### Task 4 — Restrict Firewall Port Access (SSH via IAP)

**Objective:** Replace the overly permissive `default-allow-ssh` rule with a scoped rule that only permits SSH from Google Cloud's Identity-Aware Proxy (IAP) CIDR.

**Firewall Rule Created:**

| Property | Value |
|---|---|
| Rule Name | `limit-ports` |
| Direction | Ingress |
| Action | Allow |
| Target Tags | `cc` |
| Source IP Range | `35.235.240.0/20` (GCP IAP range) |
| Protocol/Port | TCP: 22 |
| Priority | Default |

**Why IAP?** Instead of opening SSH to the entire internet (`0.0.0.0/0`), IAP acts as an authentication proxy — only GCP-authenticated users can tunnel SSH through it, eliminating direct public SSH exposure.

**Security Improvements Achieved:**
- ✅ SSH no longer open to the internet
- ✅ Access scoped to IAP-authenticated sessions only
- ✅ Firewall rule targets only VMs tagged `cc` — no blast radius

---

### Task 5 — Harden Firewall Configuration

**Objective:** Delete three overly permissive default firewall rules and enable logging on remaining rules for full audit visibility.

**Rules Deleted:**

| Rule | Protocol | Risk |
|---|---|---|
| `default-allow-ssh` | TCP/22 from 0.0.0.0/0 | Open SSH to entire internet |
| `default-allow-rdp` | TCP/3389 from 0.0.0.0/0 | Open RDP to entire internet |
| `default-allow-icmp` | ICMP from 0.0.0.0/0 | Enables network reconnaissance |

**Logging Enabled On:**
- `limit-ports` — the new IAP-scoped SSH rule
- `default-allow-internal` — monitors east-west traffic within VPC

**Security Improvements Achieved:**
- ✅ Open SSH port finding resolved
- ✅ Open RDP port finding resolved
- ✅ Firewall rule logging disabled finding resolved
- ✅ Full auditability of allowed traffic via Cloud Logging

---

### Task 6 — Verify PCI DSS 3.2.1 Compliance

**Objective:** Re-run the PCI DSS 3.2.1 compliance report to confirm all critical and high severity findings have been remediated.

**Outcome:**  
All targeted findings were resolved. The following controls moved from **Non-compliant → Compliant**:

| PCI DSS Control | Finding | Status |
|---|---|---|
| 1.2.1 | Firewall rule logging disabled | ✅ Resolved |
| 7.1.2 | Basic roles / Full API access | ✅ Resolved |
| 10.1 | Firewall rule logging disabled | ✅ Resolved |
| 10.2 | Cloud Audit Logging not configured | ✅ Resolved |

> ⚠️ **Note:** VPC Flow Logs (subnetwork-level) remained disabled as those subnetworks are lab infrastructure — this is an acknowledged exception in the remediation scope.

---

## 🔑 Key Skills Demonstrated

- **Threat Intelligence & Analysis** — Using GCP SCC to triage and prioritize security findings by severity and resource type
- **Incident Containment** — Stopping a compromised VM, restoring from a clean snapshot, scoping new resources with hardened configs
- **IAM Hardening** — Removing default service accounts, enforcing least privilege on bucket access
- **Network Security** — Replacing overly permissive firewall rules with IAP-scoped access, enabling rule logging
- **Compliance Verification** — Mapping technical findings to PCI DSS 3.2.1 controls and re-validating post-remediation
- **Cloud-native Forensics** — Analysing SCC findings (malware bad domain, public IP exposure, data exfiltration indicators) to reconstruct the attack path

---

## 📁 Repository Structure

```
google-cloud-security-capstone/
│
├── README.md                  ← This file
│
├── assets/
│   └── screenshots/           ← Evidence screenshots from GCP console
│       ├── 01_risk_overview_scc.png
│       ├── 02_findings_all.png
│       ├── 03_pci_dss_compliance.png
│       ├── 04_pci_dss_controls_detail.png
│       ├── 05_findings_storage_bucket.png
│       ├── 06_findings_compute_instance.png
│       ├── 07_findings_firewall.png
│       ├── 08_vm_instances_cc_app_01.png
│       ├── 10_stop_vm_confirmation.png
│       ├── 11_create_instance_snapshot.png
│       ├── 12_edit_cc_app_02_secure_boot.png
│       ├── 13_start_cc_app_02.png
│       ├── 14_delete_cc_app_01.png
│       ├── 15_bucket_contents_myfile_csv.png
│       ├── 16_bucket_permissions_allUsers.png
│       ├── 17_prevent_public_access_confirm.png
│       └── 18_bucket_not_public_uniform.png
│
└── docs/
    └── Capstone_Project.pdf   ← Original lab specification
```

---

## 🏆 Certification Context

This project is the capstone assessment for the **Google Cloud Cybersecurity Certificate** program, validating hands-on competency in:

- Cloud security architecture and threat detection in GCP environments
- Incident response lifecycle in a cloud-native context
- Security Command Center operations
- PCI DSS compliance verification

---

## 👤 Author

**Mboya Edgar**  
Cybersecurity Associate | Red Team Operator & OSINT Specialist  
📧 eddi.mboya@gmail.com  
🔗 [linkedin.com/in/mboya-edgar](https://linkedin.com/in/mboya-edgar)  
📍 Nairobi, Kenya

---

*Completed: March 2, 2026 | Google Cloud Qwiklabs Environment*
