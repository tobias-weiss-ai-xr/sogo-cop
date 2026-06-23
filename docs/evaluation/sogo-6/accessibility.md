# Accessibility Evaluation — SOGo 6 Demo

**Target:** [demov6.sogo.nu](https://demov6.sogo.nu/SOGo/)  
**Standard:** WCAG 2.2 Level AA  
**Stack:** React.js / Next.js (frontend) — Python Flask REST API (backend)  
**Status:** Pre-beta preview — UI is subject to change.

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

## References

- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [EN 301 549](https://www.etsi.org/deliver/etsi_en/301500_301599/301549/03.02.01_60/en_301549v030201p.pdf)
- [Next.js Accessibility docs](https://nextjs.org/docs/accessibility)
- [React Accessibility](https://react.dev/reference/react/useId)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [axe DevTools](https://www.deque.com/axe/)
- [NVDA screen reader](https://www.nvaccess.org/)
- [TPGi Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/)
