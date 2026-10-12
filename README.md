# CyberArk PAM Self-Hosted 12.6 Home Lab

Hands-on lab, Oct 2026.

Ticket-style privileged access loop in **CyberArk PAM Self-Hosted 12.6**: onboard a Domain Admin service account into a safe, make a requester ask for it, make an approver sign off, let CPM rotate the password, open a recorded PSM session, then pull the requester's access.

This is **not** Entra ID, Okta, Windows AD basics, or CyberKy/LLDAP. Those live in other repos.

## Scope

| # | Item | Dump |
| --- | --- | --- |
| 1 | 5 VMs on VMware Workstation, domain **pitythefool.com**, Vault / CPM / PVWA / PSM installed | Done (Oct 2026) |
| 2 | Onboard AD account **svc-test** (Domain Admins) into safe **Lab-Admins**, WinDomain platform | Done (Oct 2026) |
| 3 | Users via PVWA REST API: **jdoe** (requester), **mgr1** (approver) — separation of duties | Done (Oct 2026) |
| 4 | Master Policy: dual control password access approval | Done (Oct 2026) |
| 5 | Request → approve → grant (Ticket 1001) | Done (Oct 2026) |
| 6 | CPM rotation: failed winRc 2245, fixed MinPasswordAge 0, rotated → Compliant | Done (Oct 2026) |
| 7 | PSM monitored session to WIN-ADDC (Tickets 1002/1003), recorded, activities logged | Done (Oct 2026) |
| 8 | Remove jdoe from Lab-Admins | Done (Oct 2026) |
| 9 | VM snapshot **PAM-loop-complete-2026-10-12** | Done (Oct 2026) |

## Environment

| Object | Value |
| --- | --- |
| Hypervisor | VMware Workstation Pro |
| Domain | `pitythefool.com` |
| WIN-ADDC | 192.168.100.10 — domain controller (AD DS, DNS, DHCP) |
| WIN-VAULT | 192.168.100.11 — Digital Vault 12.6, workgroup (not domain-joined) |
| WIN-CPM | 192.168.100.12 — Central Policy Manager |
| WIN-PVWA | 192.168.100.13 — Password Vault Web Access (`https://win-pvwa.pitythefool.com/PasswordVault`) |
| WIN-PSM | 192.168.100.14 — Privileged Session Manager |
| Target account | `svc-test` (Domain Admins), platform WinDomain, safe **Lab-Admins** |
| Requester | `jdoe` (EPVUser, created via REST API) |
| Approver | `mgr1` (Lab-Admins member with approve rights) |

## Screenshots

Each shot has a hand-drawn oval on the thing I clicked or the result that matters. Links go to the annotated copies; the untouched originals are in `screenshots/` with the same filenames.

