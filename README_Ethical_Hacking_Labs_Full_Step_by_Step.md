# Ethical Hacking Practical Labs — Step-by-Step Command Guide

Beginner-friendly reference based on the practical notebook pages and the Lab 12 handout provided in chat.

> **Safety and accuracy:** Only use instructor-approved targets and isolated lab VMs. Do not scan public IP addresses, test real accounts, or attack real websites. Record what actually happens on your screen; do not claim a tool worked if it was missing or an error appeared.

## First: how to read every command

Commands below sometimes contain placeholders such as `LAB_DOMAIN` or `LAB_IP`. Those are labels explaining what value belongs there. **Do not type the placeholder literally.** Replace it with the real value from your lab sheet, instructor, or your own lab VM.

### Placeholder dictionary: what to put and where to find it

| Placeholder | What it means | Example / where to get the value |
|---|---|---|
| `LAB_DOMAIN` | The domain name assigned for the reconnaissance exercise | **Get it from:** the practical sheet or instructor. Your notebook's example is `dypatil.edu`; use that only if your instructor confirms it is the assigned domain. Do not make up a domain. |
| `LAB_IP` | IP address of the authorized target VM/service | **Get it from:** the instructor or target VM. On a VM you control, run `ip addr` inside it. Do not use Kali's own IP as the target unless the task says Kali is the target. |
| `LAB_CIDR` | The lab network range (subnet) | **Get it from:** instructor/lab network settings. It looks like `192.168.56.0/24`, but that is only an example format, not a value to copy without confirmation. |
| `LAB_INTERFACE` | Network interface to monitor | Run `ip addr` in Kali. Use the interface name shown for the lab network, often something like `eth0` or `ens33`. |
| `PORT_NUMBER` | The specific port to check | Use the port stated in the task or selected for your local lab, e.g. `80` is a common HTTP port. |
| `LAB_COMMUNITY_STRING` | SNMP community string for the training VM | Ask your instructor or read the lab configuration. Do not guess it. |
| `LAB_WORDLIST` | Path to the password dictionary file | Use the path supplied by your instructor or inspect the provided lab files with `ls`. |
| `HASH_FILE` | File containing instructor-provided sample password hashes | Use the exact filename supplied for the exercise; check with `ls`. |
| `SAMPLE_FILE` | Harmless training sample file to inspect | Use the exact filename your instructor provided; check with `ls`. |
| `COVER_IMAGE` | Image used for the steganography exercise | Use the exact image filename in your lab folder; check with `ls`. |
| `MESSAGE_FILE` | Text file containing the harmless message to hide | In this guide we create `secret.txt`. |
| `DVWA_URL` | URL of the DVWA instance you are authorized to test | Your Lab 12 handout says `http://127.0.0.1/dvwa/`. Use that only if DVWA is running locally at that address. |
| `YOUR_REPOSITORY_URL` | GitHub repository clone URL | Open your repository on GitHub → **Code** → copy the HTTPS URL. |
| `RULE_FILE` | Suricata local rules file | The notebook uses `/etc/suricata/rules/local.rules`. Check the actual file path on your Kali setup. |

### Identify Kali's IP and interface

Run:
```bash
ip addr
ip route
```
- `ip addr` lists network interfaces and their IP addresses.
- `ip route` shows routes and often the default gateway.
- `127.0.0.1` means this same machine (localhost), not another VM.
- The IP address assigned to Kali is not automatically the target VM's IP.

---

## Lab 1 — Set up an isolated virtual lab

**Aim:** Prepare Kali and the approved vulnerable target VM.

1. Open VMware/VirtualBox.
2. Start the approved lab VMs.
3. Keep vulnerable VMs on the instructor's host-only/internal network. Do not expose them to the public internet.
4. In Kali Terminal, run:
   ```bash
   ip addr
   ip route
   ```
5. Get the target VM IP from your instructor or by running `ip addr` **inside the target VM**.
6. If the lab permits a connectivity check:
   ```bash
   ping -c 4 LAB_IP
   ```
   Replace `LAB_IP` with the target VM's actual IP. If ping fails, the VM may still be running because ping can be blocked.

