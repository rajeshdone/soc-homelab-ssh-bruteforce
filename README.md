# 🛡️ SOC Homelab — SSH Brute-Force Attack & Detection

> A hands-on homelab project simulating a real-world SSH brute-force attack and building detection rules in Splunk SIEM.

---

## 📌 Project Overview

This project simulates a complete **attack → detect → alert** cycle inside a controlled virtual lab environment. It demonstrates core SOC analyst skills including threat simulation, log analysis, and SIEM alert creation.

---

## 🧱 Lab Architecture

| Role     | OS           | IP Address     | Tool Used             |
|----------|--------------|----------------|-----------------------|
| Attacker | Kali Linux   | 10.0.2.3       | Hydra, Nmap           |
| Victim   | Ubuntu Linux | 192.168.56.101 | SSH Server, rsyslog   |
| SIEM     | Windows Host | 192.168.56.1   | Splunk Free           |

**Virtualization:** VirtualBox with NAT Network (`LabNetwork` — 10.0.2.0/24)

---

## ⚙️ Tools & Technologies

- **VirtualBox** — Virtual machine management
- **Kali Linux** — Attacker machine
- **Ubuntu Server** — Victim machine with SSH enabled
- **Nmap** — Network reconnaissance & port scanning
- **Hydra** — SSH brute-force attack tool
- **Splunk (Free)** — SIEM for log ingestion, detection, and dashboards
- **rsyslog** — Log forwarding from Ubuntu to Splunk (UDP 514)

---

## 🔴 Attack Phase

### Step 1 — Network Reconnaissance (Nmap)

Scanned the victim machine to confirm SSH port is open.

```bash
nmap -p 22 192.168.56.101
```

**Finding:** Port 22/tcp open — SSH service confirmed running on the victim.

### Step 2 — Custom Password Wordlist

Created a targeted wordlist for the brute-force attack:

```bash
cat > mypass.txt << 'EOF'
ubuntu
password123
password
1234
123456
ubuntu123
yourspasswordhere
EOF
```

### Step 3 — SSH Brute-Force Attack (Hydra)

Launched a dictionary attack against the victim's SSH service:

```bash
hydra -l ubuntu -P mypass.txt ssh://192.168.56.101 -t 2 -v
```

**Result:** ✅ Successfully cracked credentials — `login: ubuntu` / `password: ubuntu123`

![Nmap Scan + Hydra Attack](hydra-nmap-attack.png)

---

## 🟢 Detection Phase (Splunk SIEM)

### Log Forwarding Setup
- Ubuntu configured to forward auth logs (`/var/log/auth.log`) to Splunk via **rsyslog over UDP 514**
- Splunk index: `main` | Source type: `syslog`

### Detection Search (SPL Query)

Searched Splunk for all failed SSH login attempts:

```spl
index=main "Failed password"
```

**Result:** **437 events** detected — all failed login attempts from the brute-force attack captured in Splunk.

![Splunk Failed Password Events](splunk-failed-logins.png)

---

## 🚨 Alert Phase

### Alert Configuration
- **Alert name:** `SSH Brute Force Detected`
- **Trigger condition:** More than 5 failed SSH logins within 5 minutes
- **Severity:** 🔴 High
- **Type:** Scheduled
- **Mode:** Digest

### Result — Alert Fired 9 Times

The alert successfully triggered **9 times** across the attack window, confirming real-time detection of the brute-force campaign.

![SSH Brute Force Alert Triggered](splunk-alerts-triggered.png)

---

## 🧠 Key Learnings

- How SSH brute-force attacks work at a technical level
- How to forward Linux auth logs to a SIEM using rsyslog
- Writing SPL (Splunk Processing Language) detection queries
- Building real-time scheduled alerts in Splunk with severity levels
- The full **Attack → Log → Detect → Alert** analyst workflow

---

---

## ⚠️ Disclaimer

This project was conducted in a **closed, isolated virtual lab environment** for educational purposes only. All attacks were performed on machines I own and control. This project does not promote unauthorized access to any systems.

---

## 👤 Author

**Rajesh** — Aspiring SOC Analyst
🔗 [LinkedIn Profile](https://linkedin.com/in/rajesh105)
