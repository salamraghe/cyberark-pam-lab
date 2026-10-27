# CyberArk PAM Self-Hosted 12.6 Home Lab

Hands-on lab, Oct 2026.

A ticket-style privileged access loop in CyberArk PAM Self-Hosted 12.6. I put a Domain Admin service account in a safe. A requester has to ask for it and an approver has to sign off. CPM rotates the password, the admin session runs through PSM and gets recorded, and when the work is done the requester's access is pulled. That's the whole point of PAM: nobody holds a Domain Admin password long-term, and every use leaves a trail.

## What I built

- Five Windows Server VMs in VMware Workstation, domain `pitythefool.com`: WIN-ADDC (DC), WIN-VAULT (workgroup), WIN-CPM, WIN-PVWA, WIN-PSM.
- Installed Vault 12.6, CPM (finished with a CPMEM038E platform import warning; platforms still loaded in PVWA), PVWA and PSM.
- Created `svc-test` in Domain Admins, created safe **Lab-Admins**, onboarded `svc-test` on the WinDomain platform.
- Created requester `jdoe` through the PVWA REST API and added `mgr1` to Lab-Admins as approver, so the person asking isn't the person approving.
- Turned on dual control in the Master Policy and ran Ticket 1001: jdoe asks, mgr1 confirms, the audit trail shows both sides.
- Had CPM change the password. It failed once (winRc 2245), I fixed the domain policy, and it rotated to Compliant.
- Turned on session monitoring and connected through PSM to WIN-ADDC (Tickets 1002/1003). The session showed "You are being recorded" and Activities logged PSM Connect, window titles and PSM Disconnect.
- Removed jdoe from Lab-Admins and snapshotted all five VMs (`PAM-loop-complete-2026-10-12`).

## What I learned

In ticket language: requester jdoe asked for temporary access to the Domain Admin account svc-test with a ticket number. Approver mgr1 confirmed it, so no one approved their own access. The password was never handed out long-term: CPM rotates it, and the admin session ran through PSM where it was recorded and every window was logged. When the work was done, jdoe was removed from the safe and lost access.

Interview line: "In CyberArk I locked a Domain Admin account in a safe with dual control, so a user had to ask with a ticket and a manager had to approve. CPM rotated the password, the admin session was recorded through PSM, and I removed the user's access at the end."

## Key screenshots

![Built five lab VMs in VMware: domain controller, Vault, PVWA, CPM, PSM.](key/01-five-vms-built.png)

Built five lab VMs in VMware: domain controller, Vault, PVWA, CPM, PSM.

![Master Policy turned on dual control so access needs manager approval.](key/02-dual-control-policy-on.png)

Master Policy turned on dual control so access needs manager approval.

