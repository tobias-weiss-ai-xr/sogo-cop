# Accessibility Evaluation — SOGo 6 Demo

**Target:** [demov6.sogo.nu](https://demov6.sogo.nu/SOGo/)  
**Standard:** WCAG 2.2 Level AA  
**Stack:** React.js / Next.js (frontend) — Python Flask REST API (backend)  
**Status:** Pre-beta preview — UI is subject to change.  
**Last tested:** 2026-06-23  
**Testing method:** Playwright accessibility snapshots + axe-core v4.10 automated scans

> "Keep in mind that the UI can still change and that the demo does not show the current state of our development progress."
> — SOGo Team, May 2026

---

## Why Accessibility Matters for SOGo

SOGo is widely deployed in **academic institutions** and **public-sector organisations** across Europe. These environments are frequently subject to:

- **EU European Accessibility Act (EAA)** — enforceable June 2025 for public-sector digital services
- **EN 301 549** — EU standard harmonised with WCAG 2.2
- **BITV 2.0** (Germany), **RGAA** (France), **Stanca Act** (Italy) — national implementations
- **Section 508** (US) for institutions with US funding ties

An inaccessible SOGo 6 could be a blocker for institutional adoption, especially given that SOGo 5's AngularJS frontend was notoriously difficult to retrofit for accessibility.

---

## Evaluation Methodology

### Tools

| Tool | Purpose | Phase |
|---|---|---|
| axe DevTools (browser extension) | Automated WCAG violation detection | Scan |
| Lighthouse Accessibility audit | Automated score + recommendations | Scan |
| WAVE (WebAIM) | Visual overlay of issues | Scan |
| Manual keyboard testing | Tab order, focus visibility, focus traps | Manual |
| NVDA / Firefox | Screen reader (Windows) | Manual |
| VoiceOver / Safari | Screen reader (macOS) | Manual |
| Colour Contrast Analyser (TPGi) | Contrast ratios | Scan |
| eslint-plugin-jsx-a11y (static analysis) | React-specific a11y lint | Code review |

### Process

1. **Automated scan** — axe, Lighthouse, WAVE on each major page
2. **Keyboard audit** — Tab through every interactive element without a mouse
3. **Screen reader audit** — NVDA + VoiceOver through critical paths
4. **React-specific audit** — Focus management, ARIA state sync, live regions
5. **Contrast audit** — All text/background combinations against WCAG 2.2 AA
6. **Reporting** — Document findings with severity, location, and remediation

---

## Pages / Views to Evaluate

| # | View | Critical for | Priority |
|---|---|---|---|
| 1 | **Login page** | All users (entry point) | P0 |
| 2 | **Mail — Inbox list** | Daily email users | P0 |
| 3 | **Mail — Message view** | Reading emails | P0 |
| 4 | **Mail — Compose** | Writing emails | P0 |
| 5 | **Mail — Search** | Finding messages | P1 |
| 6 | **Calendar — Month view** | Overview scheduling | P0 |
| 7 | **Calendar — Event create/edit** | Scheduling meetings | P0 |
| 8 | **Calendar — Week/Day view** | Detailed planning | P1 |
| 9 | **Contacts — List** | Address book browsing | P1 |
| 10 | **Contacts — Edit** | Contact management | P1 |
| 11 | **Settings / Preferences** | User configuration | P2 |
| 12 | **Admin interface** | Administrator tasks | P2 |

### Critical Paths (must pass)

```
Login → Inbox → Open email → Reply
Login → Calendar → Create event → Save
Login → Contacts → Search → Edit contact
Login → Settings → Change preference
```

---

## WCAG 2.2 AA Checklist (Tailored for SOGo 6 / React)

### 1. Perceivable

#### 1.1 Text Alternatives
- [ ] All non-text content (icons, buttons, avatars) has a text alternative
- [ ] Decorative icons are marked with `aria-hidden="true"` or `role="presentation"`
- [ ] Icon-only buttons (e.g., reply, delete, flag) have accessible labels
- [ ] Email attachment icons have meaningful alt text

#### 1.2 Time-based Media
- [ ] No auto-playing video/audio without controls (unlikely in SOGo, but check)

#### 1.3 Adaptable
- [ ] Heading hierarchy is logical (h1 → h2 → h3), no skips
- [ ] Landmarks used correctly: `<header>`, `<nav>`, `<main>`, `<aside>`
- [ ] Email content preserves semantic structure (headings, lists, tables)
- [ ] Calendar grid is marked up as a proper table/grid role

#### 1.4 Distinguishable
- [ ] **Colour contrast**: All text 4.5:1 (normal) / 3:1 (large) — test on:
  - Email subject lines in inbox
  - Calendar event text
  - Button text and backgrounds
  - Navigation links
  - Disabled/read-only states
- [ ] Focus indicators are visible (minimum 2px outline, 3:1 contrast against adjacent)
- [ ] Colour alone is not used to convey information (e.g., unread/read emails use icon + weight, not just colour)
- [ ] Text can be resized to 200% without loss of content or functionality
- [ ] No colour combinations that cause issues for colour vision deficiencies (simulate with tools)
- [ ] Dark mode / high-contrast mode support

### 2. Operable

#### 2.1 Keyboard Accessible
- [ ] **All actions** operable by keyboard alone — verify on:
  - Inbox navigation (up/down arrows, Enter to open)
  - Compose form (Tab through fields)
  - Calendar date picker (arrow keys)
  - Context menus (right-click alternatives)
  - Drag-and-drop (keyboard alternative required)
- [ ] No keyboard traps — focus never gets stuck in a widget
- [ ] No `tabindex` values > 0 (use logical DOM order instead)

#### 2.2 Enough Time
- [ ] Session timeout has a warning (at least 20 seconds) and option to extend
- [ ] Auto-save drafts announced to screen reader users
- [ ] No time limits on compose without save
- [ ] No moving/blinking/auto-updating content without pause mechanism

#### 2.3 Seizures
- [ ] No flashing content (more than 3 flashes per second)

#### 2.4 Navigable
- [ ] **Skip links**: "Skip to content" / "Skip to navigation" visible on focus
- [ ] Page titles are descriptive (update on route change)
- [ ] **Focus order** matches visual order — test with Tab key
- [ ] **Focus management on route change**: After navigation, focus moves to `<h1>` or a dedicated skip target (Next.js SPA challenge!)
- [ ] **Focus management on modal open/close**: Trap focus in modal, return to trigger on close
- [ ] Link text is descriptive (not "click here", "read more")
- [ ] Breadcrumb navigation (if present) is properly structured

#### 2.5 Input Modalities
- [ ] All functionality available via single-pointer actions
- [ ] No complex gestures (pinch, multi-finger) required for core functions
- [ ] Undo option for destructive actions (delete email, delete event)

### 3. Understandable

#### 3.1 Readable
- [ ] Language attribute set on `<html>` (and changes for multilingual content)
- [ ] Abbreviations/jargon explained where appropriate

#### 3.2 Predictable
- [ ] Navigation is consistent across pages
- [ ] Same icon/button always does the same thing
- [ ] No unexpected context changes on focus or input
- [ ] Consistent component positioning (compose button always in same area)

#### 3.3 Input Assistance
- [ ] **Labels** — all form fields have visible, programmatically associated labels
- [ ] **Error identification** — errors are described in text, not just colour
- [ ] **Error suggestions** — suggestions for correction where possible
- [ ] **Error grouping** — errors listed at top of form + inline
- [ ] **ARIA live regions** — dynamic updates (new email, save confirmation) announced
- [ ] Required fields are indicated (visually and programmatically)

### 4. Robust

#### 4.1 Compatible
- [ ] Valid HTML (no duplicate IDs, proper nesting)
- [ ] ARIA attributes used correctly (no invalid roles, no overridden semantics)
- [ ] Custom components have correct roles, states, and properties
- [ ] React state synced with ARIA attributes (`aria-expanded={isOpen}`, not static)

#### 4.1 React-Specific Robustness
- [ ] `useId()` used for accessible ID references (not random/manual IDs)
- [ ] Portals (modals, dropdowns) handle focus correctly
- [ ] Server-side rendering (Next.js) doesn't cause hydration mismatches that break a11y
- [ ] `aria-live` regions for:
  - New email notifications
  - Save/delete confirmations
  - Error messages from API calls
  - Loading states (spinner, skeleton)

---

## React / Next.js Pitfalls to Watch For

Because SOGo 6 is a React/Next.js application, these framework-specific patterns MUST be evaluated:

| Pattern | Risk | What to Check |
|---|---|---|
| **Route transitions** | Focus stays on old page element | After login → inbox, does focus move? |
| **Modal dialogs** | No focus trap, no return on close | Compose modal, confirm dialogs |
| **Controlled components** | ARIA not synced with state | `aria-expanded` on menus |
| **Async data loading** | Content appears without announcement | Email list loading, search results |
| **Portal rendering** | Focus is lost outside React tree | Dropdowns, popovers, tooltips |
| **CSS-in-JS / Tailwind** | Focus styles stripped or insufficient | Visible focus ring on all elements |
| **Custom form controls** | No native semantics | Tag editor, date picker, rich text editor |

---

## Automated vs Manual Testing Split

```
Automated (axe/Lighthouse/WAVE):    ~30% of issues
Manual keyboard-only:               ~25%
Manual screen reader (NVDA/VoiceOver): ~30%
Manual visual inspection:           ~15%
```

Automated tools catch missing labels, contrast errors, and invalid ARIA.  
They **cannot** catch:
- Whether focus order is logical
- Whether screen readers announce route changes
- Whether error messages are actually helpful
- Whether keyboard navigation feels natural

**Both are required.**

---

## Severity Classification

| Level | Definition | Response |
|---|---|---|
| **Critical** | Blocks core task for AT user (login, read email, create event) | Must fix before beta |
| **High** | Major friction for AT users (focus lost, missing labels, no skip link) | Should fix before GA |
| **Medium** | Annoying but workable (suboptimal announcement, minor contrast) | Fix before GA |
| **Low** | Polish (cosmetic contrast, verbose announcements) | Post-GA enhancement |
| **Info** | Observation, not a violation | Note for roadmap |

---

## Reporting Template

Each finding should follow this structure:

```markdown
### [SEVERITY] Finding title

- **Page:** [URL or view name]
- **Element:** [CSS selector or description]
- **WCAG Criterion:** [e.g., 2.4.3 Focus Order]
- **Issue:** [Description of the problem]
- **Expected:** [What should happen]
- **Actual:** [What actually happens]
- **Impact:** [Who is affected and how]
- **Recommendation:** [How to fix]
- **Screenshot:** [link to annotated screenshot]
```

---

## Initial Automated Findings (axe-core, 2026-06-23)

Automated scans were run on three key pages using axe-core v4.10.2 via Playwright. These represent a baseline — manual testing will find additional issues.

### Summary Table

| Page | Violations | Passes | Incomplete |
|---|---|---|---|
| **Login** (`/en/auth/login`) | 6 | 40+ | 2 |
| **Inbox** (`/en/u/0/INBOX`) | 13 | 40+ | 2 |
| **Contacts** (`/en/address_books/work`) | 9 | 40+ | 2 |

### Cross-Page Issues (found on ALL pages)

| ID | WCAG | Impact | Description | Prevalence |
|---|---|---|---|---|
| `html-has-lang` | 3.1.1 Language of Page | **Serious** | `<html>` element missing `lang` attribute | All pages |
| `page-has-heading-one` | Best practice | **Moderate** | No `<h1>` heading found | All pages |
| `color-contrast` | 1.4.3 Contrast (AA) | **Serious** | Multiple elements fail 4.5:1 ratio | All pages |
| `button-name` | 4.1.2 Name, Role, Value | **Critical** | Icon-only buttons without accessible names | All pages |
| `region` | Best practice | **Moderate** | Content not contained by landmarks | All pages |

### Login Page Findings

| ID | Impact | Nodes | Details |
|---|---|---|---|
| `button-name` | Critical | 1 | Unnamed button (likely the theme toggle overflow button) |
| `color-contrast` | Serious | 4 | Text/background insufficient contrast |
| `html-has-lang` | Serious | 1 | `<html>` missing `lang` attribute |
| `landmark-one-main` | Moderate | 1 | No `<main>` landmark |
| `page-has-heading-one` | Moderate | 1 | No `<h1>` |
| `region` | Moderate | 4 | Content outside landmarks |

### Inbox Page Findings

| ID | Impact | Nodes | Details |
|---|---|---|---|
| `aria-allowed-attr` | Critical | 3 | Unsupported ARIA attributes on elements (e.g., `aria-expanded` on non-interactive div) |
| `aria-valid-attr-value` | Critical | 1 | Invalid `aria-controls` reference (Radix UI dynamic ID mismatch) |
| `button-name` | Critical | 13 | Icon-only sidebar action buttons (context menus, collapse) without labels |
| `color-contrast` | Serious | 11 | Sidebar text, email preview text below 4.5:1 |
| `html-has-lang` | Serious | 1 | Missing `lang` |
| `list` | Serious | 2 | `<ul>` has non-`<li>` direct children (Radix UI sidebar) |
| `listitem` | Serious | 1 | `<li>` outside `<ul>`/`<ol>` parent |
| `nested-interactive` | Serious | 1 | Interactive control nested inside another (email action buttons) |
| `landmark-main-is-top-level` | Moderate | 1 | `<main>` nested inside another landmark |
| `landmark-no-duplicate-main` | Moderate | 1 | More than one `<main>` landmark |
| `landmark-unique` | Moderate | 1 | Landmarks lack unique labels |
| `page-has-heading-one` | Moderate | 1 | No `<h1>` |
| `region` | Moderate | 2 | Content outside landmarks |

### Contacts Page Findings

| ID | Impact | Nodes | Details |
|---|---|---|---|
| `aria-allowed-attr` | Critical | 1 | `aria-expanded="false"` on a non-interactive dropdown trigger |
| `aria-valid-attr-value` | Critical | 1 | Invalid `aria-controls` reference (Radix UI) |
| `button-name` | Critical | 3 | Unnamed sidebar action buttons |
| `color-contrast` | Serious | 9 | **Notable**: Sidebar group labels at **3.06:1** (requires 4.5:1). Button text on teal background `#4d8080` with white text at **4.45:1** — borderline fail for normal text |
| `html-has-lang` | Serious | 1 | Missing `lang` |
| `list` | Serious | 1 | `<ul>` with non-`<li>` children |
| `listitem` | Serious | 2 | `<li>` outside `<ul>`/`<ol>` |
| `page-has-heading-one` | Moderate | 1 | No `<h1>` |
| `region` | Moderate | 5 | Sidebar group labels outside landmarks |

### Console Errors Detected

```
1. [ERROR] Refused to execute script from '/_next/static/css/0127c82b1dfaee58.css'
           → MIME type 'text/css' is not executable (strict MIME checking)
           → Login page (persists across all pages)
2. [ERROR] 404 — /en/api/sse  (×4 occurrences)
           → Server-Sent Events endpoint not found
           → Occurs after login (likely SSE for real-time notifications)
3. [ERROR] 404 — /images/account-avatar.svg  (×3 occurrences)
           → Missing avatar image resource
           → All pages after login
```

**« html lang » missing (Critical)** — `<html class="__variable_fb8f2c ... light" style="color-scheme: light;">` has no `lang` attribute. Screen readers will not auto-detect page language.

---

## Accessibility Snapshot Observations (Playwright Accessibility Tree)

The following observations draw from the rendered accessibility tree, not axe automation alone.

### Login Page (`/en/auth/login`)
- **Good**: Email textbox has visible `<label>` and placeholder; Language combobox has accessible name; SOGo logo has `alt="SOGo"`.
- **Good**: Theme buttons "Dark", "Light", "Auto" have discernible names.
- **Observation**: No `<h1>` heading — the page title exists as a meta title but not as an ARIA heading. Logo is an `<img>` without a heading role.
- **Observation**: The email field is pre-filled with `sogo-tests1@example.org` — convenient for demo, but real deployments should not persist credentials.

### Password Step (`login/pwd`)
- **Good**: "Password" textbox has visible label and placeholder "Enter your password". Password is pre-filled with `sogo` for demo.
- **Good**: "Remember me" checkbox has associated label.
- **Good**: "Log in" button visible.
- **Observation**: "Email" is shown as plain text `sogo-tests1@example.org` instead of as a form field — no ability to change email without going back.

### Inbox (`/en/u/0/INBOX`)
- **Good**: Tabs (Mail, Address Books, Calendars, Tasks) are proper `tablist`/`tab` roles.
- **Good**: Filter radios (All, Read, Unread, Starred, Attachments) are in a `<group>` role.
- **Good**: View mode radios (Full view, Split view) are in a `<group>` role.
- **Good**: Email list items are buttons with descriptive accessible names including sender, subject, preview, and date.
- **Good**: Pagination controls disabled/enabled correctly; page indicator shows "1 / 3".
- **Good**: "Toggle Sidebar" has visible accessible name.
- **Good**: Folder badges show unread count (e.g., "Inbox 5", "Drafts 2", "Junk 1").
- **Issue**: **13 unnamed buttons** — sidebar action buttons (context menus for folders) have no accessible label. These are `data-sidebar="menu-action"` buttons with only an SVG icon.
- **Issue**: **Nested interactive** — email row contains a button that itself contains interactive elements.
- **Issue**: **Duplicate `<main>`** — both the sidebar region and the content area contain a `<main>` landmark. Only one should be present.

### Message Detail (`/en/u/0/INBOX/inbox_001`)
- **Good**: Subject rendered as `<h1>` heading — `"Entretien candidat — poste développeur"`.
- **Good**: "From" (David Gueto) and "To" (John Paul) are buttons for contact actions.
- **Good**: Action toolbar with"More actions" named button.
- **Issue**: Several toolbar icon buttons lack accessible names (reply, forward, archive, delete, more actions — icons only).

### Calendar (Week View)
- **Good**: Day headers are proper buttons with accessible names ("21 Sun", "22 Mon", etc.).
- **Good**: Previous/Today/Next navigation buttons, view combobox (Week), timezone combobox.
- **Good**: Events are buttons with text labels ("Team Standup", "Doctor Appt", etc.).
- **Good**: Calendar checkboxes for toggling visibility (Personal, Birthdays, Team Calendar, etc.).
- **Issue**: The week view uses `rowgroup`/`row` roles with generic `<div>` elements rather than a proper `<table>` grid for the time slots.

### Contacts (`/en/address_books/work`)
- **Good**: "Search contacts…" textbox has placeholder.
- **Good**: Address book navigation uses proper `<heading level=2>` for "Distribution lists" and "Contacts" sections.
- **Good**: Contact rows show initials + name in descriptive buttons.
- **Good**: "Select a contact" prompt displayed when nothing selected.
- **Issue**: **Contrast** — sidebar group labels `#cad9d9` on `#4d8080` bg = **3.06:1** (fail). Menu item text `#ffffff` on `#4d8080` bg = **4.45:1** (borderline fail for small text).

---

## References

- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [EN 301 549](https://www.etsi.org/deliver/etsi_en/301500_301599/301549/03.02.01_60/en_301549v030201p.pdf)
- [Next.js Accessibility docs](https://nextjs.org/docs/accessibility)
- [React Accessibility](https://react.dev/reference/react/useId)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [axe DevTools](https://www.deque.com/axe/)
- [NVDA screen reader](https://www.nvaccess.org/)
- [TPGi Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/)
