# Using-a-Scan-Template-to-Scan-a-Windows-10-VM

<img width="1914" height="870" alt="image" src="https://github.com/user-attachments/assets/fca70403-607e-4c56-8de7-d67517d7c412" />
<img width="1837" height="606" alt="image" src="https://github.com/user-attachments/assets/1b041a65-28f2-472f-92e6-320834c07525" />
# 🧪 Windows 10 Vulnerability Management Lab — Using a Scan Template (Azure + Tenable)

This repo contains a **full hands-on lab project** where we:
1. Build a Windows 10 VM in Azure.  
2. Configure intentional vulnerabilities (weak accounts, disabled firewall, etc.).  
3. Create a **custom Tenable scan template** with compliance checks (DISA STIG).  
4. Run authenticated scans against the VM.  
5. Analyze results, audits, and remediations.  
6. Clean up the lab environment.

The instructions below are built from both **step-by-step lab notes** and the **transcribed walkthrough** for a realistic learning flow.

---

## 📘 Prerequisites
- Azure account → https://portal.azure.com  
- Tenable Vulnerability Management account → https://cloud.tenable.com  
- `Tenable_Vulnerability_Management-User_Guide.pdf`  
- RDP / Azure Bastion access to your VM  
- Basic Windows & PowerShell knowledge  

⚠️ **Security Warning:**  
This lab intentionally weakens system security for training. Never run these steps on production or internet-facing systems you care about. Always delete your VM and clean up resources after the exercise.

---

## 🖥️ Step 1 — Create a Windows 10 Virtual Machine
- Log into Azure and create a new resource group.  
- Deploy a **Windows 10 Pro** VM.  
- VM size: `Standard_DS1_v2` (or equivalent).  
- Username: `labuser` (example).  
- Password: `Cyberlab123!` (⚠️ weak, for demo only).  
- Use a standard HDD for disks.  
- Place it in your CyberRange VNet (or equivalent).  
- Check “Delete NIC when VM is deleted.”  
- Disable boot diagnostics.  
- Deploy the VM.  

*Transcript reference:* (0:00–2:20) VM creation.  

📸 Add screenshot: `screenshots/azure-create-vm.png`

---

## 🛠️ Step 2 — Configure Vulnerabilities (Inside VM)
Once the VM is running, log in via RDP or Bastion:

1. **Disable Windows Firewall**  
   - Run → `wf.msc` → Disable Domain, Private, and Public profiles.  

2. **Create Insecure Accounts**  
   - Enable the built-in **Administrator** account.  
   - Set password to `password` (weak).  
   - Set password to never expire.  
   - Add account to Administrators group.  
   - Enable **Guest** account.  
   - Add Guest account to Administrators group.  

3. **Verify Services**  
   - Remote Registry service → enabled.  
   - Server service → enabled.  

📸 Add screenshot: `screenshots/vm-config.png`  

*Transcript reference:* (3:00–7:30) disabling firewall, enabling accounts.

---

## 🌐 Step 3 — Networking
- If using **internal scan engine**: no NSG changes needed.  
- If using **cloud scan engine**: open the NSG completely (all inbound).  
- Test connectivity:  

```bash
# 🔑 Step 4 — Create Tenable Scan Template

Log into Tenable → **Scans > Create Scan Template > Advanced Network Scan**.  

Configure settings as follows:

### Basic
- Name: `Win10-DISA-STIG-Template`  
- ✅ Start Remote Registry service  
- ✅ Enable administrative shares  
- ✅ Start Server service  

### Discovery
- ✅ Ping the remote host  
- ✅ Use fast network discovery  
- ✅ TCP port scanning  

### Assessment
- ✅ Perform thorough tests  
- ❌ Uncheck “Only use credentials provided by the user”  

### Credentials
- Add `labuser` / `Cyberlab123!` (or the account you created).  

### Compliance
- Add **DISA Windows 10 STIG** compliance audit.  

📸 Add screenshot: `screenshots/tenable-template-config.png`  

*Transcript reference:* (8:00–14:55) scan template creation.  

---

# ▶️ Step 5 — Create and Run a Scan

1. Create a new scan → **User Defined → Your Template**.  
2. Name it: `DISA-STIG-Win10-Test`.  
3. Scanner: **Internal** (preferred).  
   - If using external: open NSG + use Public IP.  
4. Target: **Private IP** (preferred).  
5. Credentials: already included in template.  
6. Launch the scan.  

📸 Add screenshot: `screenshots/tenable-scan-launch.png`  

*Transcript reference:* (15:00–18:10).  

---

# 📊 Step 6 — Analyze Results

### Example Output
- Duration: ~42 minutes (longer because of compliance checks).  
- Findings:  
  - 1 Critical  
  - 15 High  
  - 23 Medium  
  - 2 Low  

### Vulnerabilities
- Many related to missing Windows Updates.  
- Password complexity & account policies flagged.  

### Audits
Example results from DISA STIG compliance audit:
- ❌ Guest account enabled → flagged.  
- ❌ Administrator account uses weak password.  
- ❌ Password history not configured (requires 24).  
- ❌ Minimum password length not 14 characters.  
- ❌ Legal banner missing.  

### Remediations
- Install Windows Updates → mitigates multiple vulns.  
- Disable Guest account.  
- Enforce password policies via Group Policy or PowerShell.  

📸 Add screenshots:  
- `screenshots/tenable-vulns.png`  
- `screenshots/tenable-audits.png`  
- `screenshots/tenable-remediations.png`  

*Transcript reference:* (18:20–24:30).  

---

# 🧹 Step 7 — Cleanup
- Delete the Windows 10 VM.  
- Confirm **resource group deletion** (removes NIC, disks, NSG, etc.).  
- Remove scan + template from Tenable console.  

📸 Add screenshot: `screenshots/azure-cleanup.png`  

*Transcript reference:* (25:00–end).  

---

# 🗂 Example Repo Layout

```bash
📦 win10-vuln-scan-template-lab/
├── 📁 screenshots/
│   ├── azure-create-vm.png
│   ├── vm-config.png
│   ├── nsg-open.png
│   ├── tenable-template-config.png
│   ├── tenable-scan-launch.png
│   ├── tenable-vulns.png
│   ├── tenable-audits.png
│   ├── tenable-remediations.png
│   └── azure-cleanup.png
├── 📁 reports/
│   ├── vuln-report.pdf
│   └── compliance-report.csv
├── 📄 Tenable_Vulnerability_Management-User_Guide.pdf
└── 📄 README.md

ping <VM Public IP>

