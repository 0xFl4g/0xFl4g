# Homelab security engineering — 0xFl4g

I run a five-host homelab as a small production environment: a SOC on Elastic Security, detection-as-code
with tests and CI, and the infrastructure underneath it. I treat it like work: changes are planned and
reviewed, incidents get postmortems, and the numbers below are measured, not estimated.

**Start here:**
- **[homelab-detections](https://github.com/0xFl4g/homelab-detections):** 86 Elastic detection rules (78 mapped
  to MITRE ATT&CK v20), Sigma rules, a daily canary of 119 synthetic-event checks, and CI that lints, validates
  and leak-scans every push.
- **[Incident write-ups](https://github.com/0xFl4g/homelab-detections#incident-write-ups):** five blameless
  postmortems and engineering notes. Start with the [credential leak through Docker exec command
  lines](https://github.com/0xFl4g/homelab-detections/blob/main/writeups/2026-09-28-docker-exec-credential-leak.md)
  or the [Atomic Red Team coverage
  test](https://github.com/0xFl4g/homelab-detections/blob/main/writeups/2026-10-18-atomic-red-team-linux-coverage.md).

## Security operations

- **SIEM:** single-node Elastic Security holding about 72 million log documents: firewall, Suricata IDS, DNS,
  NetFlow, Docker events, host syslog and auth, osquery, a honeypot, and an endpoint lab. Migrated from Wazuh in
  September 2026: custom rules ported to Elastic and covered by the canary.
- **Detection engineering:** rules as code in Git, deployed by script, ATT&CK-tagged from one validated map.
  A daily canary injects synthetic events into the live SIEM and fails if any rule stops alerting.
- **Adversary emulation:** an isolated lab VLAN with an Ubuntu target running Elastic Defend and auditd.
  Ran 216 Atomic Red Team tests, measured 45% undetected, found a Linux telemetry gap (no file-modification
  events), and closed it with 12 verified rules.
- **Automated response:** an alert-driven responder adds attacking addresses to a firewall block list through
  the OPNsense API, with a 24-hour expiry and the lab excluded.
- **Threat intelligence and anomaly detection:** abuse.ch and CrowdSec feeds enrich logs in the pipeline and
  are correlated against NetFlow; 29 machine-learning jobs and entity analytics run on top.
- **Vulnerability management:** an assess, remediate, re-scan cycle with Docker Bench, Lynis and Greenbone
  (2026-04: CIS Docker score −11 → −5); a CVE and version sweep in October 2026 patched the firewall (six
  published advisories), the KVM device and five pinned services.

## Infrastructure

- **Platform:** 144 running containers on an Unraid server with Docker Compose, plus 77 services kept
  on-demand. Eight segmented reverse-proxy networks, Traefik with a CrowdSec bouncer, Authentik SSO in front
  of internal services, encrypted storage.
- **Network:** OPNsense firewall with Suricata and CrowdSec, an isolated lab VLAN, a WireGuard tunnel to an
  offsite NAS, and remote access through three Tailscale subnet routers for redundancy.
- **Observability:** Prometheus and Grafana with 44 alert rules, self-hosted Sentry for error tracking.
- **Backups:** nightly database dumps that are verified before they replace the previous copy, then encrypted
  offsite backups over WireGuard.
- **Practices:** the whole lab lives in a Git repo with a wiki; a pre-commit hook blocks secrets; scripts ship
  with self-tests; upgrades follow written checklists.

## Skills

| For SOC / detection roles | For DevOps / platform roles |
|---|---|
| Elastic Security, KQL and EQL, Sigma | Docker and Compose at 140+ containers |
| MITRE ATT&CK mapping and coverage testing | Linux across Slackware (Unraid), FreeBSD (OPNsense), Arch ARM |
| Suricata IDS, CrowdSec, threat-intel enrichment | Networking: VLANs, NAT, WireGuard, Tailscale |
| Alert triage, tuning and false-positive analysis | GitHub Actions CI, supply-chain pinning |
| Incident response and postmortem writing | Prometheus, Grafana, Sentry |
| Atomic Red Team adversary emulation | Python and Bash, with tests |

**Scope, honestly:** this is homelab scale, on a single-node SIEM. Windows and Active Directory coverage is the
next build: a domain controller and workstation in the lab VLAN, attacked and detected the same way.
