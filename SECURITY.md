# Security Policy

This document outlines the security reporting process for the SOGo Community of Practice
documentation repository and points upstream for vulnerabilities in the SOGo product itself.

## Reporting a Vulnerability in this CoP

This repository is documentation-only. However, if you find a security-relevant issue in
the documentation — for example, an accidentally committed credential, a misleading
security claim, or a broken link pointing to a malicious mirror — please report it.

* Open a **private** GitHub issue by contacting the maintainers directly, or
* Send an email to `security@sogo-cop.example` (placeholder — replace before going public).

We aim to acknowledge receipt within **7 days** and will work with you to resolve the
issue promptly.

## Reporting a Vulnerability in SOGo Itself

For vulnerabilities in the SOGo product itself (sogod, SOGo 6 backend/frontend,
ActiveSync, CalDAV/CardDAV, etc.), **do NOT report here**. Contact the SOGo team
directly:

* Issue tracker: https://bugs.sogo.nu
* Mailing list: `users@sogo.nu`

Refer to the upstream SOGo SECURITY policy for details.

## Supported Versions

| CoP docs version | Supported | Notes |
|-----------------|-----------|-------|
| main (rolling)  | Yes       | This CoP follows the rolling main branch. We backport documentation fixes for the last 12 months of meeting notes. |

## Acknowledgements

Security fixes are credited in the commit log and acknowledged in the next CoP meeting
notes unless the reporter requests anonymity.
