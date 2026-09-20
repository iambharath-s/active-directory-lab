# Security Policy and Responsible Use

## Intended Use

corp-lab is an **isolated training environment** designed for:

- CEH (Certified Ethical Hacker)/ OSCP exam preparation
- Penetration testing skill development
- Classroom demonstrations in authorized cybersecurity courses
- Personal practice in a fully isolated VirtualBox network

## What This Lab Is NOT

- A tool for attacking systems you do not own
- A production environment template (every setting is deliberately insecure)
- Suitable for deployment on the internet or any shared network

## Network Isolation

The lab uses VirtualBox Host-Only networking exclusively.
No attack traffic from this lab reaches the internet or other networks by design.
Do not add a bridged or NAT adapter to the DC01 VM.

## Legal Disclaimer

By using this repository, you agree that:

1. You will only use this lab against the virtual machines created by this project
2. You will not use techniques demonstrated here against unauthorized systems
3. The author bears no responsibility for any misuse of the content in this repository
4. All activities conducted with this lab are your sole legal and ethical responsibility

Unauthorized access to computer systems is illegal under the Computer Fraud and Abuse Act (US),
the Computer Misuse Act (UK), the Information Technology Act (India), and similar laws worldwide.

## Reporting Issues

If you find a bug in a script or documentation error, open a GitHub Issue.
Do not open issues for questions about attacking unauthorized systems.

## Safe Distribution

If you distribute this lab as OVA files to students:
- Ensure all students understand the legal and ethical boundaries above
- Only distribute within a closed classroom or lab environment
- Remove all lab VMs after the course or training session ends
