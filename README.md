# wazuh-soc-lab
Home SOC Lab: Wazuh SIEM deployment monitoring a Windows endpoint via Sysmon
# Home SOC Lab: Wazuh SIEM Deployment

A self-hosted Security Operations Center (SOC) lab built to gain hands on experience with SIEM deployment, endpoint monitoring, and security event analysis. Built as part of my preparation for entry level SOC Analyst and IT Security roles.

## Summary

Deployed and configured a Wazuh SIEM in a multi VM VirtualBox lab to monitor a Windows endpoint via Sysmon telemetry, resolving real infrastructure issues (disk capacity, LVM partition resizing, network configuration) encountered during the build.

## Architecture

| Component | Role | Details |
|---|---|---|
| **Wazuh Manager/Indexer/Dashboard** | SIEM core | Ubuntu Server 22.04 LTS, Wazuh 4.9.2, bridged networking |
| **Windows 11 Endpoint** | Monitored host | Sysmon (SwiftOnSecurity config) + Wazuh agent |
| **Kali Linux** | Attacker box | Used to generate simulated malicious traffic |

All VMs run in VirtualBox on a shared internal network (`soclab`), with the Wazuh Manager VM bridged to the host network for dashboard access.

## What I Built

- Deployed Wazuh 4.9.2 (indexer, manager, and dashboard) using the official all in one installation script
- Installed and configured Sysmon on a Windows 11 VM using the SwiftOnSecurity configuration for detailed process, network, and file telemetry
- Deployed the Wazuh agent on the Windows endpoint and verified live event ingestion
- Enabled File Integrity Monitoring (FIM) to track file/directory changes
- Ran Security Configuration Assessment (SCA) scans against the CIS Microsoft Windows 11 Enterprise Benchmark
- Verified full event flow from Sysmon to Windows Event Log to Wazuh Agent to Wazuh Manager to Indexer to Dashboard (424+ events ingested and categorized by rule group)

## Troubleshooting and Problem-Solving

Building this lab surfaced a number of real infrastructure problems. Working through them was as valuable as the deployment itself:

- **Disk capacity failure mid-install**: Diagnosed a full disk (`df -h`) that caused the Wazuh dashboard installation to fail and corrupted a Python bytecode cache, breaking `cloud-init` on boot. Resolved by freeing space, resizing the VM's virtual disk via `VBoxManage`, and growing the underlying partition and filesystem (`growpart`, `resize2fs`) without a full VM rebuild.
- **Networking (NAT vs. Bridged)**: Diagnosed a connectivity issue where the host machine could not reach the VM's dashboard or SSH despite the services running correctly. Traced the cause to VirtualBox NAT isolating the guest, and resolved it by switching to bridged networking.
- **OpenSearch read only index locks**: After the disk full event tripped OpenSearch's flood-stage watermark, indices were locked read only. Cleared the block via the OpenSearch REST API (`PUT /_all/_settings`).
- **Platform compatibility**: An initial deployment attempt on Ubuntu 26.04 failed due to Wazuh package incompatibility (missing manager binaries after install). Traced the issue to unsupported OS version, then rebuilt the manager VM on Ubuntu 22.04 LTS (an officially supported platform), resolving it cleanly.

## Skills Demonstrated

`Linux system administration` `SIEM deployment and configuration` `VM networking (NAT vs. Bridged)` `Disk/LVM management` `Endpoint telemetry (Sysmon)` `REST API troubleshooting` `Security compliance scanning (CIS Benchmarks)` `VirtualBox lab architecture`

## Screenshots

**Dashboard Overview**
![Dashboard Overview](dashboard-overview.png)

**Windows Agent Connected**
![Agent Connected](agent-connected.png)

**Live Event Ingestion (424 events, categorized by rule group)**
![Event Ingestion](event-ingestion.png)

**Sysmon Telemetry (Windows Event Viewer)**
![Sysmon Events](sysmon-events.png)

**Detected Alert: Simulated Nmap Scan from Kali**
![Nmap Scan Alert](nmap-scan-alert.png)

## Next Steps

- [ ] Write custom detection rules mapped to MITRE ATT&CK techniques
- [ ] Simulate additional attack scenarios (brute-force login attempts, EICAR test file detection)
- [ ] Produce a sample incident report based on a detected alert
- [ ] Add a Linux endpoint to the monitored fleet

## About

Built by Darius Mills as part of hands on preparation for SOC Analyst / IT Security roles.
[Resume & Portfolio](https://github.com/dmills6/Darius-Mills-Resume)
