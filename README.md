# Penetration Test Walkthrough: Empire: Breakout

This repository documents a full penetration test of the "Empire: Breakout" vulnerable machine from VulnHub. The report covers the entire process, from initial network reconnaissance to gaining root-level access through privilege escalation.

**Environment:**
*   **Target Machine:** Empire: Breakout (from VulnHub)
*   **Attacker Machine:** Kali Linux

---

## Phase 1: Reconnaissance and Enumeration

The initial phase focused on identifying the target machine and enumerating its open services.

### 1.1. Host Discovery
The target machine's IP address was identified on the local network using `netdiscover`. A simple `ping` was used to confirm connectivity.

### 1.2. Port Scanning
An aggressive `nmap` scan (`-A -sV`) was performed to discover open ports and enumerate service versions.

**Key Findings:**
*   **Port 80 (HTTP):** Apache httpd 2.4.51
*   **Ports 139/445 (SMB ):** Samba smbd 4.6.2
*   **Port 10000 & 20000 (HTTP):** Webmin (MiniServ 1.981 & 1.830)

*(See `nmap-scan.png` for the full scan results.)*

---

## Phase 2: Initial Access (RCE)

This phase covers the steps taken to gain an initial foothold on the target machine.

### 2.1. Web Enumeration (Port 80)
The web page on port 80 contained a hidden comment with a string of characters. This was identified as a Brainfuck cipher. Decoding the cipher revealed a potential password.

*(See `brainfuck-cipher.png` for the cipher and its decoded output.)*

### 2.2. SMB Enumeration
`enum4linux` was used to enumerate the SMB service, which revealed a valid username: `cyber`.

### 2.3. Webmin Access & Reverse Shell
The discovered credentials (`cyber` and the decoded password) were used to log into the Webmin service on port 20000. The Webmin interface included a command shell utility, which was used to execute a `bash` reverse shell payload. This successfully established a connection back to a `netcat` listener on the attacker machine, achieving Remote Code Execution (RCE).

*(See `webmin-access-and-rce.png` for a visual of the Webmin login and reverse shell execution.)*

---

## Phase 3: Privilege Escalation (LPE)

With user-level access, the next goal was to escalate privileges to root.

### 3.1. Internal Enumeration
While exploring the file system as the `cyber` user, the `/var/backups` directory was found to contain a file named `.old_pass.bak`. The `cyber` user did not have permission to read this file.

### 3.2. SUID Abuse (Tar)
Further enumeration revealed that the `tar` binary had the SUID bit set, meaning it could be run with the permissions of its owner (root). This is a common privilege escalation vector.

### 3.3. Exploitation
The `tar` binary's capabilities were exploited to read the protected `.old_pass.bak` file. This was done by using `tar` to create an archive of the file and then extracting it, which bypasses the standard file permissions. The file contained the root user's password.

### 3.4. Root Access
The `su` command was used with the extracted password to switch to the `root` user. The `whoami` command confirmed that root access was successfully obtained. The final root flag was then retrieved from `/root/root.txt`.

*(See `lpe-and-root.png` for a summary of the privilege escalation process.)*
