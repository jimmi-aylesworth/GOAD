---
title: Standard Operator Playbook
context: HTB · GOAD · OSCP+ · General Penetration Test
audience: practitioners working authorized engagements, CTF/lab environments, and certification exams
status: living document
placeholder_domain: target.example (RFC 2606/6761 reserved)
framework: PTES-aligned · MITRE ATT&CK referenced
tags: [playbook, methodology, htb, goad, oscp, pentest, red-team]
---

# 🎯 Standard Operator Playbook

> [!brief] What this is
> One repeatable methodology for four contexts: **Hack The Box (HTB)**, **Game of Active Directory (GOAD)**, the **OSCP+ exam**, and **general penetration tests (PT)**. The *spine* — the phase-by-phase operator loop — is identical across all four. What changes is the **lane**: scope, OPSEC posture, allowed tooling, proof requirements, time-box, and reporting. Each phase below states the standard action, then calls out where the lanes diverge.
>
> This document is the connective tissue for the field manual — it ties the individual tool and technique cheat sheets together into a single flow. Where a step needs command-level detail, it points you to the relevant chapter rather than repeating it.

> [!warning] Scope is the first control, not an afterthought
> Every technique here assumes you are operating **inside an authorized boundary**: a box you own on HTB, your isolated GOAD lab, your OSCP+ exam target set, or a client engagement with a **signed authorization / Rules of Engagement (ROE)**. Acting outside that boundary is not pentesting — it is an offense. Phase 0 exists to make the boundary explicit before any packet leaves your host.

## 📑 Contents