**Exam steps:** Install/open hypervisor → import VMs → configure isolated network → boot VMs → identify IP addresses → test connectivity.

---

## Lab 2 — Basic passive reconnaissance

**Aim:** Collect public information about an instructor-approved domain.

### A. WHOIS
1. Ask which domain the practical sheet assigns.
2. Example from your notebook: `dypatil.edu`. Use it only if your instructor confirms it is the intended target.
3. Run:
   ```bash
   whois LAB_DOMAIN
   ```
4. **Replace `LAB_DOMAIN` with the actual assigned domain.** For example, if the instructor explicitly assigned `example.com`, type `whois example.com`, not `whois LAB_DOMAIN`.
5. Record registrar, registration/expiry dates, domain status, and name servers when shown.

### B. DNS lookups
Use the same instructor-approved domain each time:
```bash
nslookup LAB_DOMAIN
nslookup -type=A LAB_DOMAIN
nslookup -type=MX LAB_DOMAIN
nslookup -type=NS LAB_DOMAIN
nslookup -type=TXT LAB_DOMAIN
```
Replace each occurrence of `LAB_DOMAIN` with the exact domain from the practical sheet. These commands query the default DNS result, IPv4 address records, mail server records, name server records, and TXT records respectively.

### C. Google Dorking
This is done in a browser, not Terminal.
1. Open Google.
2. Type `site:LAB_DOMAIN` and replace `LAB_DOMAIN` with the approved domain.
3. For publicly indexed PDFs, type `site:LAB_DOMAIN filetype:pdf`.
4. Record public results only. Do not attempt to access private or restricted data.

### D. Shodan
1. Open https://www.shodan.io/
2. Search the instructor-approved domain/host.
3. Record displayed services, ports, banners, and result details.
4. Treat results as clues; they may be old.

**Order to remember:** WHOIS → DNS → Google search operators → Shodan → document findings.

---

## Lab 3 — Port scanning, service detection, OS detection (Nmap/Masscan)

**Aim:** Discover hosts and services in an authorized lab.

### A. Check Nmap
```bash
nmap --version
```
A version number means it is installed. `command not found` means it is unavailable.

### B. Discover hosts
1. Ask your instructor for the lab subnet.
2. Run:
   ```bash
   nmap -sn LAB_CIDR
   ```
3. Replace `LAB_CIDR` with the exact subnet assigned to your lab. Do not guess the range.
4. Record hosts that are reported up.

### C. Scan one lab VM
```bash
nmap LAB_IP
```
Replace `LAB_IP` with the target VM IP provided by your instructor or shown inside that target VM.

### D. Detect service versions
```bash
nmap -sV LAB_IP
```
Replace `LAB_IP` with the same approved target IP. `-sV` tries to identify service/version information on open ports.

### E. Check a specific port
```bash
nmap -p PORT_NUMBER LAB_IP
```
Replace `PORT_NUMBER` with the port number your task asks you to examine, and `LAB_IP` with the target VM IP.

### F. OS detection
```bash
sudo nmap -O LAB_IP
```
Replace `LAB_IP` with the authorized lab target. Enter the Kali password if prompted. OS detection is not always accurate or successful.

### G. Banner detection and save output
```bash
nmap -sV --script=banner LAB_IP
nmap -oN scan-results.txt LAB_IP
```
Replace `LAB_IP`. The first attempts to read banners; the second saves output to `scan-results.txt` in the current directory.

### H. Check Masscan
```bash
masscan --version
```
Masscan can generate high traffic. Run it only on the exact isolated lab target/rate specified by the instructor. Do not run broad or public-IP scans.

**Options:** `-sn` host discovery; `-sV` service/version detection; `-O` OS detection; `-p` select port; `-oN` save output.

---

## Lab 4 — Enumeration using NetBIOS, SNMP, and LDAP

**Aim:** Collect service information from an authorized training VM.

