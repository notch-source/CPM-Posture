# Compliance Posture Mapper (CPM)

> Turn your CMDB services into a living compliance map. Service owners upload evidence, the compliance team approves it, and everyone sees what is covered, in one portal.


## Why this exists

Compliance evidence usually lives in mailboxes, shared drives and spreadsheets. Nobody can answer simply: **which of our services are covered today, who owns the gap, and what expires next month?**

CPM answers that question from the CMDB itself:

- Controls are attached to **services and configuration items (CSDM)**, not to a spreadsheet row.
- The **owner in the CMDB** is the person asked for evidence, automatically.
- A service is **covered** when it has at least one valid approved evidence. No manual status updates.

## Features

| Area | What you get |
|---|---|
| Dashboard | One screen: banner, stats guide on the left, work area in the middle, all the numbers on the right. Coverage ring, controls by status, frameworks, approvals of the last 30 days |
| Evidence workflow | Owners drop or paste screenshots and PDFs, the compliance team approves or rejects with a reason, renewals are accepted before expiry |
| CMDB driven | Targets can be services or CIs, optionally with their components (CMDB relationships). The list of components is frozen into every approval |
| Automation | Nightly job syncs owners and states and sends reminders (30, 7 and 1 days before expiry, expired, new and open items) |
| Exceptions handled | A CI without owner, an inactive owner or an owner without email is reported to the compliance team instead of failing silently |
| Audit | Integrity seal (SHA-256) on every approval, "changed after approval" detection, audit CSV export |
| Security | Compliance tables hidden from everyone but the compliance team, files served only by the app, server-side file validation, evidence is final once decided |
| Experience | Fast paged cards, searchable stats guide, keyboard friendly, remembers your tab |


## Requirements

- A ServiceNow instance 
- Service Portal active and a CMDB with `cmdb_ci_service` records.
- A modern browser: Chrome or Edge 105+, Safari 16+, Firefox 110+.

## Install

1. In **Studio**, choose **Import From Source Control**, paste this repository URL and the branch. A public repository needs no credential, a private one needs a credential holding an access token.
2. Create a group, **Compliance Posture Admins**, and give the role `x_1929776_compli_0.admin` **to the group only**.
3. Set the [properties](#configuration) (at least `admin_group` and `service_scope`).
4. Create the unique index on the mapping table: fields **Control** and **Target**.
5. Check that the six events and their notifications exist and that the nightly scheduled job is active.
6. Open `/sp?id=cpm`.


## Configuration

All properties are prefixed with `x_1929776_compli_0.`

| Property | Default | Meaning |
|---|---|---|
| `admin_group` | empty | Exact name of the compliance group. When set, the portal and the Studio guard require membership. Empty means role only |
| `service_scope` | empty | Encoded query on `cmdb_ci_service` that defines the services in scope (for example operational status, environment) |
| `fallback_owner` | empty | sys_id of the user who covers a CI that has no owner in the CMDB. Leave empty to have such CIs reported to the team |
| `warning_days` | 30 | Window during which an approval counts as "expiring soon" and a renewal can be sent |
| `green_min` / `yellow_min` | 80 / 50 | Thresholds of the traffic lights, and the coverage goal drawn on the ring |
| `max_file_mb` | 3 | Maximum size of one uploaded file |
| `allow_self_review` | false | Allow a reviewer to decide on evidence they submitted themselves |

## Roles and who sees what

| Person | Sees | Can do |
|---|---|---|
| Service owner (any user) | Only the controls they own, in the portal | Upload evidence for the controls they own |
| Compliance team (role and group) | Everything, plus the three tables | Review, approve, reject, export the audit, refresh owners |

## Daily operation

The nightly job runs five isolated steps: sync mappings with their controls, refresh owners from the CMDB, bring states up to date (expiry), remove abandoned drafts, send reminders and the team alert. The portal shows a **Setup check** to the compliance team when a field is missing, a Script Include is out of date, or the nightly job did not run.

## Project layout

| Part | Files |
|---|---|
| Tables | Control, Mapping, Evidence |
| Script Includes | `CPMConst`, `CPMUtil`, `CPMAccess`, `CPMLookup`, `CPMScope`, `CPMMapping`, `CPMEvidence`, `CPMReview`, `CPMStats`, `CPMNotify`, `CPMMsg` |
| Portal | One widget (HTML, SCSS, client script, server script) and the page `cpm` |
| Automation | 4 business rules, 1 scheduled job, 6 events with their notifications |


## Known limitations

- The integrity seal detects ordinary or accidental changes. It is not an HMAC, so someone able to recompute SHA-256 could forge it.
- Files are validated by type, size and signature. There is no antivirus scan unless your instance provides one for attachments.
- Users in different time zones can see an expiry one day apart around midnight.
- The interface is in English.


## Disclaimer

This is an independent open source project. It is not affiliated with, endorsed by or supported by ServiceNow, Inc. ServiceNow is a trademark of ServiceNow, Inc.
