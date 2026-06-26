# SOGo Theming Guide

How to brand and customize the SOGo web interface — from basic logo swaps to full
color-theme overrides, on bare-metal, Docker, and Kubernetes (Helm) deployments.

<!-- TOC -->
* [SOGo Theming Guide](#sogo-theming-guide)
  * [Quick start](#quick-start)
  * [Theming layers](#theming-layers)
  * [Layer 1 — sogo.conf preferences](#layer-1--sogoconf-preferences)
  * [Layer 2 — Angular Material color theme](#layer-2--angular-material-color-theme)
  * [Layer 3 — Custom CSS injection](#layer-3--custom-css-injection)
  * [Layer 4 — Static asset replacement](#layer-4--static-asset-replacement)
  * [Docker / Podman theming](#docker--podman-theming)
  * [Kubernetes / Helm theming](#kubernetes--helm-theming)
  * [Portal-level theming (openDesk)](#portal-level-theming-opendesk)
  * [SOGo 6 theming outlook](#sogo-6-theming-outlook)
  * [Troubleshooting](#troubleshooting)
<!-- TOC -->

## Quick start

The three things most people want:

| Goal | Method | File / Setting |
|------|--------|----------------|
| Change page title | `sogo.conf` | `SOGoPageTitle = "My Mail"` |
| Replace logo | Static file swap | `sogo-full.svg`, `sogo-short.svg` |
| Change colors | Angular Material theme JS | `js/theme.js` → compile to `css/theme-default.css` |

## Theming layers

SOGo's web interface (v5.x) is built on AngularJS + Angular Material. Theming can
be applied at four independent layers, each overriding the previous:

1. **sogo.conf preferences** — page title, language, timezone
2. **Angular Material color theme** — primary/accent/background palettes via JS
3. **Custom CSS injection** — arbitrary style overrides via `SOGoUIAdditionalCSSFiles`
4. **Static asset replacement** — logos, favicons, images

All layers compose. You can mix a custom color theme (layer 2) with asset replacements
(layer 4) without conflicts.

## Layer 1 — sogo.conf preferences

These are set in `/etc/sogo/sogo.conf` (or the equivalent mounted config file):

```
SOGoPageTitle  = "My University Mail";
SOGoLanguage   = German;
SOGoTimeZone   = "Europe/Berlin";
```

For Kubernetes deployments using the opendesk-edu Helm chart, these map to `values.yaml`:

```yaml
sogo:
  pageTile: "My University Mail"
  language: "German"
  timezone: "Europe/Berlin"
```

The ConfigMap template at `helmfile/charts/sogo/templates/configmap.yaml` renders these
as `SOGoPageTitle`, `SOGoLanguage`, and `SOGoTimeZone` in the `90-helm.yaml` drop-in.

## Layer 2 — Angular Material color theme

SOGo 5.x uses AngularJS Material's `$mdThemingProvider` to define all interface colors.
To override the default palette:

### Step 1 — Create a theme JS file

Create `js/theme.js` (or any filename) under the SOGo WebServerResources directory:

```
/usr/lib/GNUstep/SOGo/WebServerResources/js/theme.js
```

```javascript
(function() {
  "use strict";

  angular.module("SOGo.Common")
    .config(["$mdThemingProvider", function($mdThemingProvider) {

      $mdThemingProvider.definePalette("brand-primary", {
        "50":  "#e8eaf6",
        "100": "#c5cae9",
        "200": "#9fa8da",
        "300": "#7986cb",
        "400": "#5c6bc0",
        "500": "#3f51b5",
        "600": "#3949ab",
        "700": "#303f9f",
        "800": "#283593",
        "900": "#1a237e",
        "A100": "#8c9eff",
        "A200": "#536dfe",
        "A400": "#3d5afe",
        "A700": "#304ffe",
        "contrastDefaultColor": "light",
        "contrastDarkColors": ["50", "100", "200", "300", "A100"],
        "contrastLightColors": undefined
      });

      $mdThemingProvider.definePalette("brand-accent", {
        "50":  "#fce4ec",
        "100": "#f8bbd0",
        "200": "#f48fb1",
        "300": "#f06292",
        "400": "#ec407a",
        "500": "#e91e63",
        "600": "#d81b60",
        "700": "#c2185b",
        "800": "#ad1457",
        "900": "#880e4f",
        "A100": "#ff80ab",
        "A200": "#ff4081",
        "A400": "#f50057",
        "A700": "#c51162",
        "contrastDefaultColor": "light",
        "contrastDarkColors": ["50", "100", "200"],
        "contrastLightColors": undefined
      });

      $mdThemingProvider.theme("default")
        .primaryPalette("brand-primary", {
          "default": "500",
          "hue-1":  "500",
          "hue-2":  "800",
          "hue-3":  "A700"
        })
        .accentPalette("brand-accent", {
          "default": "500",
          "hue-1":  "400",
          "hue-2":  "600",
          "hue-3":  "A700"
        })
        .backgroundPalette("grey");

      $mdThemingProvider.generateThemesOnDemand(true);
    }]);
})();
```

### Step 2 — Register the theme file in sogo.conf

```
SOGoUIAdditionalJSFiles = (js/theme.js);
```

### Step 3 — Compile the theme to CSS

SOGo caches compiled CSS. Enable debug mode, extract the generated CSS, then disable
debug mode:

1. Set `SOGoUIxDebugEnabled = YES;` in `sogo.conf`
2. Restart sogod
3. Open SOGo in a browser, open the JavaScript console, and run:

```javascript
copy([].slice.call(document.styleSheets)
  .map(e => e.ownerNode)
  .filter(e => e.hasAttribute('md-theme-style'))
  .map(e => e.textContent)
  .join('\n'))
```

4. Save the clipboard content to
   `WebServerResources/css/theme-default.css`
5. Set `SOGoUIxDebugEnabled = NO;` (or remove it)
6. Restart sogod

> **Why the CSS step?** With `SOGoUIxDebugEnabled = NO` (production), SOGo serves
> the pre-compiled `theme-default.css` file instead of running the JS theming engine
> on every request. Skipping this step means your color changes only take effect in
> debug mode.

### Palette reference

| Palette slot | What it controls |
|---|---|
| `primaryPalette.default` | Top toolbar background |
| `primaryPalette.hue-1` | Top toolbar active elements |
| `primaryPalette.hue-2` | Sidebar toolbar background |
| `primaryPalette.hue-3` | Emphasis / focus states |
| `accentPalette.default` | FAB buttons, login screen accent |
| `accentPalette.hue-1` | Center list toolbar |
| `accentPalette.hue-2` | Selected mail, current calendar day |
| `backgroundPalette` | Page background, cards, dividers |

See the [AngularJS Material Theming docs](https://material.angularjs.org/latest/Theming/01_introduction)
for the full palette format.

## Layer 3 — Custom CSS injection

For fine-grained overrides that go beyond color palettes (spacing, font sizes,
hiding elements), use `SOGoUIAdditionalCSSFiles`:

```
SOGoUIAdditionalCSSFiles = ("css/custom.css");
```

Place `custom.css` in the `WebServerResources/css/` directory. Example overrides:

```css
/* Hide the SOGo logo on the login page */
.loginLogo {
  display: none;
}

/* Change the login background */
.md-page-content {
  background-color: #1a237e;
}

/* Adjust mail list row height */
.md-virtual-repeat-container .md-virtual-repeat-offsetter > * {
  min-height: 48px;
}
```

Multiple CSS files are supported:

```
SOGoUIAdditionalCSSFiles = ("css/custom.css", "css/org.css");
```

## Layer 4 — Static asset replacement

SOGo serves static assets from the `WebServerResources/` directory:

```
/usr/lib/GNUstep/SOGo/WebServerResources/
  img/sogo-full.svg          # Login page and header logo (full)
  img/sogo-short.svg         # Compact logo (favicon area)
  img/sogo-logo.png          # PNG version
```

To replace assets in Docker or Kubernetes, mount over the defaults:

**Docker Compose:**

```yaml
services:
  sogo:
    image: sonroyaalmerol/docker-sogo:5.12.8-2
    volumes:
      - ./branding/sogo-full.svg:/usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-full.svg:ro
      - ./branding/sogo-short.svg:/usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-short.svg:ro
```

**Kubernetes (Helm values):**

```yaml
extraVolumes:
  - name: branding
    configMap:
      name: sogo-branding
extraVolumeMounts:
  - name: branding
    mountPath: /usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-full.svg
    subPath: sogo-full.svg
    readOnly: true
  - name: branding
    mountPath: /usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-short.svg
    subPath: sogo-short.svg
    readOnly: true
```

> **SVG format recommended.** SOGo renders logos via `<img>` tags. SVG scales without
> quality loss and is smaller than equivalent PNGs.

### Favicons

Replace the default favicon by swapping `sogo.ico` in the `WebServerResources/` root.
Both `.ico` and `.png` files are accepted (rename `.png` to `.ico`).

## Docker / Podman theming

### Approach A — Volume mounts (recommended)

Mount custom files over the container defaults. Works for any SOGo image.

```yaml
services:
  sogo:
    image: sonroyaalmerol/docker-sogo:5.12.8-2
    volumes:
      - sogo-data:/var/lib/sogo
      - ./branding/sogo-full.svg:/usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-full.svg:ro
      - ./branding/sogo-short.svg:/usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-short.svg:ro
      - ./branding/theme.js:/usr/lib/GNUstep/SOGo/WebServerResources/js/theme.js:ro
      - ./branding/theme-default.css:/usr/lib/GNUstep/SOGo/WebServerResources/css/theme-default.css:ro
      - ./branding/custom.css:/usr/lib/GNUstep/SOGo/WebServerResources/css/custom.css:ro
      - ./config/90-theme.conf:/etc/sogo/sogo.conf.d/90-theme.conf:ro
```

With `90-theme.conf` containing:

```
SOGoPageTitle = "My Organization Mail";
SOGoUIAdditionalJSFiles = (js/theme.js);
SOGoUIAdditionalCSSFiles = ("css/custom.css");
```

### Approach B — Custom Docker image

Build an image layer on top of the upstream SOGo image:

```dockerfile
FROM sonroyaalmerol/docker-sogo:5.12.8-2

COPY branding/sogo-full.svg  /usr/lib/GNUstep/SOGo/WebServerResources/img/
COPY branding/sogo-short.svg /usr/lib/GNUstep/SOGo/WebServerResources/img/
COPY branding/theme.js         /usr/lib/GNUstep/SOGo/WebServerResources/js/
COPY branding/theme-default.css /usr/lib/GNUstep/SOGo/WebServerResources/css/
COPY branding/custom.css      /usr/lib/GNUstep/SOGo/WebServerResources/css/
COPY config/90-theme.conf     /etc/sogo/sogo.conf.d/
```

This is more portable (no volume-mount coordination needed) but requires rebuilding
when branding changes.

## Kubernetes / Helm theming

### opendesk-edu Helm chart (SOGo 5.x)

The [opendesk-edu SOGo chart](https://github.com/opendesk-edu/opendesk-edu/tree/main/helmfile/charts/sogo)
exposes basic branding through `values.yaml`:

```yaml
sogo:
  pageTile: "openDesk Mail"    # → SOGoPageTitle
  language: "English"          # → SOGoLanguage
  timezone: "Europe/Berlin"    # → SOGoTimeZone
```

For deeper theming (colors, logos, CSS), use extra volumes and ConfigMaps:

```yaml
sogo:
  pageTile: "My University Mail"
  language: "German"

extraVolumes:
  - name: sogo-branding
    configMap:
      name: sogo-branding
      defaultMode: 0644

extraVolumeMounts:
  - name: sogo-branding
    mountPath: /usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-full.svg
    subPath: sogo-full.svg
    readOnly: true
  - name: sogo-branding
    mountPath: /usr/lib/GNUstep/SOGo/WebServerResources/img/sogo-short.svg
    subPath: sogo-short.svg
    readOnly: true
  - name: sogo-branding
    mountPath: /usr/lib/GNUstep/SOGo/WebServerResources/css/theme-default.css
    subPath: theme-default.css
    readOnly: true
  - name: sogo-branding
    mountPath: /usr/lib/GNUstep/SOGo/WebServerResources/css/custom.css
    subPath: custom.css
    readOnly: true
```

Create the ConfigMap from your branding files:

```bash
kubectl create configmap sogo-branding \
  --from-file=sogo-full.svg=branding/sogo-full.svg \
  --from-file=sogo-short.svg=branding/sogo-short.svg \
  --from-file=theme-default.css=branding/theme-default.css \
  --from-file=custom.css=branding/custom.css \
  -n sogo
```

### sonroyaalmerol/docker-sogo Helm chart

The community [Helm chart](https://github.com/sonroyaalmerol/docker-sogo/tree/main/charts/sogo)
supports SOGo configuration via `.sogo.configs` in `values.yaml`. Theming keys
can be placed there:

```yaml
sogo:
  configs:
    SOGoPageTitle: "My Mail"
    SOGoUIAdditionalJSFiles: (js/theme.js)
    SOGoUIAdditionalCSSFiles: ("css/custom.css")
```

## Portal-level theming (openDesk)

In openDesk / openDesk Edu deployments, SOGo runs alongside other apps behind a
unified portal (Nubus). Theming is managed centrally through:

```
helmfile/environments/default/theme.yaml.gotmpl
helmfile/files/theme/
```

The central theme defines:
- **Colors**: primary, secondary, background palette values (hex codes)
- **Images**: logos and favicons for each service, base64-encoded as Go template values
- **Stylesheets**: portal-specific CSS

Theme propagation flow:

```
theme.yaml.gotmpl        → helmfile renders values
  ↓
helmfile/files/theme/     → logo/icon source files per service
  ↓
per-chart values          → each chart receives theme.* via global values
  ↓
ConfigMaps / volumeMounts → charts mount branded assets into containers
```

### Current SOGo theme support in openDesk

The `theme.yaml.gotmpl` already includes groupware image entries:

```yaml
imager:
  groupware:
    faviconIco: {{ readFile "./../../files/theme/groupware_mail/favicon.ico" | b64enc | quote }}
    faviconSvg: {{ readFile "./../../files/theme/groupware_mail/favicon.svg" | b64enc | quote }}
```

However, the opendesk-edu SOGo chart (`helmfile/charts/sogo`) does not yet mount
these favicons into the SOGo container. The color palette from `theme.colors` is
also not passed to SOGo's Angular Material theme system.

To bridge this gap in a deployment, use the extra-volume approach described in the
Kubernetes section above, referencing the same source files from `helmfile/files/theme/`.

### Proposed enhancement for the SOGo Helm chart

A future chart update could automate theme propagation by adding:

1. A `theme` block in `values.yaml` to accept the central theme values:

```yaml
sogo:
  theme:
    faviconIco: ""     # base64 from theme.yaml.gotmpl
    faviconSvg: ""     # base64 from theme.yaml.gotmpl
    primaryColor: ""   # hex from theme.colors.primary
    accentColor: ""    # hex from theme.colors.secondaryBlue
```

2. A template that generates the `theme.js` file from the color values

3. Volume mounts for the favicon and generated CSS

This would make SOGo theming a one-line enablement in `theme.yaml.gotmpl`,
consistent with how other openDesk components handle branding.

## SOGo 6 theming outlook

SOGo 6 (pre-beta, Summer 2026) replaces AngularJS with Angular 16+ and CKEditor 5.
The Angular Material theming system changes significantly:

- **No more `$mdThemingProvider`** — Angular Material v16+ uses CSS custom properties
  (`--mat-*` variables) and the `@use` directive with SCSS
- **Theme configuration moves to SCSS** instead of runtime JS
- **CSS custom properties** allow runtime color changes without recompilation
- **`SOGoUIAdditionalJSFiles`** will likely be replaced or deprecated
- **`SOGoUIAdditionalCSSFiles`** may continue to work for injection

If you are planning a migration to SOGo 6, invest in CSS-variable-based overrides
(layer 3) rather than AngularJS Material JS themes (layer 2), as these are more
future-proof.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Theme JS loads but colors don't change | `theme-default.css` not regenerated | Run the CSS extraction steps; check `SOGoUIxDebugEnabled` |
| Colors change in debug mode only | CSS step skipped | Extract and save `theme-default.css`, set `SOGoUIxDebugEnabled = NO` |
| Changes lost after restart | Files not persisted in Docker | Mount as volumes instead of `COPY` in ephemeral containers |
| Logo shows broken image | Wrong format or path | Use SVG; verify mount target path matches `WebServerResources/img/` |
| Custom CSS not loading | Wrong path in `SOGoUIAdditionalCSSFiles` | Use relative path from `WebServerResources/`, e.g. `"css/custom.css"` |
| Memcached serving stale theme | Cache not cleared | Restart memcached after theme changes: `systemctl restart memcached` |
| Favicon unchanged | Browser cache | Hard-refresh or clear browser cache; favicon is aggressively cached |

## References

- [SOGo Installation Guide — Web Interface Configuration](https://www.sogo.nu/files/docs/SOGoInstallationGuide.html)
- [SOGo Developer Guide — Alternate Color Theme](https://github.com/Alinto/sogo/blob/main/Documentation/SOGoDevelopersGuide.asciidoc)
- [AngularJS Material Theming](https://material.angularjs.org/latest/Theming/01_introduction)
- [openDesk Edu — SOGo Helm Chart](https://github.com/opendesk-edu/opendesk-edu/tree/main/helmfile/charts/sogo)
- [openDesk Edu — Theming Documentation](https://github.com/opendesk-edu/opendesk-edu/blob/main/docs/theming.md)
- [mailcow — SOGo Theme Customization](https://docs.mailcow.email/manual-guides/SOGo/u_e-sogo/)