### A. Check which tools are installed
```bash
which nbtscan
which enum4linux
which snmpwalk
which snmp-check
which ldapsearch
```
A printed path means that tool is installed. No output may mean it is missing.

### B. NetBIOS
1. Get the target VM's IP from the lab sheet/instructor.
2. Run:
   ```bash
   nbtscan LAB_IP
   ```
3. Replace `LAB_IP` with the authorized target IP.
4. Record host/workgroup names shown.
5. If required, check enum4linux syntax with:
   ```bash
   enum4linux -h
   ```
   Use only the instructor's approved options.

### C. SNMP
1. Ask the instructor for the training VM's SNMP community string.
2. Run:
   ```bash
   snmpwalk -v2c -c LAB_COMMUNITY_STRING LAB_IP system
   ```
3. Replace `LAB_COMMUNITY_STRING` with the exact supplied string and `LAB_IP` with the lab target IP.
4. Record returned system information or the actual error.

### D. LDAP
1. Get the LDAP server's lab IP.
2. Run:
   ```bash
   ldapsearch -x -H ldap://LAB_IP -s base
   ```
3. Replace `LAB_IP` with that authorized LDAP server IP.
4. If the instructor supplies a base DN or credentials, use the exact values provided. Do not guess credentials.

**Ports to remember:** NetBIOS 137/138 UDP and 139 TCP; SNMP queries 161 UDP; LDAP 389 TCP; LDAPS commonly 636 TCP.

---

## Lab 5 — Password auditing with John the Ripper and Hydra

**Aim:** Understand dictionary attacks and brute-force attacks in an approved lab.

### A. Check tools
```bash
john --help
hydra -h
which john
which hydra
```

### B. John the Ripper with sample hashes
1. Use only the sample hash file supplied by your instructor.
2. List files:
   ```bash
   ls
   ```
3. Find the exact sample hash filename and wordlist path supplied by the instructor.
4. Run:
   ```bash
   john --wordlist=LAB_WORDLIST HASH_FILE
   john --show HASH_FILE
   ```
5. Replace `LAB_WORDLIST` with the actual wordlist path, such as a path shown in your lab sheet. Replace `HASH_FILE` with the actual sample hash filename. Do not type either placeholder literally.
6. The `--show` command displays results John has recovered for that file.

### C. Hydra
Hydra performs online login attempts and can lock accounts. Use only the isolated test account/target and the exact command taught by your instructor. Do not use it against real accounts or public services.

**Theory:** Dictionary attack tries words from a list. Brute force tries combinations. Defences include long unique passwords, MFA, rate limits, monitoring, and secure salted password hashing.

---

## Lab 6 — Safe Trojan/malware sample analysis

**Aim:** Inspect a harmless instructor-provided sample in a sandbox.

1. Use only the file supplied by your instructor. Do not download or execute random malware.
2. Open Terminal and navigate to the folder containing the sample. Use `ls` to list files.
3. Replace `SAMPLE_FILE` with the exact sample filename.
4. Run:
   ```bash
   file SAMPLE_FILE
   sha256sum SAMPLE_FILE
   strings SAMPLE_FILE
   ```
5. `file` reports file type; `sha256sum` computes a hash; `strings` displays readable strings.
6. Record the actual outputs. Do not execute the sample unless specifically directed to do so in a disposable, isolated VM.

**Terms:** Static analysis examines without running. Dynamic analysis observes execution in a sandbox. IoC means indicator of compromise.

---

## Lab 7 — SQL injection assessment using SQLmap on local DVWA

**Aim:** Test the deliberately vulnerable local DVWA app.

1. Confirm DVWA is running on your own lab VM.
2. Your notebook handout shows a local DVWA URL pattern. The Lab 12 sheet specifies `http://127.0.0.1/dvwa/`. Use that address only if DVWA is actually running locally there.
3. Open the local DVWA page in a browser and follow the class instructions to log in and set the required security level.
4. Check SQLmap:
   ```bash
   sqlmap --version
   sqlmap -h
   ```
