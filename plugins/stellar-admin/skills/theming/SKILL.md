---
name: theming
description: >-
  Configures the look of a StellarAdmin app — picking a theme stylesheet, customizing theme
  values with CSS variables, enabling dark mode, using the design tokens in your own
  markup, and tuning the menu color / appearance / accent. Use when the user wants to
  change the StellarAdmin theme, add or toggle dark mode, adjust menu or dropdown
  appearance, or asks which colors or classes to use with StellarAdmin.
metadata:
  author: StellarAdmin
---

# Theming StellarAdmin

**A theme is a stylesheet.** The package ships one self-contained CSS bundle per theme; the layout links exactly one, and switching themes means switching that `<link>`. Nothing about the theme is configured in C#. Dark mode is a CSS class, and day-to-day styling means using the semantic design tokens instead of hard-coded colors.

## Pick a theme (the layout `<link>`)

StellarAdmin ships ten themes: `concourse`, `ledger`, `vega`, `nova`, `luma`, `lyra`, `maia`, `mira`, `rhea`, `sera`. Concourse and Ledger are independently designed; the other eight derive from shadcn/ui.

The eight shadcn-derived themes share the same base palette and radius; their component geometry, density, and token usage differ. Ledger supplies its own warm light palette, charcoal dark palette, typography, and raised button treatment. Select a theme for its visual design; override semantic variables when the app needs different brand colours.

```razor
<link rel="stylesheet" href="/_content/StellarAdmin.TagHelpers/stellar-admin.nova.css" asp-append-version="true"/>
```

To change the theme, change `nova` to another theme name — that's the whole operation. You **must** link exactly one; without it, components render unstyled.