![Manager approved jdoe's request; CyberArk confirmed the request successfully.](key/03-access-request-approved.png)

Manager approved jdoe's request; CyberArk confirmed the request successfully.

![Error I hit: CPM password change failed until I fixed the password policy.](key/04-error-cpm-change-failed.png)

Error I hit: CPM password change failed until I fixed the password policy.

![After the fix, CPM rotated the password and the account shows Compliant.](key/05-password-rotated-compliant.png)

After the fix, CPM rotated the password and the account shows Compliant.

![PSM session to the domain controller is being recorded, as shown.](key/06-psm-session-recorded.png)

PSM session to the domain controller is being recorded, as shown.

![Account activity log shows the PSM connect, window titles and disconnect.](key/07-session-activity-audit.png)

Account activity log shows the PSM connect, window titles and disconnect.

![Removed jdoe from the Lab-Admins safe to end their privileged access.](key/08-jdoe-access-removed.png)

Removed jdoe from the Lab-Admins safe to end their privileged access.

## Environment

| Object | Value |
| --- | --- |
| Hypervisor | VMware Workstation Pro |
| Domain | `pitythefool.com` |
| WIN-ADDC | 192.168.100.10, domain controller (AD DS, DNS, DHCP) |
| WIN-VAULT | 192.168.100.11, Digital Vault 12.6, workgroup |
| WIN-CPM | 192.168.100.12, Central Policy Manager |
| WIN-PVWA | 192.168.100.13, Password Vault Web Access |
| WIN-PSM | 192.168.100.14, Privileged Session Manager |
| Target account | `svc-test` (Domain Admins), WinDomain platform, safe Lab-Admins |
| Requester / approver | `jdoe` (EPVUser, via REST API) / `mgr1` (Lab-Admins, approve rights) |

## What broke / issues I fixed

- **CPM change failed, winRc 2245.** The error said the new password didn't meet policy. Likely cause: the domain minimum password age blocked changing a freshly set password again. Fix: `Set-ADDefaultDomainPasswordPolicy -MinPasswordAge 0`, `gpupdate /force`, change again → Compliant.
- **Connect button stayed gray after approval.** Session monitoring wasn't on in the Master Policy. Turned on "Require privileged session monitoring and isolation" → Connect lit up.
- **PSM connection went nowhere.** The PSM server object pointed at the NAT IP. Changed the Address to `192.168.100.14`.
- **"Privileged session could not be established securely."** RDP security on PSM (NLA / SecurityLayer) didn't line up; adjusted it on WIN-PSM.
- **Session couldn't reach the DC.** Remote Desktop was off on WIN-ADDC; enabled it.

## Gaps

- No before-shots for the NAT IP address, the NLA/SecurityLayer change, or enabling RDP on WIN-ADDC. Those fixes are from my notes; the screenshots show the after state (48 address = .14, 49 to 51 working session).
- The 2245 root cause is my best diagnosis, not something a screenshot proves directly.
- Two proof shots are left out on purpose because they show plaintext passwords (27 REST call creating jdoe, 35 revealed svc-test password).

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
11. Safes → Lab-Admins → Members → jdoe → Remove. Snapshot all 5 VMs.

## Demo walkthrough

1. Show the five VMs and the domain (06–16).
2. Show svc-test in Domain Admins and onboarded in Lab-Admins (24–26).
3. Show mgr1 as an approver and the dual control policy (28–29).
4. Walk Ticket 1001: jdoe asks (30–31), mgr1 approves (32–33), jdoe is confirmed (34).
5. Show the 2245 failure, the policy fix, and Compliant (36–39).
6. Walk Ticket 1002/1003: approval, Connect enabled, "You are being recorded", session on ADDC, Activities with window titles and PSM Disconnect (41–56).
7. Remove jdoe (57–59), then show the snapshots on all 5 VMs (60–64).

<details><summary>Full proof (all screenshots)</summary>

![01-installers-downloaded](full-proof/01-installers-downloaded.png)

![02-vmware-official-download](full-proof/02-vmware-official-download.png)

![03-vmware-installed](full-proof/03-vmware-installed.png)

![04-base-vm-loaded](full-proof/04-base-vm-loaded.png)

![05-base-vm-logged-in](full-proof/05-base-vm-logged-in.png)

![07-win-addc-configured](full-proof/07-win-addc-configured.png)

![08-win-cpm-demoted-renamed](full-proof/08-win-cpm-demoted-renamed.png)

![09-win-pvwa-demoted-renamed](full-proof/09-win-pvwa-demoted-renamed.png)

![10-win-psm-demoted-renamed](full-proof/10-win-psm-demoted-renamed.png)

![11-win-vault-demoted-renamed](full-proof/11-win-vault-demoted-renamed.png)

![12-win-cpm-domain-join](full-proof/12-win-cpm-domain-join.png)

![13-win-pvwa-domain](full-proof/13-win-pvwa-domain.png)

![14-win-psm-after-join](full-proof/14-win-psm-after-join.png)

![15-win-addc-running](full-proof/15-win-addc-running.png)

![16-ping-tests](full-proof/16-ping-tests.png)

![17-vault-setup-complete](full-proof/17-vault-setup-complete.png)

![18-cpm-cpmem038e-platform-import-warning](full-proof/18-cpm-cpmem038e-platform-import-warning.png)

![19-cpm-install-complete](full-proof/19-cpm-install-complete.png)

![20-pvwa-login-page](full-proof/20-pvwa-login-page.png)

![21-pvwa-signed-in-account-view](full-proof/21-pvwa-signed-in-account-view.png)

![22-platforms-loaded](full-proof/22-platforms-loaded.png)

![23-psm-install-complete](full-proof/23-psm-install-complete.png)

![24-svc-test-created](full-proof/24-svc-test-created.png)

![25-safe-lab-admins-created](full-proof/25-safe-lab-admins-created.png)

![26-svc-test-onboarded](full-proof/26-svc-test-onboarded.png)

![28-mgr1-approver-added](full-proof/28-mgr1-approver-added.png)

![30-jdoe-request-form](full-proof/30-jdoe-request-form.png)

![31-request-sent-pending](full-proof/31-request-sent-pending.png)

![32-mgr1-incoming-request](full-proof/32-mgr1-incoming-request.png)

![34-jdoe-confirmed-audit-trail](full-proof/34-jdoe-confirmed-audit-trail.png)

![37-min-password-age-0-gpupdate](full-proof/37-min-password-age-0-gpupdate.png)

![38-cpm-change-dialog](full-proof/38-cpm-change-dialog.png)

![40-psm-service-running](full-proof/40-psm-service-running.png)

![41-ticket1002-request-sent](full-proof/41-ticket1002-request-sent.png)

![42-mgr1-incoming-ticket1002](full-proof/42-mgr1-incoming-ticket1002.png)

![43-mgr1-approve-reason](full-proof/43-mgr1-approve-reason.png)

![44-ticket1002-confirmed](full-proof/44-ticket1002-confirmed.png)

![45-jdoe-confirmed-connect-gray](full-proof/45-jdoe-confirmed-connect-gray.png)

![46-master-policy-psm-monitoring-active](full-proof/46-master-policy-psm-monitoring-active.png)

![47-jdoe-connect-enabled](full-proof/47-jdoe-connect-enabled.png)

![48-psm-server-address-fixed](full-proof/48-psm-server-address-fixed.png)

![50-psm-reached-addc-secure-error](full-proof/50-psm-reached-addc-secure-error.png)

![51-psm-session-on-addc](full-proof/51-psm-session-on-addc.png)

![52-psm-session-activity-notepad](full-proof/52-psm-session-activity-notepad.png)

![53-psm-activity-window-titles-disconnect](full-proof/53-psm-activity-window-titles-disconnect.png)

![54-activity-ticket1003-connect-approval](full-proof/54-activity-ticket1003-connect-approval.png)

![55-activity-psm-connect-window-titles](full-proof/55-activity-psm-connect-window-titles.png)

![57-remove-jdoe-menu](full-proof/57-remove-jdoe-menu.png)

![58-remove-jdoe-confirm](full-proof/58-remove-jdoe-confirm.png)

![60-addc-snapshot-taken](full-proof/60-addc-snapshot-taken.png)

![61-vault-snapshot-taken](full-proof/61-vault-snapshot-taken.png)

![62-pvwa-snapshot-taken](full-proof/62-pvwa-snapshot-taken.png)

![63-cpm-snapshot-taken](full-proof/63-cpm-snapshot-taken.png)

![64-psm-snapshot-taken](full-proof/64-psm-snapshot-taken.png)

</details>