5. Use the exact URL and any session details supplied in your lab exercise. Do not invent a cookie or use a URL from a real website.
6. Run only the instructor-approved SQLmap test against local DVWA and record the actual output.

**Option meanings:** `-u` target URL; `--batch` accept default prompts; `-D` select a database; `-T` select a table; `-C` select columns. Use database-enumeration options only if they are explicitly required for your authorized local exercise.

**Defence:** Parameterized queries/prepared statements and least-privileged database accounts.

---

## Lab 8 — ARP spoofing and traffic sniffing with Ettercap

**Aim:** Understand ARP and packet observation in an isolated network.

1. Confirm the instructor-approved lab VMs are on an isolated internal/host-only network.
2. In Kali, run:
   ```bash
   ip addr
   ip route
   ```
3. Find the interface connected to the lab network. Replace `LAB_INTERFACE` only if the instructor's command requires an interface; use the exact interface name shown by `ip addr`.
4. Check Ettercap:
   ```bash
   ettercap --version
   ettercap -h
   ```
5. Ask the instructor which two test VMs are the client and gateway. Do not choose devices from your home/college network.
6. Follow the instructor's exact controlled demonstration, observe only test traffic, stop the exercise, and restore network settings.

The notebook refers to selecting two targets and starting ARP poisoning. A copy-paste poisoning command is not included because incorrect targets can intercept or disrupt other people's traffic.

**Theory:** ARP maps local IPv4 addresses to MAC addresses. ARP spoofing sends forged ARP messages. Packet sniffing observes network packets.

---

## Lab 9 — Steganography with Steghide and Python

**Aim:** Hide and recover a harmless message in an image.

### A. Check Steghide
```bash
steghide --version
```
If unavailable, note that the tool is missing. Install it only if permitted by the instructor.

### B. Create a text file
1. In Terminal:
   ```bash
   nano secret.txt
   ```
2. Type a harmless test message.
3. Save with `Ctrl+O`, press `Enter`, then exit with `Ctrl+X`.
4. Put the image supplied for the lab in the same folder. Check its exact filename using `ls`. The example below assumes the image is named `cover.jpg`.

### C. Embed the message
```bash
steghide embed -cf cover.jpg -ef secret.txt
```
`cover.jpg` is the cover image; `secret.txt` is the file to hide. If your image has a different name, replace `cover.jpg` with its exact filename. Enter a lab passphrase when prompted.

### D. Extract
```bash
steghide extract -sf cover.jpg
```
`-sf` identifies the image containing the hidden data. Replace the filename if necessary. Enter the same passphrase, then run `ls` to check whether the file was extracted.

### E. Python script
The photos mention `steganography.py` and `extract.py`, but their source code was not fully visible. Do not assume those files exist. Check:
```bash
ls
python3 --version
```
Use the scripts supplied by the instructor and their documented arguments.

---

## Lab 10 — Intrusion detection/prevention with Suricata

**Aim:** Install/check Suricata, configure a local alert rule, and inspect logs.

### A. Check installation
```bash
suricata -V
```
If missing and installation is allowed:
```bash
sudo apt update
sudo apt install suricata
```

### B. Identify the interface
```bash
ip addr
ip route
```
Read the output and identify the interface connected to your lab network. It may be `eth0`, `ens33`, or another name. Do not guess it.

### C. Test configuration
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```
Read the output for configuration errors.

### D. Start/check service
```bash
sudo systemctl start suricata
sudo systemctl status suricata
```

### E. Add an alert-only test rule
1. The notebook uses `/etc/suricata/rules/local.rules`. Check whether it exists:
   ```bash
   ls /etc/suricata/rules/
   ```
2. If it exists and the instructor permits editing, back it up:
   ```bash
   sudo cp /etc/suricata/rules/local.rules /etc/suricata/rules/local.rules.backup
   ```
3. Open it:
   ```bash
   sudo nano /etc/suricata/rules/local.rules
   ```
4. Add this controlled lab alert rule:
   ```text
   alert icmp any any -> any any (msg:"ICMP test detected"; sid:1000002; rev:1;)
   ```
5. Save with `Ctrl+O`, press `Enter`, and exit with `Ctrl+X`.
6. Make sure the main configuration loads `local.rules`. If not, follow your instructor's setup rather than editing unrelated configuration blindly.

### F. Test and view logs
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl restart suricata
ls /var/log/suricata/
sudo tail -f /var/log/suricata/fast.log
```
Press `Ctrl+C` to stop viewing the live log. Alerts may not appear until the service, rule, and permitted test traffic are all configured correctly. Record only the output actually seen.

