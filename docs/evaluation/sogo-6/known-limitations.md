# Known Limitations & Risks — SOGo 6 and the 5→6 Transition

**Last updated:** 2026-06-26  
**Scope:** SOGo 6 public demo (pre-beta) and SOGo 5.x → 6.x migration window  
**Status:** Living document — entries will be added/closed as upstream issues resolve.

---

## SOGo 6 Demo — Console & Network Errors (2026-06-23 baseline)

The following errors were observed in the browser console while using the public SOGo 6 demo at `demov6.sogo.nu`. They indicate missing endpoints, misconfigured static asset serving, or incomplete features in the pre-beta build.

| Severity | Issue | Reproduction | Source |
|---|---|---|---|
| **High** | Strict MIME check rejects Next.js CSS as executable script → `/_next/static/css/*.css` | Load the login page | Console error; `accessibility.md` § Console Errors Detected |
| **High** | `/en/api/sse` returns 404 (×4 occurrences) | Log in and navigate to any page after login | Console error; `snapshots/console-errors.md` |
| **Medium** | `/images/account-avatar.svg` returns 404 (×3 occurrences) | Navigate to any page after login | Console error; `snapshots/console-errors.md` |

These were observed against the public demo on 2026-06-23. They are not yet filed in a public upstream tracker (SOGo 6 repo is closed pre-release).

---

## SOGo 6 Demo — Accessibility (WCAG 2.2 AA) Open Issues

Automated axe-core scans on 2026-06-23 identified five violations present on every page tested (Login, Inbox, Contacts). Manual testing will surface additional issues.

| ID | Impact | Pages affected | Status |
|---|---|---|---|
| `html-has-lang` | Serious | All | Open |
| `page-has-heading-one` | Moderate | All | Open |
| `color-contrast` | Serious | All | Open |
| `button-name` | Critical | All | Open |
| `region` | Moderate | All | Open |

See `accessibility.md` for the full per-page breakdown and remediation guidance.

---

## SOGo 6 — Functional Gaps vs SOGo 5

The following capabilities are present in SOGo 5 but not yet available in the SOGo 6 pre-beta, or have been replaced by a different approach.

| Capability | SOGo 5 | SOGo 6 (pre-beta) | Notes |
|---|---|---|---|
| CardDAV address book | Supported | Not yet done | Per upstream: "Not the most complicated part" |
| ActiveSync (EAS) | Supported | Reconsidered for v1 | May arrive post-v1 |
| MAPI over HTTP | Not supported | Planned | Microsoft protocol replacement |
| `sogo.conf` file | Required | Replaced | Admin UI + ENV + database |
| Memcached | Caching layer | Replaced | Now uses Redis |
| AngularJS frontend | Default | Replaced | Now uses React/Next.js |
| Objective-C backend | Default | Replaced | Now uses Python/Flask |

Source: `features.md` (May 2026).

---

## SOGo 5.x — Security Floor

> **Important:** Anyone running SOGo 5.x in production as of June 2026 should be on **at least 5.12.8** (released 2026-05-12). The 5.12.7 and 5.12.8 releases address four critical vulnerabilities affecting prior versions.

| Release | Why |
|---|---|
| **5.12.8** (2026-05-12) | 2× XSS via malicious mail, 1× SQL injection, 1× OIDC user-source impersonation |
| **5.12.7** (2026-03-30) | CVE-2026-39178 (PostgreSQL user source), CVE-2026-39179 (SQL cleartext password) |
| **5.12.9** (2026-05-27) | Regression fixes for 5.12.8 (preferences save, search, event-invite display) |

Upstream tracker: https://github.com/Alinto/sogo/releases.  
Issue filing: https://bugs.sogo.nu (GitHub issue creation is restricted).

---

## SOGo 5.x — ActiveSync & Mobile Sync Pain (Last 90 Days)

Community forums (NethServer, YunoHost, mailcow) report recurring ActiveSync and mobile-sync issues in the 5.12.x line. All are upstream issues — no CoP-specific patches exist.

**Android calendar push bug.** When `SOGoURLEncryptionEnabled = YES` and ActiveSync pushes content to Android clients (Etar, Google Calendar) before the local sync cycle completes, events are deleted or duplicated. The workaround is to disable URL encryption, which carries a security cost. Reports recur across NethServer, YunoHost, and mailcow forums.

**ActiveSync regression after 5.12.x upgrades.** Multiple users on NethServer and YunoHost report that ActiveSync breaks entirely after updating to certain 5.12.x builds. The root cause appears to be nginx proxy config changes shipped with the packages. Downgrading to `5.12.x-1` recovers functionality.

**iOS 18.4 CalDAV URL encoding.** Apple's iOS 18.4 began percent-encoding `@` characters in CalDAV URLs, breaking calendar sync. The fix shipped in SOGo 5.12.x decodes the encoded `@` back before routing.

**Outlook `valarm` ordering.** SOGo 5.12.1 moved `valarm` blocks to the end of `VEVENT` iCalendar entries for compatibility with `outlook.office.com`. This changes the existing behaviour for custom calendar clients that depend on the previous ordering.

All of the above are upstream issues — no CoP-specific patches. If you depend on ActiveSync, pin a known-good version and test upgrades in a staging environment before rolling out.

---

## Reporting Issues

- **SOGo 5.x**: File at https://bugs.sogo.nu or email `users@sogo.nu`. GitHub issue creation is restricted.
- **SOGo 6**: There is no public issue tracker yet. Document findings locally (or in this repo's `snapshots/`) and share with the CoP until the upstream repo is public.
- **This Community of Practice repo**: See `SECURITY.md` for how to report CoP-doc issues.

---

## How to Contribute Updates

- Add a new entry under the appropriate section.
- Mark closed items with **Resolved in `<version>`** (strike-through the row or move it to a "Closed" subsection).
- Update `Last updated:` at the top.

---

Maintained by the SOGo Community of Practice. Not affiliated with the SOGo project.
