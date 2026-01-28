# Incident Response: SSH Brute Force Triage & Mitigation
**Author:** Brian Zeilenga  
**Focus:** Security Operations (SecOps), Incident Response, System Hardening  

---

## 1. Executive Summary
This project demonstrates the end-to-end Incident Response (IR) lifecycle following the **NIST SP 800-61 Rev. 3** framework. I simulated a credential-stuffing (brute force) attack against a Linux-based asset, analyzed the resulting telemetry in the system authentication logs, and implemented automated containment and eradication measures.

## 2. Environment & Toolset
* **OS:** Ubuntu 24.04 LTS  
* **Services:** OpenSSH Server  
* **Defense Tools:** UFW (Uncomplicated Firewall)  
* **Analysis Tools:** Linux CLI (tail, grep)  

## 3. The Simulation (Attack Vector)
To validate the detection capabilities of the system, a Bash-based simulation script was utilized to generate 10+ failed authentication events. The system successfully logged these events in /var/log/auth.log with the expected 'Failed password' string, confirming the visibility of the attack vector.
<div> <img width="732" height="232" alt="image" src="https://github.com/user-attachments/assets/34450ae5-9555-4a02-b34d-9a5b336baea4" />
</div>
<img width="1451" height="349" alt="image" src="https://github.com/user-attachments/assets/0d11d928-996d-4bca-b9d4-f068c8ee138a" />


## 4. Incident Response Phases
### Detection and Analysis 
Upon reviewing system logs, I identified a high volume of failed authentication attempts.

Log Source: /var/log/auth.log

Findings: The logs indicated multiple Failed password for root entries originating from a single source. This pattern is consistent with a brute force attack rather than a forgotten password scenario.


### Containment
Manual Intervention: Immediately blocked the offending IP address using the Host-based firewall

Command: sudo ufw deny from 127.0.0.1 to any

<img width="529" height="172" alt="image" src="https://github.com/user-attachments/assets/7a4d0f8d-a509-45f4-81ee-3c22489ff7ba" />

### Eradication - System Hardening
This phase of the response involved hardening the attack surface to ensure the vulnerability (password-based authentication) was removed.

1. **Root Login Disablement:** Updated SSH configuration to `PermitRootLogin no` to prevent direct targeting of the administrative account.
2. **Principle of Least Privilege (Accounts)** Just because you can, you shouldn't give every user (or service) high-level access. Remove Unused Accounts: Audit /etc/passwd and delete accounts for former users or services you don't use. Enforce Sudo: Ensure no one logs in as root. Force users to log in with a standard account and use sudo for administrative tasks. This creates an audit trail of who did what.

## 6. Project Impact & Skills Applied
This project demonstrates a proactive security posture, where lessons learned can be implemented effectively.

* **Tool Proficiency:** Linux, OpenSSH, UFW.
* **Analytical Thinking:** Log correlation and pattern recognition.
* **Process Discipline:** Adherence to the NIST Incident Response lifecycle.

---

### ⚓ Multi-Career Perspective
In both critical care nursing and fire/rescue, we rely on **Standard Operating Procedures (SOPs)** to ensure success under pressure. This project is a digital SOP. It demonstrates that whether I am stabilizing a patient or a network, I follow a disciplined, repeatable process to ensure the "safety of the scene" and the integrity of the mission.

---

#### Bash Script Utilized:
```bash
#!/bin/bash

# Target: Your local machine
TARGET="127.0.0.1"
USER_TO_TARGET="root"
# A small list of common passwords to simulate the attack
PASSWORDS=("password123" "admin" "123456" "qwerty" "letmein" "security")

echo "Starting Simulated Brute Force Attack on $TARGET..."
echo "Monitoring /var/log/auth.log for failures..."

for PASS in "${PASSWORDS[@]}"
do
    echo "Attempting login for $USER_TO_TARGET with password: $PASS"
    # sshpass allows us to provide a password via script to trigger the failure log
    # We use -o ConnectTimeout to keep the script moving quickly
    sshpass -p "$PASS" ssh -o ConnectTimeout=2 -o StrictHostKeyChecking=no $USER_TO_TARGET@$TARGET "exit" 2>/dev/null
    sleep 1
done

echo "Attack simulation complete. Check your logs with: tail -n 20 /var/log/auth.log"