**Remember:** IDS detects and alerts. IPS can block when configured inline. The example rule only alerts; it does not block.

---

## Lab 11 — Safe Python keyboard-input demonstration

**Aim:** Demonstrate input handling without recording system-wide keystrokes.

1. Check Python:
   ```bash
   python3 --version
   ```
2. Create a file:
   ```bash
   nano keyboard_demo.py
   ```
3. Paste:
   ```python
   text = input("Type a sample message for the lab demo: ")
   print("Characters entered:", len(text))
   print("Demo finished. No keys were recorded in the background.")
   ```
4. Save with `Ctrl+O`, press `Enter`, then exit with `Ctrl+X`.
5. Run:
   ```bash
   python3 keyboard_demo.py
   ```
6. Type a harmless sample message and read the result.

The notebook mentions `pynput` and a listener, but a covert keylogger could capture passwords and private messages. This safe version records only text intentionally entered into its prompt. For a viva, explain that key listeners detect keyboard events and require consent and controlled use.

---

## Lab 12 — Penetration testing practical (from the supplied handout)

**Platform:** Kali Linux  
**Target shown:** DVWA (localhost), `127.0.0.1`  
**URL printed:** `http://127.0.0.1/dvwa/`

**Aim:** Assess common web application vulnerabilities in the intentionally vulnerable local DVWA application and document findings, impact, and controls.

### A. Planning and scoping
1. Confirm the target is the DVWA instance running on your own Kali/lab VM.
2. The handout says external public IPs and other users' systems are out of scope.
3. Write the target and scope in your report: local DVWA only, plus the modules your instructor asks you to test.

### B. Reconnaissance and scanning
1. Confirm the local app address from the handout: `http://127.0.0.1/dvwa/`.
2. Check Nmap:
   ```bash
   nmap --version
   ```
3. If instructed to scan localhost, run:
   ```bash
   nmap 127.0.0.1
   ```
4. Record actual open ports and service/version details. If the handout specifically asks for service versions:
   ```bash
   nmap -sV 127.0.0.1
   ```
These commands scan your own machine, not a public target.

### C. Open DVWA
1. Open Firefox in Kali.
2. Navigate to `http://127.0.0.1/dvwa/`.
3. If the page does not load, DVWA may not be installed/running at that address. Do not substitute a public website. Ask the instructor or use the provided lab VM.
4. Log in with the lab credentials supplied by your instructor. Do not guess or reuse personal passwords.
5. Set the DVWA security level exactly as the practical requires and record it. Findings depend on that setting.

### D. Assess the listed vulnerability areas
Use the relevant DVWA module, follow the lab worksheet's benign test inputs, and record the actual result. Do not test outside local DVWA.

| Area in handout | What to look for | Defensive control to write |
|---|---|---|
| SQL injection | Whether unsafe input affects database queries | Parameterized queries/prepared statements; least-privileged DB account |
| Cross-site scripting (XSS) | Whether untrusted input is returned as executable browser content | Context-appropriate output encoding; input validation |
| Command injection | Whether input is passed to an operating-system command | Avoid shell commands where possible; safe APIs; strict allow-lists |
| File inclusion | Whether unsafe file paths can load unintended files | Allow-list files; canonicalize/validate paths; least privilege |
| File upload | Whether unsafe files can be uploaded or executed | Allow-list file types; verify content; rename uploads; store outside web root |
| CSRF | Whether a sensitive action lacks request-forgery protections | Anti-CSRF tokens; SameSite cookies; re-authentication for sensitive actions |
| Authentication / information disclosure | Whether authentication or error messages reveal sensitive details | Secure session settings; generic errors; least privilege; avoid exposing secrets |