- [0 · How to use this playbook](#0--how-to-use-this-playbook)
- [1 · The four lanes at a glance](#1--the-four-lanes-at-a-glance)
- [2 · The operator loop](#2--the-operator-loop)
- [Phase 0 · Pre-engagement & scope control](#phase-0--pre-engagement--scope-control)
- [Phase 1 · Recon & enumeration](#phase-1--recon--enumeration)
- [Phase 2 · Initial access / foothold](#phase-2--initial-access--foothold)
- [Phase 3 · Stabilize & local situational awareness](#phase-3--stabilize--local-situational-awareness)
- [Phase 4 · Privilege escalation](#phase-4--privilege-escalation)
- [Phase 5 · Credential access & looting](#phase-5--credential-access--looting)
- [Phase 6 · Active Directory attack path](#phase-6--active-directory-attack-path)
- [Phase 7 · Lateral movement & pivoting](#phase-7--lateral-movement--pivoting)
- [Phase 8 · Domain & forest dominance](#phase-8--domain--forest-dominance)
- [Phase 9 · Persistence (lane-gated)](#phase-9--persistence-lane-gated)
- [Phase 10 · Proof, evidence & collection](#phase-10--proof-evidence--collection)
- [Phase 11 · Reporting & cleanup](#phase-11--reporting--cleanup)
- [12 · OPSEC posture by lane](#12--opsec-posture-by-lane)
- [13 · Time-management templates](#13--time-management-templates)
- [14 · Quick command index → cheat-sheet map](#14--quick-command-index--cheat-sheet-map)
- [Appendix A · GOAD lane reference](#appendix-a--goad-lane-reference)
- [Appendix B · OSCP+ lane reference](#appendix-b--oscp-lane-reference)

---

## 0 · How to use this playbook

Read the spine top to bottom once so the **loop** is muscle memory. On a live target you won't move linearly — you'll bounce between recon, foothold, and privesc as new information arrives — but the phases give you a checklist so nothing is skipped and a vocabulary so your notes stay consistent across every engagement.

> [!tip] The discipline that passes exams and clears engagements
> The operators who clear boxes fastest and pass OSCP+ on the first try almost never fail on a clever exploit. They fail on **incomplete enumeration**, **lost notes**, and **bad time management**. Enumerate fully, take notes as you go (command → result → next step), and time-box every rabbit hole.

---

## 1 · The four lanes at a glance

| Dimension | **HTB** | **GOAD** | **OSCP+** | **General PT** |
|---|---|---|---|---|
| **Authorization** | Platform ToS | Your own lab (you built it) | Exam control panel + Exam Guide | Signed SOW + ROE + authorization letter |
| **Target scope** | One box (or Pro Lab / season network) | Fixed: 5 hosts, 3 domains, 2 forests | 3 standalone + 1 AD set (3 hosts) | Only assets listed in ROE |
| **Start position** | External, unauth | External or assumed (lab choice) | Standalone: external. **AD set: assumed breach** (given a user) | Defined by ROE (external, internal, assumed-breach) |
| **OPSEC posture** | Loud is fine | Loud is fine (lab) | Loud is fine — focus is exploitation, not evasion | **Quiet & log-aware** unless ROE says otherwise |
| **Tooling limits** | None | None | **Metasploit on ONE target only**, no auto-exploit frameworks, no commercial tools, no AI/LLM | Client-permitted tools; avoid destructive/DoS |
| **Win condition** | `user.txt` + `root.txt` | Domain Admin in each domain → Enterprise Admin / cross-forest | 70 / 100 points | All ROE objectives + documented business risk |
| **Proof** | Flag hashes | Notes / writeup | `local.txt` + `proof.txt` in panel **and** screenshot w/ target IP | Evidence chain: screenshots, artifacts, timestamps |
| **Persistence** | No | Optional (practice) | No | Only if in scope, reversible, documented |
| **Cleanup** | Revert box | Revert lab | N/A | **Mandatory** — remove every artifact, document changes |
| **Time-box** | Per box (hours) | Self-paced | 23h 45m + 24h report | Engagement window (days–weeks) |
| **Deliverable** | Optional writeup | Optional writeup | Formal OSCP+ report | Formal client report (exec + technical + remediation) |

> [!note] The mental model
> **HTB** trains breadth of individual techniques. **GOAD** trains the *Active Directory chain* end to end. **OSCP+** is a graded, time-boxed, manual-exploitation gauntlet that now mirrors a real internal AD engagement. **General PT** is the real thing, where scope discipline, stealth, evidence, and remediation advice matter as much as the exploit.

---

## 2 · The operator loop

Every phase feeds the next, and any new credential, host, or piece of access **restarts the loop** from enumeration with your expanded access.

```
            ┌─────────────────────────────────────────────┐
            │                                             │
   ENUMERATE ─▶ IDENTIFY ─▶ EXPLOIT ─▶ STABILIZE ─▶ LOOT ─▶ PIVOT
            │     (attack    (foothold   (solid     (creds,  (new host
            │      surface)   / shell)    shell)     files)   / domain)
            └─────────────────────────────◀──────────────────┘
                     new access ⇒ enumerate again
```

> [!tip] Three rules that hold in every lane
> 1. **Enumerate until it hurts, then enumerate more.** 80% of the work is enumeration; exploitation is the last 20%.
> 2. **Every credential is a key to a new door.** A password, hash, ticket, or key found on host A is the first thing you try against hosts B–Z and the domain.
> 3. **Note as you go.** Command → output → decision. Your notes *are* your report (PT/OSCP+) and your memory when you come back after a break.

---

## Phase 0 · Pre-engagement & scope control

The action that prevents the worst mistakes. Establish the boundary, the objective, and the proof format **before** scanning.

**Standard actions**

- Define the in-scope target list and record it where you'll see it (top of notes, `/etc/hosts`, a scope file).
- Confirm the objective and the proof format for *this* lane.
- Stage your workspace: notes tree, screenshot folder, loot folder, a `scope.txt`, and a logging shell (`tmux` + logging, or `script`).
- Set up name resolution for AD targets (`/etc/hosts`, `krb5.conf` realm) so Kerberos and tooling behave.

**Lane callouts**

- 🟦 **HTB** — Add the box IP to `/etc/hosts` (e.g., `10.10.x.x  target.htb`). Connect via the HTB VPN. No legal paperwork, but the platform ToS is your boundary.
- 🟪 **GOAD** — You own it. Map the hosts (default `192.168.56.0/24`; see Appendix A), set your `/etc/hosts` and `krb5.conf`, and decide which variant/subnet you're attacking (Full / Light / Mini).
- 🟥 **OSCP+** — Read the **Exam Guide** in full *before* the clock starts. Note the per-target point values in the control panel, the proof rules, and the Metasploit-on-one-target restriction. Prepare your report template **now**. Connect with the provided OpenVPN pack.
- 🟧 **General PT** — No packet leaves your host until you have a **signed authorization**, the ROE (windows, allowed techniques, exclusions, emergency contacts, handling of sensitive data), and a confirmed scope. Verify source IPs are whitelisted if required. Know who to call if you trip an incident.

> [!warning] Out-of-scope is out-of-scope
> On a PT, a tempting host that isn't in the ROE is off-limits — full stop. On OSCP+, touching another candidate's range or using a banned tool can void the attempt. Scope discipline is a skill graders and clients both evaluate.

---

## Phase 1 · Recon & enumeration

Map the attack surface completely before touching anything. *(Field manual: **01 Recon & OSINT** — Nmap, recon-ng, Shodan, Censys; **03 Wireless & Network**.)* — ATT&CK **TA0043 / T1595 / T1046 / T1087**.

**Standard actions**

- **Host discovery** (internal/lab): sweep the subnet for live hosts and quick service fingerprints.
  ```bash
  # fast host + SMB sweep across an internal range
  nxc smb 192.0.2.0/24
  fping -aqg 192.0.2.0/24
  ```
- **Port & service enumeration**: all ports, then versioned/scripted scans on what's open. Don't let the default top-1000 hide a service.
  ```bash
  # discover open ports fast, then enumerate them in depth
  nmap -p- --min-rate 2000 -Pn target.example -oA scans/allports
  nmap -p<ports> -sCV -Pn target.example -oA scans/services
  ```
- **Per-service enumeration**: web (vhosts, dirs, tech stack), SMB (shares, null sessions), LDAP/RPC (users, policy), DNS, SNMP, databases, mail. Enumerate *every* open port — the foothold is often on the boring one.
- **Web surface**: directory/vhost brute force, technology fingerprint, parameter discovery, source/JS review. *(Field manual: **02 Web & Access** — Burp, the bug-bounty methodology, Nikto, WordPress; **12 OWASP WSTG** for the structured checklist.)*
- **AD surface** (where present): identify the DC(s), domain/forest names, and whether anonymous/guest enumeration is allowed.

**Lane callouts**

- 🟦 **HTB / 🟪 GOAD / 🟥 OSCP+** — Scan loudly and thoroughly. On OSCP+ standalones, "scan all ports" is non-negotiable; the intended path is frequently on a high/uncommon port.
- 🟧 **General PT** — Throttle to avoid knocking over fragile services, prefer passive/OSINT recon first, and stay inside the ROE windows. Mass vulnerability scanners (Nessus/OpenVAS) may be in scope here — but they're **banned on OSCP+**.

> [!tip] Enumeration completeness check
> Before leaving Phase 1, you should be able to answer: every open port and its version? every web vhost and obvious directory? every SMB share and whether null/guest works? the domain/forest layout and DC list? If any answer is "I'm not sure," you're not done.

---

## Phase 2 · Initial access / foothold

Turn an enumerated weakness into code execution or a valid session. *(Field manual: **02 Web & Access**, **04 Social Engineering**, **07 Credentials**.)* — ATT&CK **T1190 / T1078 / T1110 / T1566**.

**Common foothold vectors (in rough order of "try this first")**

1. **Default / weak / reused credentials** — service logins, admin panels, databases, SNMP community strings. Spray *known* creds before brute-forcing. *(Field manual: Hydra, the password-attack notes.)*
2. **Public-facing web vulns** — file upload → web shell, SQLi, SSTI, LFI/RFI, deserialization, auth bypass, known CVEs in the detected stack. *(Field manual: Burp, SQLMap, the bug-bounty methodology.)*
3. **Known service exploits** — match versions from Phase 1 to a vetted exploit; read the code before you run it.
4. **Exposed secrets** — creds in shares, config files, repos, LDAP descriptions, GPP/SYSVOL.
5. **Phishing / client-side** *(PT lane, when in scope)* — pretext, payload, listener. *(Field manual: SET cheat sheet.)*

**Lane callouts**

- 🟦 **HTB** — Anything goes. Manual or framework, your choice.
- 🟪 **GOAD** — Often an **assumed/low-priv start** (anonymous user listing on `winterfell`, a password in an LDAP `description`, or a provided lab credential). From there it's pure AD chain (Phase 6).
- 🟥 **OSCP+** — **Manual exploitation is the point.** You may use Metasploit/Meterpreter against **one** target of your choice and it locks to that target once used (this includes `check`); `msfvenom`, `multi/handler`, `pattern_create`, and `pattern_offset` are allowed everywhere. Automated exploitation frameworks and mass scanners are banned. The **AD set starts assumed-breach** — you're handed a domain user, so "foothold" there means *using* those creds, not popping a box.
- 🟧 **General PT** — Prefer the quietest reliable path. Avoid anything that risks availability (no untested memory-corruption exploits against production, no DoS). Document the exact request/payload that worked.

> [!warning] Read exploits before you run them
> Public PoCs can be destructive, backdoored, or noisy. Review the code, understand the payload, and adjust the target/listener. On a PT, an unreviewed exploit that crashes a production host is a serious incident.

---

## Phase 3 · Stabilize & local situational awareness

Make the shell reliable, then learn where you are. *(Field manual: Reverse Shell cheat sheet.)* — ATT&CK **T1059 / T1082 / T1083 / T1057**.

**Standard actions**

- **Upgrade the shell**: get a full TTY (Linux `python3 -c 'import pty; pty.spawn("/bin/bash")'` → `stty raw -echo; fg`), or a stable session on Windows. Set up a reliable listener and a fallback.
- **Identify context**: user, privileges/groups, hostname, OS/build, domain membership.
  ```bash
  # linux
  id ; hostname ; uname -a ; sudo -l
  ```
  ```bash
  # windows
  whoami /all ; systeminfo ; ipconfig /all
  ```
- **Map local surroundings**: running processes/services, listening ports, scheduled tasks/cron, installed software, other interfaces (a second NIC = a pivot path), and readable interesting files.

> [!tip] Note your access level immediately
> Record exactly what access you have and how you got it the moment you land. On OSCP+/PT this is the screenshot and the report paragraph; on any lane it's the checkpoint you return to if a later step blows up your shell.

---

## Phase 4 · Privilege escalation

Go from a foothold account to root/SYSTEM (or a more privileged user). *(Field manual: **05 Privilege Escalation** — Windows & Linux methodologies.)* — ATT&CK **TA0004 / T1068 / T1548 / T1574**.

**Standard checklist (both OSes)**

- Run a recognized enumeration script *and* verify findings manually (don't trust one tool blindly).
- **Linux**: `sudo -l`, SUID/SGID binaries, capabilities, cron jobs, writable paths in `$PATH`/services, kernel version vs known exploits, readable secrets, container/escape indicators.
- **Windows**: privilege tokens (`SeImpersonate` → potato family, `SeBackup`/`SeRestore`, etc.), unquoted service paths, weak service/file ACLs, AlwaysInstallElevated, stored creds (Credential Manager, registry, unattended files), scheduled tasks.
- **Always**: re-check for credentials — privesc is frequently "find the password the admin left behind," not a kernel exploit.

**Lane callouts**

- 🟥 **OSCP+** — Privesc is **10 of the 20 points** on each standalone. Enumerate methodically; the path is almost always a misconfiguration or a stored credential, not an exotic 0-day. Capture `proof.txt` from the privileged context.
- 🟧 **General PT** — Prefer low-risk escalation. Note the root cause precisely (it becomes a remediation item) and avoid kernel exploits on production unless explicitly sanctioned.

---

## Phase 5 · Credential access & looting

Harvest everything that opens the next door. *(Field manual: **07 Credentials** — John, Hashcat, Hydra; **06 Active Directory** — Mimikatz.)* — ATT&CK **T1003 / T1555 / T1552 / T1558**.

**Standard actions**

- **Dump local creds**: SAM/LSA/LSASS on Windows; `/etc/shadow`, SSH keys, history, and config secrets on Linux.
- **Hunt stored secrets**: browser/credential stores, DPAPI, config files, scripts, password managers, `.env`/`.git`, KeePass DBs, cloud tokens.
- **Crack offline** what you can't use directly; identify hash type first, then dictionary + rules before brute force. *(Field manual: Hashcat / John.)*
- **Catalogue** every credential with *where it works* — local admin, domain user, service account, DB, app.

> [!tip] Credential reuse is the lateral-movement engine
> The single highest-value habit: every time you obtain a secret, immediately test it (and its hash) against every other host and the domain. This is what turns one foothold into a network — and it's exactly how the OSCP+ AD set and GOAD are designed to fall.

---

## Phase 6 · Active Directory attack path

The chained heart of GOAD and the OSCP+ AD set, and the core of most internal PTs. AD compromise is a **graph problem** — map it first, then walk the shortest path to Domain/Enterprise Admin. *(Field manual: **06 Active Directory** — AD Attack cheat sheet, Mimikatz.)*

**The AD kill chain**

1. **Domain recon** *(T1087, T1069)* — users, groups, computers, GPOs, ACLs, trusts. Run **BloodHound** early and let the graph drive your decisions.
   ```bash
   # collect once you have any domain credential
   nxc ldap dc.target.example -u user -p 'pass' --bloodhound -c all
   ```
2. **First credentials** *(T1558.004 / T1558.003)* — **AS-REP roast** accounts without pre-auth, then **Kerberoast** service accounts; spray/poison where appropriate. Crack offline.
3. **ACL & object abuse** *(T1222 / T1098)* — `GenericAll`, `WriteDACL`, `WriteOwner`, `AddMember`, force-change-password, and similar rights become privilege along BloodHound edges.
4. **Delegation abuse** *(T1558)* — unconstrained, constrained, and **resource-based constrained delegation (RBCD)** to impersonate privileged users.
5. **ADCS abuse** *(T1649)* — vulnerable certificate templates (**ESC1–ESC8**) to mint authentication certs for high-value principals.
6. **Credential dumping & movement** *(T1003.006 DCSync)* — pull hashes via DCSync once you have replication rights; pass-the-hash / pass-the-ticket onward.
7. **Domain dominance** *(T1558.001)* — golden/diamond tickets, `krbtgt` control, then **trusts** for cross-domain and cross-forest reach.

**Lane callouts**

- 🟪 **GOAD** — This *is* the lab. Expect anonymous enumeration → roastable accounts → ACL chains → MSSQL trusted-link hops → NTLM relay (SMB signing off) → ADCS (ESC1-3/ESC8 on `essos`) → cross-forest trust abuse. See Appendix A for the topology and known paths.
- 🟥 **OSCP+** — The AD set is **assumed-breach**: you start with a domain user and pivot to full domain compromise, with **partial credit** for each machine. Expect enumeration, Kerberoasting, ACL abuse, and living-off-the-land — and remember **Metasploit can't be used for pivoting** here.
- 🟧 **General PT** — Same techniques, quietly. Roasting and BloodHound collection are relatively low-noise; relay and coercion attacks can be disruptive — confirm they're in scope and time them to the ROE window.

---

## Phase 7 · Lateral movement & pivoting

Use harvested access to reach new hosts and segments. *(Field manual: **08 Post-Exploitation** — Pivoting (Ligolo-ng / Chisel), Port Forwarding & OPSEC.)* — ATT&CK **T1021 / T1550 / T1090 / T1572**.

**Standard actions**

- **Execute remotely** with creds/hashes/tickets: SMB/WinRM/RDP/SSH and exec tooling (psexec/wmiexec/smbexec patterns, `evil-winrm`, `nxc`). Pass-the-hash and pass-the-ticket where passwords aren't available.
- **Pivot** into unreachable segments: SOCKS proxy + tunnels (Ligolo-ng, Chisel), or local/dynamic port forwards. Map the new subnet, then **restart the loop at Phase 1** from your pivot.

**Lane callouts**

- 🟥 **OSCP+** — Pivoting is in scope and expected for the AD set, but **not via Metasploit** (that would use it against more than one target). Use SSH tunneling, Chisel, Ligolo-ng, or proxychains.
- 🟧 **General PT** — Tunnel deliberately and document each hop. Note segmentation gaps — they're findings in their own right.

---

## Phase 8 · Domain & forest dominance

Convert privileged access into full control and demonstrate blast radius. — ATT&CK **T1003.006 / T1558.001 / T1484 / T1482**.

- Achieve **Domain Admin** (or equivalent high-value access) and demonstrate it: DCSync, access to the DC, control of `krbtgt`.
- Where trusts exist, pursue **Enterprise Admin** and **cross-forest** compromise (GOAD's `sevenkingdoms` ↔ `north` parent/child and the `essos` forest trust are built for exactly this).
- Stop at the objective. On a PT, "I *could* reach X" with proof of the path is often preferable to actually detonating on a sensitive system — document the path and the impact.

---

## Phase 9 · Persistence (lane-gated)

> [!warning] Default: don't. This phase is gated by lane.
> - 🟦 **HTB** / 🟥 **OSCP+** — **Not needed and not wanted.** Boxes revert; the exam scores access and proof, not persistence.
> - 🟪 **GOAD** — Optional, as *practice* (golden tickets, DSRM, ACL backdoors) to learn detection/cleanup — it's your lab.
> - 🟧 **General PT** — **Only if in scope**, explicitly authorized, fully **reversible**, and **documented** (what, where, when, how to remove). Persistence you forget to remove is an incident you caused. Many engagements forbid it outright.

---

## Phase 10 · Proof, evidence & collection

Capture proof in the format the lane requires — incomplete proof can zero out otherwise-perfect work.

| Lane | What counts as proof |
|---|---|
| 🟦 **HTB** | Submit the `user.txt` and `root.txt` flag hashes on the platform. |
| 🟪 **GOAD** | Your writeup/notes — screenshots of each milestone for your own learning record. |
| 🟥 **OSCP+** | `local.txt` (low-priv) **and** `proof.txt` (root/admin) entered in the control panel **and** shown in a screenshot **with the target IP visible** (e.g., `ipconfig` / `ip a` in-frame). Missing either form = **no points** for that target. |
| 🟧 **General PT** | Evidence chain: timestamped screenshots, request/response captures, artifacts, and the exact reproduction steps. Handle any captured sensitive data per the ROE. |

> [!tip] Screenshot the win as you get it
> Don't plan to "go back and grab proof later." On OSCP+ especially, capture the required proof screenshot the moment you have the privileged shell — before something reverts or your session dies.

---

## Phase 11 · Reporting & cleanup

**Reporting**

- 🟦 **HTB** / 🟪 **GOAD** — Optional writeup; valuable for retention and for teaching. Reproducible steps + lessons learned.
- 🟥 **OSCP+** — A formal report is **mandatory** and submitted within **24 hours** of the exam end; an unreported compromise earns no points. Use the OffSec template, document every step reproducibly with the proof screenshots, and submit before the deadline.
- 🟧 **General PT** — The report *is* the product. Executive summary (business risk, in plain language) → methodology/scope → findings (each with severity, evidence, reproduction, and **remediation**) → strategic recommendations → appendices. Tie findings to impact, not just CVSS.

**Cleanup** *(PT lane — mandatory)*

- Remove uploaded tools, web shells, payloads, scheduled tasks/services, and any persistence.
- Restore changed configs; return accounts/files to their prior state.
- Provide an **artifact log**: every file written, account created, and change made, with timestamps, so the client (and their blue team) can verify the environment is clean.

---

## 12 · OPSEC posture by lane

| | Noise tolerance | Evade detection? | Clean up? |
|---|---|---|---|
| 🟦 **HTB** | High | No | Revert |
| 🟪 **GOAD** | High | Optional (great for *practicing* detection with the ELK/SIEM add-ons) | Revert |
| 🟥 **OSCP+** | High — graded on exploitation, not stealth | No | N/A |
| 🟧 **General PT** | **Low by default** | Often yes — assume a SOC is watching; know your footprint (auth logs, EDR, AV, Kerberos anomalies) | **Always** |

> [!note] Use GOAD to bridge offense and defense
> Because GOAD can ship with ELK/Kibana, it's the ideal place to run an attack loudly, then pivot to the blue-team chair and see exactly which artifacts (4624/4768/4769 events, LSASS access, ADCS issuance) your actions generated. That offense→defense feedback loop is what separates an operator from a button-pusher. *(Field manual: **10 Defense** — Blue Team methodology, Wireshark.)*

---

## 13 · Time-management templates

**OSCP+ — 23h 45m (one workable plan)**

| Block | Focus |
|---|---|
| First ~30 min | Kick off all-port scans on every target in parallel; read the control-panel objectives. |
| Next ~6–8 h | Work the **AD set** (40 pts) — assumed-breach → enumerate → roast/ACL → DA. All-or-most of these points is the surest path to 70. |
| Next ~6–8 h | Two **standalones** (40 pts). Rotate when stuck; never sink >90 min into one path without stepping back. |
| Remaining | Third standalone, re-attempts on partial boxes, and **verify every proof screenshot exists**. |
| +24 h | Write and submit the report. Don't start the exam without a finished template. |

> [!tip] The rabbit-hole timer
> Set a hard timer (e.g., 60–90 min) per attack path. When it rings, write down where you are and rotate to another target. Returning with fresh eyes beats grinding a dead end — the most common OSCP+ failure mode is tunnel vision plus poor time management.

**HTB box** — enumerate fully → one solid foothold → stabilize → privesc → grab both flags → (optional) writeup.

**General PT** — front-load OSINT/passive recon and scope confirmation, schedule loud/disruptive tests inside agreed windows, reserve the back third of the engagement for analysis, **report writing**, and cleanup verification.

---

## 14 · Quick command index → cheat-sheet map

The playbook tells you *what* to do and *when*; these chapters give you the *exact commands*.

| Phase | Field-manual chapters |
|---|---|
| 0 · Pre-engagement | **00 Start Here** (Methodology, Learning Path) · **11 OSCP / PTES Notes** |
| 1 · Recon & enum | **01 Recon & OSINT** (Nmap, recon-ng, Shodan, Censys, Sherlock, VPN Hunting) · **03 Wireless & Network** |
| 2 · Foothold (web) | **02 Web & Access** (Burp, Bug-Bounty, SQLMap, Nikto, WordPress) · **12 OWASP WSTG** |
| 2 · Foothold (human) | **04 Social Engineering** (SET) |
| 3 · Shells | **08 Post-Exploitation** (Reverse Shells) |
| 4 · Privesc | **05 Privilege Escalation** (Windows, Linux) |
| 5 · Creds & cracking | **07 Credentials** (John, Hashcat, Hydra) |
| 6 · Active Directory | **06 Active Directory** (AD Attack, Mimikatz) |
| 7 · Lateral / pivot | **08 Post-Exploitation** (Pivoting — Ligolo-ng/Chisel, Port Forwarding & OPSEC, Metasploit) |
| 9 · Containers (as needed) | **09 Containers** (Docker) |
| 10–12 · Detection view | **10 Defense** (Blue Team, Wireshark) |

---

## Appendix A · GOAD lane reference

> [!brief] GOAD (full) topology
> 5 Windows VMs · 3 domains · 2 forests · bidirectional trusts. Default subnet `192.168.56.0/24`. Built by Orange Cyberdefense / @M4yFly; walkthroughs at mayfly277.github.io.

| Host | FQDN | Role | OS | Notes |
|---|---|---|---|---|
| kingslanding | `kingslanding.sevenkingdoms.local` | DC — `sevenkingdoms.local` | WS 2019 | Defender on |
| winterfell | `winterfell.north.sevenkingdoms.local` | DC — `north.sevenkingdoms.local` (child) | WS 2019 | Anonymous user listing |
| castelblack | `castelblack.north.sevenkingdoms.local` | Member server | WS 2019 | IIS (ASP upload), MSSQL; Defender **off** by default |
| meereen | `meereen.essos.local` | DC — `essos.local` (separate forest) | WS 2016 | **ADCS** (ESC1-3 / ESC8) |
| braavos | `braavos.essos.local` | Member server | WS 2016 | MSSQL (trusted link to castelblack) |

**Forest/trust shape:** `sevenkingdoms.local` ⇄ child `north.sevenkingdoms.local` (intra-forest), with a separate forest `essos.local` joined by a trust — purpose-built for cross-domain and cross-forest practice.

**Representative attack paths to chain (Phase 6):**

- Anonymous/low-priv enumeration → user with password in LDAP `description` (e.g., `samwell.tarly`) or a provided lab credential (e.g., `tywin.lannister`).
- **AS-REP roast** + **Kerberoast** → crack offline.
- **ACL abuse** edges (`GenericAll`, `WriteProperty`/self-membership, add-member to privileged groups) — let BloodHound draw the route.
- **MSSQL trusted links** (castelblack ⇄ braavos) + `EXECUTE AS` impersonation.
- **NTLM relay** (SMB signing not enforced) → admin on a relayed host.
- **ADCS ESC1-3 / ESC8** on `essos.local` → certificate-based auth as a privileged principal.
- **DCSync** → **golden ticket** → **cross-forest** trust abuse to finish all three domains.

> [!note] Lighter variants
> **GOAD-Light** (3 VMs, `sevenkingdoms` + `north`, ~20 GB — no `essos`, so no cross-forest or MSSQL-link/ADCS-ESC4 scenarios) and **GOAD-Mini** (2 VMs) exist for smaller hardware. The chain is the same; some endgame paths are absent.

---

## Appendix B · OSCP+ lane reference

> [!brief] OSCP+ exam shape (current format)
> 23h 45m hands-on + 24h reporting · 100 points · **70 to pass** · proctored, open-book · **no AI/LLM chatbots** during the exam or reporting.

**Point map**

| Component | Machines | Points |
|---|---|---|
| Standalone targets | 3 | 20 each = **60** (10 foothold + 10 privesc per box) |
| Active Directory set | 3 (assumed breach) | **40**, with partial credit per machine |

- **AD set = assumed breach:** you're given a domain user and pivot to full domain compromise. Partial points per AD machine (the old all-or-nothing gate and the bonus points are gone as of the Nov 2024 update).
- **Metasploit:** usable against **one** target of your choice and **locks** to it the moment you use it (including `check`); never for pivoting. `msfvenom`, `exploit/multi/handler`, `pattern_create`, `pattern_offset` are allowed everywhere.
- **Banned:** AI/LLM chatbots, commercial tools (Burp Pro, Metasploit Pro), mass vulnerability scanners (Nessus/OpenVAS), spoofing, and automated-exploitation frameworks.
- **Proof per target:** `local.txt` and `proof.txt` submitted in the control panel **and** captured in a screenshot **with the target IP visible**. Miss either and that target scores zero.
- **Connectivity:** Kali + the provided OpenVPN pack (emailed at the exact start time).

> [!tip] Strategy
> Bank the **AD set (40)** plus **two standalones (40)** and you're at 80 — clear of the 70 line. Enumerate everything fully, keep Metasploit in reserve for the one box that truly needs it, screenshot proof immediately, and never skip the report.

> [!warning] Verify the current guide before exam day
> Exam formats change. Treat this appendix as a working summary and confirm the specifics against the **official OffSec OSCP+ Exam Guide** before you schedule — the Guide is authoritative if anything here has shifted.

---

*Living document — pair it with the field manual chapters referenced throughout. Every example uses the RFC-reserved `target.example`; swap in your authorized scope.*
