# WordPress Web Forensics — Hack The Box Sherlock

## Project Overview

In this Hack The Box Sherlock lab, I conducted a Digital Forensics and Incident Response (DFIR) investigation into a compromised WordPress server. Using Splunk SIEM, I analyzed Apache2 access logs to identify suspicious activity, reconstruct the attack timeline, and determine how the attacker gained unauthorized access.

The investigation involved identifying reconnaissance activity, analyzing brute-force login attempts, and investigating a PHP web shell used to maintain access to the compromised server.

## Lab Environment & Tools

- **Platform:** Hack The Box — Sherlock
- **Investigation:** WordPress Web Forensics
- **Difficulty:** Medium
- **Operating System:** Kali Linux
- **SIEM:** Splunk Enterprise
- **Log Source:** Apache2 Access Logs
- **Skills:** Log Analysis, Threat Detection, Digital Forensics, Incident Response, Web Security

## Investigation Findings

### Task 1 — Identifying the Attacker's IP Address

I began by searching the Apache access logs for WPScan activity to identify the source of the reconnaissance.

**Splunk Query:**
```spl
index=* "wpscan"
```

The search returned multiple requests associated with WPScan, allowing me to identify the attacker's originating IP address.

**Finding:** `203.101.190.116`

![Attacker IP and WPScan Activity](screenshots/screenshot1.png)

### Task 2 — Identifying Initial Reconnaissance

Next, I sorted the WPScan events chronologically to determine when the attacker first began scanning the WordPress server.

**Splunk Query:**
```spl
index=* "wpscan"
| sort _time
```

The earliest recorded activity was a HEAD request to the WordPress directory.

**Finding:** `2024-08-28 03:15:23 UTC`

![Initial Reconnaissance](screenshots/screenshot2.png)

### Task 3 — Identifying the WPScan Version

By examining the user-agent field associated with the reconnaissance requests, I identified the specific version of WPScan used during the attack.

**Finding:** `WPScan v3.8.25`

![WPScan Version](screenshots/screenshot3.png)

### Task 4 — Investigating the Brute-Force Attack

The attacker targeted the WordPress authentication page, `wp-login.php`, with repeated POST requests.

**Splunk Query:**
```spl
index=* "wpscan" uri_path="*wp-login.php"
```

I analyzed the login requests and identified repeated HTTP 200 responses followed by an HTTP 302 redirect, indicating a likely successful authentication attempt in the context of this investigation.

**Findings:**

- First unsuccessful login attempt: `03:50:33 UTC`
- Successful login attempt: `03:50:40 UTC`
- Brute-force duration: **7 seconds**

![WordPress Login Analysis](screenshots/screenshot4.png)

### Task 5 — Identifying the Compromised WordPress Theme

After investigating the authentication activity, I searched for requests involving WordPress theme files.

**Splunk Query:**
```spl
index=* uri="*themes*"
```

The logs showed activity involving the WordPress theme editor and requests to the Twenty Twenty-Four theme directory.

**Finding:** `twentytwentyfour`

![Compromised WordPress Theme](screenshots/screenshot5.png)

### Task 6 — Identifying the PHP Web Shell

I continued investigating requests associated with the compromised WordPress theme to identify the suspicious PHP file.

**Splunk Query:**
```spl
index=* uri="*twentytwentyfour*"
```

The logs revealed repeated requests to a suspicious PHP file within the theme's patterns directory.

**Finding:** `hidden-404.php`

**File Path:**
```text
/wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-404.php
```

![PHP Web Shell](screenshots/screenshot6.png)

### Task 7 — Identifying Initial Web Shell Activity

I filtered the logs for POST requests made to the suspicious PHP file to investigate potential remote command execution.

**Splunk Query:**
```spl
index=* uri="*hidden-404.php*" method="POST"
```

The search returned four POST requests occurring within a short period, suggesting the attacker was interacting with the web shell to execute commands.

**Finding:** `2024-08-28 04:03:51 UTC`

The access logs indicate suspicious web shell activity, although the actual commands executed are not visible in the provided evidence.

![First Suspected Command Execution](screenshots/screenshot7.png)

### Task 8 — Identifying the Web Shell User-Agent

Finally, I examined the user-agent field associated with the suspicious PHP requests to identify the browser used to access the web shell.

**Finding:**
```text
Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:129.0) Gecko/20100101 Firefox/129.0
```

This indicates that the requests advertised a Firefox 129.0 browser running on Windows 10.

![Web Shell User-Agent](screenshots/screenshot8.png)

---

## Incident Summary

During this investigation, I analyzed Apache2 web server logs using Splunk SIEM to determine how an attacker compromised a WordPress environment.

The investigation revealed that the attacker, originating from **203.101.190.116**, initiated reconnaissance using WPScan v3.8.25 on August 28, 2024, at 03:15:23 UTC.

Following reconnaissance, the attacker performed a brute-force attack against the WordPress login page, `wp-login.php`. The attack lasted approximately seven seconds before the attacker obtained valid credentials and gained access to the WordPress administration panel.

After gaining access, the attacker targeted the `twentytwentyfour` WordPress theme and modified a PHP file named `hidden-404.php`, using it as a web shell entry point to maintain access to the compromised server.

Further investigation identified multiple POST requests to the suspicious PHP file, beginning at 04:03:51 UTC. These requests indicated possible remote command execution through the web shell.

### Attack Timeline

| Timestamp (UTC) | Activity |
|---|---|
| 03:15:23 | Initial reconnaissance using WPScan |
| 03:50:33 | WordPress brute-force attack begins |
| 03:50:40 | Successful authentication indicated |
| After authentication | WordPress theme modification |
| 04:03:51 | First suspected web shell command execution |
| 04:05:26 | Additional web shell activity detected |

### Indicators of Compromise (IOCs)

| Indicator | Value |
|---|---|
| Attacker IP | `203.101.190.116` |
| Reconnaissance tool | WPScan v3.8.25 |
| Targeted login page | `/wordpress/wp-login.php` |
| Compromised theme | `twentytwentyfour` |
| Suspected web shell | `hidden-404.php` |
| Web shell access method | HTTP POST |
| Web shell user-agent | Firefox/129.0 |

## Security Recommendations

Based on the investigation, the following security measures could help prevent similar incidents:

- Implement multi-factor authentication (MFA) for WordPress administrator accounts.
- Configure login rate limiting to reduce brute-force attack attempts.
- Restrict access to the WordPress theme editor.
- Monitor web server logs for unusual authentication activity and suspicious PHP requests.
- Keep WordPress, themes, and plugins updated.
- Implement file integrity monitoring to detect unauthorized modifications to website files.

## Conclusion

This investigation provided hands-on experience analyzing a compromised WordPress environment using Splunk SIEM. I successfully identified the attacker's IP address, reconstructed the reconnaissance and brute-force activity, and investigated the PHP web shell used to maintain access.

Through this lab, I strengthened my skills in **security log analysis, incident investigation, threat detection, timeline reconstruction, and digital forensics**.

The investigation also highlighted the importance of monitoring authentication attempts, securing administrator access, and detecting unauthorized modifications to web application files.

**Platform:** Hack The Box  
**Lab:** WordPress Web Forensics  
**Category:** Digital Forensics & Incident Response (DFIR)