Two shots are left out on purpose because they show plaintext passwords: `27-jdoe-created-via-rest-api` (REST call with the admin password and jdoe's initial password on screen) and `35-grant-password-shown` (the revealed svc-test password). The REST output for jdoe and the grant itself are still covered by 28 and 34.

### Build: VMware, 5 VMs, domain, installs

- [installers downloaded](screenshots/annotated/01-installers-downloaded.png)
- [vmware official download](screenshots/annotated/02-vmware-official-download.png)
- [vmware installed](screenshots/annotated/03-vmware-installed.png)
- [base vm loaded](screenshots/annotated/04-base-vm-loaded.png)
- [base vm logged in](screenshots/annotated/05-base-vm-logged-in.png)
- [five vms cloned](screenshots/annotated/06-five-vms-cloned.png)
- [win addc configured](screenshots/annotated/07-win-addc-configured.png)
- [win cpm demoted renamed](screenshots/annotated/08-win-cpm-demoted-renamed.png)
- [win pvwa demoted renamed](screenshots/annotated/09-win-pvwa-demoted-renamed.png)
- [win psm demoted renamed](screenshots/annotated/10-win-psm-demoted-renamed.png)
- [win vault demoted renamed](screenshots/annotated/11-win-vault-demoted-renamed.png)
- [win cpm domain join](screenshots/annotated/12-win-cpm-domain-join.png)
- [win pvwa domain](screenshots/annotated/13-win-pvwa-domain.png)
- [win psm after join](screenshots/annotated/14-win-psm-after-join.png)
- [win addc running](screenshots/annotated/15-win-addc-running.png)
- [ping tests](screenshots/annotated/16-ping-tests.png)
- [vault setup complete](screenshots/annotated/17-vault-setup-complete.png)
- [cpm cpmem038e platform import warning](screenshots/annotated/18-cpm-cpmem038e-platform-import-warning.png)
- [cpm install complete](screenshots/annotated/19-cpm-install-complete.png)
- [pvwa login page](screenshots/annotated/20-pvwa-login-page.png)
- [pvwa signed in account view](screenshots/annotated/21-pvwa-signed-in-account-view.png)
- [platforms loaded](screenshots/annotated/22-platforms-loaded.png)
- [psm install complete](screenshots/annotated/23-psm-install-complete.png)

### 1. Onboard svc-test into Lab-Admins (WinDomain)

- [svc test created](screenshots/annotated/24-svc-test-created.png)
- [safe lab admins created](screenshots/annotated/25-safe-lab-admins-created.png)
- [svc test onboarded](screenshots/annotated/26-svc-test-onboarded.png)

### 2. Users via REST API: jdoe requester, mgr1 approver

- [mgr1 approver added](screenshots/annotated/28-mgr1-approver-added.png)

### 3. Master Policy: dual control

- [master policy dual control active](screenshots/annotated/29-master-policy-dual-control-active.png)

### 4. Request → approve → grant (Ticket 1001)

- [jdoe request form](screenshots/annotated/30-jdoe-request-form.png)
- [request sent pending](screenshots/annotated/31-request-sent-pending.png)
- [mgr1 incoming request](screenshots/annotated/32-mgr1-incoming-request.png)
- [request approved](screenshots/annotated/33-request-approved.png)
- [jdoe confirmed audit trail](screenshots/annotated/34-jdoe-confirmed-audit-trail.png)

### 5. CPM rotation: fail 2245 → fix → Compliant

- [cpm change failed 2245](screenshots/annotated/36-cpm-change-failed-2245.png)
- [min password age 0 gpupdate](screenshots/annotated/37-min-password-age-0-gpupdate.png)
- [cpm change dialog](screenshots/annotated/38-cpm-change-dialog.png)
- [cpm rotation compliant](screenshots/annotated/39-cpm-rotation-compliant.png)

### 6. PSM monitored session (Tickets 1002/1003)

- [psm service running](screenshots/annotated/40-psm-service-running.png)
- [ticket1002 request sent](screenshots/annotated/41-ticket1002-request-sent.png)
- [mgr1 incoming ticket1002](screenshots/annotated/42-mgr1-incoming-ticket1002.png)
- [mgr1 approve reason](screenshots/annotated/43-mgr1-approve-reason.png)
- [ticket1002 confirmed](screenshots/annotated/44-ticket1002-confirmed.png)
- [jdoe confirmed connect gray](screenshots/annotated/45-jdoe-confirmed-connect-gray.png)
- [master policy psm monitoring active](screenshots/annotated/46-master-policy-psm-monitoring-active.png)
- [jdoe connect enabled](screenshots/annotated/47-jdoe-connect-enabled.png)
- [psm server address fixed](screenshots/annotated/48-psm-server-address-fixed.png)
- [psm session being recorded](screenshots/annotated/49-psm-session-being-recorded.png)
- [psm reached addc secure error](screenshots/annotated/50-psm-reached-addc-secure-error.png)
- [psm session on addc](screenshots/annotated/51-psm-session-on-addc.png)
- [psm session activity notepad](screenshots/annotated/52-psm-session-activity-notepad.png)
- [psm activity window titles disconnect](screenshots/annotated/53-psm-activity-window-titles-disconnect.png)
- [activity ticket1003 connect approval](screenshots/annotated/54-activity-ticket1003-connect-approval.png)
- [activity psm connect window titles](screenshots/annotated/55-activity-psm-connect-window-titles.png)
- [activity notepad psm disconnect](screenshots/annotated/56-activity-notepad-psm-disconnect.png)

### 7. Remove jdoe from Lab-Admins

- [remove jdoe menu](screenshots/annotated/57-remove-jdoe-menu.png)
- [remove jdoe confirm](screenshots/annotated/58-remove-jdoe-confirm.png)
- [jdoe removed from lab admins](screenshots/annotated/59-jdoe-removed-from-lab-admins.png)

### 8. Snapshot

- [addc snapshot taken](screenshots/annotated/60-addc-snapshot-taken.png)

## What was built

- Downloaded the CyberArk PAM Self-Hosted and VMware installers, built one Windows Server 2016 base VM and cloned it into five.
- Renamed each clone, made WIN-ADDC the DC for **pitythefool.com**, joined CPM / PVWA / PSM to the domain, kept the Vault in a workgroup.
- Installed Vault 12.6, CPM (got the CPMEM038E platform import warning, finished anyway and the platforms showed up in PVWA), PVWA, and PSM.
- Created **svc-test** in AD and put it in Domain Admins, created safe **Lab-Admins**, onboarded svc-test on the WinDomain platform.
- Created **jdoe** through the PVWA REST API and added **mgr1** to Lab-Admins as the approver, so the person asking is not the person approving.
- Turned on dual control in the Master Policy.
- Ran the full request loop: jdoe asks with a reason, mgr1 confirms, jdoe gets access, the audit trail shows both sides.
- Had CPM change the password. It failed once, I fixed the domain policy, and it rotated to Compliant.
- Turned on session monitoring, fixed the PSM server address, and connected through PSM to WIN-ADDC. The session showed "You are being recorded", and the account's Activities logged PSM Connect, window titles (notepad.exe, explorer.exe), and PSM Disconnect.
- Removed jdoe from Lab-Admins and took a snapshot.

## How to redo it

1. Build 5 Windows Server VMs on 192.168.100.0/24: ADDC .10, VAULT .11, CPM .12, PVWA .13, PSM .14. Promote ADDC, join CPM/PVWA/PSM, leave VAULT in a workgroup.
2. Install Vault → CPM → PVWA → PSM. Sign in to PVWA as Administrator and check Platforms loaded.
3. In AD, create `svc-test` and add it to Domain Admins.
4. PVWA → Policies → Safes → create **Lab-Admins**. Add account `svc-test` on the WinDomain platform.
5. Create `jdoe` with `POST /PasswordVault/API/Users` (EPVUser). Add `mgr1` to Lab-Admins via `POST /API/Safes/Lab-Admins/Members` with request-authorization rights.
6. Master Policy → Privileged Access Workflows → **Require dual control password access approval** = Active.
7. As jdoe, request the account with a ticket reason. As mgr1, Incoming Requests → Confirm. As jdoe, see "Confirmed request".
8. On the DC, set `MinPasswordAge 0` and `gpupdate /force`. In PVWA, Change → let CPM rotate; confirm Compliant.
9. Master Policy → Session Management → **Require privileged session monitoring and isolation** = Active. Set the PSM server Address to `192.168.100.14`. Make sure RDP is on at the target and PSM's RDP security settings match.
10. As jdoe (after approval), Connect. Do something in the session (notepad), disconnect, check Activities.
11. Safes → Lab-Admins → Members → jdoe → Remove. Snapshot.

## Issues I fixed

- **CPM change failed, winRc 2245.** Cause: the domain password policy blocked the change; the error said the password didn't meet policy, and the minimum password age meant a freshly set password couldn't be changed again yet. Fix: `Set-ADDefaultDomainPasswordPolicy -MinPasswordAge 0`, `gpupdate /force`, change again → Compliant.
- **Connect button stayed gray after approval.** Cause: session monitoring wasn't on in the Master Policy, so there was no PSM connection to offer. Fix: set "Require privileged session monitoring and isolation" to Active → Connect lit up.
- **PSM connection went nowhere.** Cause: the PSM server object was pointing at the NAT IP, not the lab network. Fix: changed the Address to `192.168.100.14`.
- **"Privileged session could not be established securely."** Cause: RDP security on PSM (NLA / SecurityLayer) didn't line up with the connection. Fix: adjusted NLA / SecurityLayer on WIN-PSM.
- **Session couldn't reach the DC.** Cause: Remote Desktop was off on WIN-ADDC. Fix: enabled RDP on WIN-ADDC.

Gaps in the screenshots: there's no shot of the PSM address *before* (NAT IP), of the NLA/SecurityLayer change, or of RDP being enabled on ADDC. Those fixes are written from my notes; the screenshots show the after state (48 address = .14, 49–51 working session).

## Demo walkthrough

1. Show the five VMs and the domain (06–16).
2. Show svc-test in Domain Admins and onboarded in Lab-Admins (24–26).
3. Show mgr1 as an approver and the dual control policy (28–29).
4. Walk Ticket 1001: jdoe asks (30–31), mgr1 approves (32–33), jdoe is confirmed (34).
5. Show the 2245 failure, the policy fix, and Compliant (36–39).
6. Walk Ticket 1002/1003: approval, Connect enabled, "You are being recorded", session on ADDC, Activities with window titles and PSM Disconnect (41–56).
7. Remove jdoe (57–59).

In ticket language: *Requester jdoe asked for temporary access to the Domain Admin account svc-test with a ticket number. Approver mgr1 confirmed it, so no one approved their own access. The password was never handed out long-term: CPM rotates it, and the admin session ran through PSM where it was recorded and every window was logged. When the work was done, jdoe was removed from the safe and lost access.*