Preview all ten themes with the StellarAdmin documentation demo picker. The [shadcn/ui Create page](https://ui.shadcn.com/create) covers the eight upstream-derived styles; Concourse and Ledger are specific to StellarAdmin.

### Concourse

Use `stellar-admin.concourse.css` as the single theme bundle. Load Source Sans 3 (UI, weights 400/500/600/700) and IBM Plex Mono (identifiers and values, weights 400/500) from the app layout, self-hosted or through a font provider. The stylesheet does not fetch fonts. Existing tag helpers and the `.dark` class work unchanged.

Concourse has cool grey surfaces, a blue accent, 4px corners, and 34px default controls. Actions use colour changes without press movement; neutral hover feedback stays distinct from persistent selection. Use existing components rather than the handoff's `.cc-*` classes. Concourse-specific `--sa-concourse-*` variables are implementation details. When overriding the primary colour, coordinate its `--sa-concourse-primary-hover`, `--sa-concourse-primary-pressed`, and `--sa-concourse-primary-border` companions in both modes; destructive actions have corresponding hover, border, and foreground companions. Touch target sizing belongs to the app's responsive composition.

### Ledger

Use `stellar-admin.ledger.css` as the single theme bundle. Load Lexend (UI, weights 300–700) and JetBrains Mono (identifiers and shortcuts, weights 400–500) from the app's layout, self-hosted or through a font provider. The library stylesheet does not fetch fonts. Existing tag helpers and the `.dark` class work unchanged, including shared surfaces used by Pro.

Preserve raised borders and shadows on primary, secondary, outline, and destructive buttons; ghost and link actions are flat. Do not reproduce the original prototype's `.ldg-*` classes. Use StellarAdmin's normal components. Ledger-specific `--sa-ledger-*` variables are implementation details rather than shared tokens. If customising primary/destructive colours, coordinate their `--sa-ledger-primary-hover`, `--sa-ledger-primary-border`, `--sa-ledger-destructive-hover`, `--sa-ledger-destructive-border`, and `--sa-ledger-destructive-foreground` companions in both modes.

## Dark mode

Dark mode is a **class-based variant** — every theme bundle already carries both the light (`:root`) and dark (`.dark`) token values, so there's no extra CSS and no configuration method.

Enable it by putting the **`dark` class on a containing element** (usually `<html>`); every descendant then reads the dark values:

```razor
<html lang="en" class="dark">
```

How the class gets there is up to the app — server-rendered from a saved preference, or client-side. A minimal client script that honors a saved choice and falls back to the OS setting, placed in `<head>` **before** the theme stylesheet:

```html
<script>
    (function () {
        var saved = null;
        try { saved = localStorage.getItem("theme"); } catch (e) { }
        var prefersDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
        var dark = saved === "dark" || (saved !== "light" && prefersDark);
        document.documentElement.classList.toggle("dark", dark);
        document.documentElement.style.colorScheme = dark ? "dark" : "light";
    })();
</script>
```

Setting `color-scheme` alongside the class keeps native controls — scrollbars, date pickers, form elements — in step with the page.

## Customize theme values

Every color and radius is a CSS custom property; the compiled rules all reference `var(--…)`. Override by redeclaring the properties in the app's own stylesheet, **after** the theme `<link>` — no build tooling required:

```html
<link rel="stylesheet" href="/_content/StellarAdmin.TagHelpers/stellar-admin.nova.css" asp-append-version="true"/>
<style>
  :root {
    --primary: oklch(0.6725 0.1362 42);
    --radius: 0.5rem;
    --font-sans: "JetBrains Mono", ui-monospace, monospace;
  }
  .dark {
    --primary: oklch(0.705 0.13 42);
  }
</style>
```

The variables you can override:

| Variable | What it colors |
|----------|----------------|
| `--background` / `--foreground` | The base surface and text color — page shell, sections, body text. |
| `--card` / `--card-foreground` | Raised surfaces: Card, dashboard panels, settings panels. |
| `--popover` / `--popover-foreground` | Overlay surfaces: Popover, DropdownMenu and similar floating components. |
| `--primary` / `--primary-foreground` | The brand color — default Button, selected states, active accents. |
| `--secondary` / `--secondary-foreground` | Filled but quieter elements: secondary buttons and badges. |
| `--muted` / `--muted-foreground` | Recessive surfaces and text: descriptions, placeholders, helper text, empty states. |
| `--accent` / `--accent-foreground` | Interaction highlights: ghost buttons, hovered rows, highlighted menu entries. |
| `--destructive` | Danger and error: destructive buttons, invalid form states, destructive menu items. |
| `--border` | Borders and separators between cards, menus, tables, layout. |
| `--input` | Borders and surfaces of form controls. |
| `--ring` | The focus ring on any focusable control. |
| `--chart-1` ... `--chart-5` | The default series palette for charts. |
| `--sidebar` / `--sidebar-foreground` | The Sidebar surface and its default text. |
| `--sidebar-primary` / `--sidebar-primary-foreground` | The sidebar's most prominent elements: active items, icon tiles, badges. |
| `--sidebar-accent` / `--sidebar-accent-foreground` | Hovered and selected sidebar entries. |
| `--sidebar-border` / `--sidebar-ring` | Borders and focus rings inside the sidebar. |
| `--radius` | The base corner radius the `radius-*` variables derive from. |
| `--font-sans` / `--font-mono` | The type families. |

Values live on `:root` with dark-mode overrides under `.dark` — override both when a color needs to differ between modes.

## Use the design tokens in your own markup

The theme bundle styles the `<sa-*>` components. It does **not** make token utilities available to your own markup: writing `class="bg-primary"` on your own `<div>` does nothing, because that utility only exists inside the prebuilt bundle.

If the app runs its own Tailwind v4 build, copy [`theme-tokens.css`](https://github.com/stellar-admin/stellar-admin/blob/master/src/StellarAdmin.TagHelpers/Client/css/theme-tokens.css) (`src/StellarAdmin.TagHelpers/Client/css/theme-tokens.css`) into the project and import it from the Tailwind entry stylesheet:

```css
@import "tailwindcss";
@import "./theme-tokens.css";
```

Now `bg-primary`, `text-muted-foreground`, `bg-card`, `rounded-lg`, `dark:*` and the rest work in your own markup, and adapt to theme changes and customizations automatically. The file carries only the token *vocabulary* — the values still come from the linked theme bundle at runtime, so keep the `<link>` in place.

| Use | Tokens |
|-----|--------|
| Primary action | `bg-primary` / `text-primary-foreground` |
| Surfaces | `bg-card` / `text-card-foreground`, `bg-popover` / `text-popover-foreground` |
| Muted / secondary | `text-muted-foreground`, `bg-secondary`, `bg-muted` |
| Accent (hover/active) | `bg-accent` / `text-accent-foreground` |
| Danger | `text-destructive` |
| Borders / inputs / focus | `border`, `bg-input`, `ring-ring` |

Also available: the `sidebar-*` and `chart-*` token families and the `--radius` variable.

```razor
<!-- good: adapts to theme + dark mode -->
<div class="rounded-lg border bg-card text-card-foreground p-4">...</div>

<!-- avoid: hard-coded palette colors don't follow the theme -->
<div class="rounded-lg border border-gray-200 bg-white text-gray-900 p-4">...</div>
```

## Menu surfaces (`Program.cs`)

Floating menu surfaces — Dropdown Menu content and sub-menus — have three app-wide appearance settings, configured once by chaining `ConfigureMenu` off `AddTagHelpers()`:

```csharp
using StellarAdmin;
using StellarAdmin.TagHelpers;

builder.Services.AddStellarAdmin()
    .AddTagHelpers()
    .ConfigureMenu(menu =>
    {
        menu.Color = MenuColor.Inverted;              // Default | Inverted
        menu.Appearance = MenuAppearance.Translucent; // Solid | Translucent
        menu.Accent = MenuAccent.Bold;                // Subtle | Bold
    });
```

| Setting | Values | Meaning |
|---------|--------|---------|
| `Color` | `Default`, `Inverted` | `Default` renders the menu in the page's current scheme; `Inverted` renders it in the dark scheme regardless of the page. |
| `Appearance` | `Solid`, `Translucent` | `Solid` is an opaque popover surface; `Translucent` is frosted, with a backdrop blur. |
| `Accent` | `Subtle`, `Bold` | The highlight on the focused or hovered item — `Subtle` uses the muted accent color, `Bold` a solid primary highlight. |

Defaults are `Color=Default`, `Appearance=Solid`, `Accent=Subtle` — only call `ConfigureMenu` to change them.

## Rules

1. Pick the theme by linking one `stellar-admin.<theme>.css` in the layout; switch themes by switching the `<link>`. No C# theme configuration exists.
2. Customize theme values by redeclaring the CSS custom properties (`--primary`, `--radius`, ...) in the app's own stylesheet, after the theme link. Override `.dark` too where the value should differ.
3. Enable dark mode with the `dark` class on an ancestor; don't write your own dark CSS — the tokens are already themed for both modes. Set `color-scheme` alongside it.
4. To use token utilities in your own markup, the app needs its own Tailwind build plus `theme-tokens.css`. Without that, `class="bg-primary"` silently does nothing.
5. Prefer semantic tokens (`bg-primary`, `text-muted-foreground`, `bg-card`, `border`, `text-destructive`) over hard-coded colors.
6. Configure menu options once via `ConfigureMenu`, chained off `.AddTagHelpers()`.
7. For one-off tweaks, override via the `class` attribute rather than editing the shipped bundles.