The handout's table describes risks and controls, but it does **not** supply a universal exploit command for every module. Use only the specific inputs and steps provided by your instructor. Do not claim a vulnerability was found unless you observed evidence.

### E. Record evidence and assess risk
For each module, record:
- Module name and DVWA security level
- Test performed
- Actual result or error
- Evidence (screenshot or copied output, avoiding real credentials/secrets)
- Potential impact
- Recommended control

The handout says findings should be supported by evidence and risk should reflect the configured security level. A test that did not confirm a vulnerability should be recorded as inconclusive/not confirmed, not as a confirmed finding.

### F. Report format
1. Title and date
2. Aim and scope
3. Target: local DVWA only
4. Tools used: browser, Nmap, and only the tools actually used
5. Methodology: planning → reconnaissance → scanning → enumeration → vulnerability assessment → validation → reporting
6. Findings table: vulnerability, evidence, impact, risk, defensive recommendation
7. Overall risk and limitations
8. Conclusion

**The handout's final reminder:** verify results against your practical work before submission. It specifically expects real evidence, not example results presented as if they happened.

---

## Lab 13 — Windows system hardening with PowerShell

**Aim:** Check Windows services and firewall configuration on your own/test Windows VM.

Open PowerShell as Administrator only if required by the lab.

### A. Check Windows version
```powershell
winver
```
This opens a Windows version dialog; it does not print a version in the terminal.

### B. List services
```powershell
Get-Service
```
This lists Windows services and their statuses.

### C. Stop and verify Print Spooler
```powershell
Stop-Service -Name Spooler
Get-Service -Name Spooler
```
Do this only on the test VM when required; it may affect printing.

### D. View firewall profiles
```powershell
Get-NetFirewallProfile
```

### E. Create an inbound TCP block rule for test port 8080
```powershell
New-NetFirewallRule -DisplayName "Block Test Port 8080" -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Block
```
This creates a named inbound block rule on the test machine.

### F. Verify and remove the rule
```powershell
Get-NetFirewallRule -DisplayName "Block Test Port 8080"
Remove-NetFirewallRule -DisplayName "Block Test Port 8080"
```
The first checks the rule; the second removes the test rule. Do not leave test rules behind unless instructed.

**Hardening means:** reduce attack surface, disable unnecessary services, patch systems, restrict access, and configure the firewall carefully.

---

## Quick tool map

| Lab | Main tools |
|---|---|
| 1 | VMware/VirtualBox; `ip addr`, `ip route`, `ping` |
| 2 | `whois`, `nslookup`; Google and Shodan in browser |
| 3 | `nmap`, `masscan` |
| 4 | `nbtscan`, `enum4linux`, `snmpwalk`, `snmp-check`, `ldapsearch` |
| 5 | `john`, `hydra` |
| 6 | `file`, `sha256sum`, `strings` |
| 7 | `sqlmap` against local DVWA only |
| 8 | `ip addr`, `ip route`, `ettercap` |
| 9 | `steghide`, `python3`, `nano`, `ls` |
| 10 | `suricata`, `systemctl`, `nano`, `tail` |
| 11 | `python3`, `nano` |
| 12 | Local DVWA, browser, Nmap, report writing |
| 13 | PowerShell: `Get-Service`, `Stop-Service`, `Get-NetFirewallProfile`, `New-NetFirewallRule`, `Get-NetFirewallRule`, `Remove-NetFirewallRule` |

## Exam answer template

For each practical, write:
1. Aim
2. Requirements/tools
3. Theory (2–4 lines)
4. Procedure (numbered steps)
5. Commands and what each one does
6. Observation/output (only what actually appeared)
7. Result
8. Precautions/defences

**If a tool is missing:** run the availability check, record the actual `command not found`/error, and write the intended procedure if asked. Do not claim the practical succeeded if it did not.
