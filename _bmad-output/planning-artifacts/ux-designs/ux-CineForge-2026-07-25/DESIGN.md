---
name: CineForge
description: Dynamic QR redirect and analytics platform for film directors.
colors:
  surface-base: '#FFFFFF'
  surface-raised: '#F8FAFC'
  surface-base-dark: '#0F172A'
  surface-raised-dark: '#1E293B'
  ink-primary: '#0F172A'
  ink-secondary: '#64748B'
  ink-disabled: '#CBD5E1'
  ink-primary-dark: '#F1F5F9'
  ink-secondary-dark: '#94A3B8'
  ink-disabled-dark: '#475569'
  primary: '#0F172A'
  primary-foreground: '#FFFFFF'
  accent: '#F59E0B'
  accent-foreground: '#1A1A2E'
  border-hairline: '#E2E8F0'
  border-hairline-dark: '#334155'
  destructive: '#EF4444'
  destructive-foreground: '#FFFFFF'
typography:
  body:
    fontFamily: 'Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif'
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.5'
    letterSpacing: 0
  body-sm:
    fontFamily: 'Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif'
    fontSize: 14px
    fontWeight: '400'
    lineHeight: '1.5'
    letterSpacing: 0
  label:
    fontFamily: 'Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif'
    fontSize: 14px
    fontWeight: '500'
    lineHeight: '1.4'
    letterSpacing: 0
  mono:
    fontFamily: 'ui-monospace, "SF Mono", "Fira Code", "JetBrains Mono", Consolas, monospace'
    fontSize: 14px
    fontWeight: '400'
    lineHeight: '1.5'
    letterSpacing: 0
rounded:
  sm: 4px
  md: 6px
  lg: 8px
  full: 9999px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '5': 24px
  '6': 32px
  '7': 48px
  '8': 64px
components:
  button-primary:
    background: '{colors.accent}'
    foreground: '{colors.accent-foreground}'
    radius: '{rounded.md}'
    border: 'none'
  button-secondary:
    background: 'transparent'
    foreground: '{colors.primary}'
    radius: '{rounded.md}'
    border: '1px solid {colors.border-hairline}'
  button-ghost:
    background: 'transparent'
    foreground: '{colors.ink-secondary}'
    radius: '{rounded.md}'
    border: 'none'
  button-destructive:
    background: '{colors.destructive}'
    foreground: '{colors.destructive-foreground}'
    radius: '{rounded.md}'
    border: 'none'
  input-url:
    background: '{colors.surface-raised}'
    foreground: '{colors.ink-primary}'
    radius: '{rounded.md}'
    border: '1px solid {colors.border-hairline}'
  campaign-card:
    background: '{colors.surface-raised}'
    foreground: '{colors.ink-primary}'
    radius: '{rounded.md}'
    border: '1px solid {colors.border-hairline}'
  interstitial:
    background: '{colors.surface-base}'
    foreground: '{colors.ink-primary}'
    radius: '{rounded.md}'
---

## Brand & Style

CineForge is a tool for film directors who need to change what a printed QR code points to — fast. The brand posture is professional, focused, and cinematic-without-gimmick. A director is a creative professional, not a marketer; they value tools that get out of the way and let them ship.

The visual language follows: a deep slate primary that reads like a cinema screen in the dark, a single warm amber accent that means "this is the action" (like a projector beam cutting the dark), and visual restraint everywhere else. No movie-themed decoration — no clapperboard icons, no film strips, no ticket stubs. The name "CineForge" is the only cinematic reference; the interface is a tool, not a movie poster.

Light mode is the default surface; dark mode is a setting. The palette is restrained on purpose — a redirect-and-analytics tool should not compete with the campaign it points to.

## Colors

The palette is deliberately restrained — two brand colors plus neutral surfaces. A director's dashboard should not draw attention to itself.

- **Deep Slate (`#0F172A`)** is the primary color and the dark-mode canvas. Used on the primary button, active nav items, and the dark-mode background. Evokes a cinema screen in the dark — professional, serious, focused. Replaces shadcn's default `primary`.
- **Warm Amber (`#F59E0B` light / `#D4A574` dark)** is the accent. Used exclusively for primary actions — the "Save" button, the "Create campaign" button, the "Export" button. Never used for decoration, never used for state badges, never used for non-primary actions. Amber means "this fires."
- **Surface White (`#FFFFFF` light / `#0F172A` dark)** is the canvas. Light mode is the default; dark mode uses the deep slate as the base.
- **Surface Raised (`#F8FAFC` light / `#1E293B` dark)** is for cards, inputs, and elevated surfaces — distinguished from the canvas by a subtle tone shift.
- **Ink Primary (`#0F172A` light / `#F1F5F9` dark)** is body text and headings.
- **Ink Secondary (`#64748B` light / `#94A3B8` dark)** is secondary text — scan counts, dates, truncated URLs.
- **Hairline (`#E2E8F0` light / `#334155` dark)** separates list items and cards at the lowest possible contrast.
- **Destructive (`#EF4444`)** is for delete confirmation and error states only.

Avoid: chromatic flourishes, gradient surfaces, more than two brand colors, movie-themed icons or imagery, celebratory animations.

## Typography

Inter is the type system — clean, readable, professional. It ships with a system-font fallback stack so the interface renders immediately even before the font loads. No display serif; clarity over decoration. Directors need to read scan counts and URLs, not admire typography.

- **Body** (16px, Regular) — primary text, scan counts, campaign names.
- **Body-sm** (14px, Regular) — secondary text, dates, truncated URLs, form labels.
- **Label** (14px, Medium) — form labels, table headers, section headings.
- **Mono** (14px, Regular) — redirect URLs, campaign IDs, data exports. Uses a platform monospace stack for legibility at small sizes.

Headings use the Label weight at Body size — no oversized display text. The interface is a tool, not a marketing page.

## Layout & Spacing

Tailwind's 4-based scale inherited as-is (4, 8, 12, 16, 24, 32, 48, 64). Maximum content width: `max-w-3xl` (768px) for the dashboard — CineForge is not a wide-table product. A single-column layout keeps the surface focused on the redirect URL, the primary action.

The dashboard uses a top bar (session menu, theme toggle) + main content area. The campaign list is a single-column list of cards. The campaign detail is a single-column form. Analytics uses a 2-column grid on `md+` (chart + summary stats), stacking to single-column on `sm`.

The redirect interstitial (served by the backend, not the SPA) is a centered card on a full-viewport surface — no nav, no chrome, just the message and the geolocation prompt.

## Elevation & Depth

Subtle. Cards and inputs sit on `{colors.surface-raised}`, distinguished from `{colors.surface-base}` by a tone shift. Borders use `{colors.border-hairline}` at the lowest legible contrast. No drop shadows as a hierarchy device — hierarchy comes from layout, typography weight, and the single accent color. The interstitial has no elevation at all — it is a flat surface on a flat background.

## Shapes

`rounded/sm` (4px) for inputs and small interactive elements. `rounded/md` (6px) for cards, buttons, and dialogs. `rounded/lg` (8px) for modals. `rounded/full` (9999px) only for status pills (active / deleted). Nothing sharper than 4px, nothing softer than full — the corners read "tool" not "consumer app."

## Components

shadcn/ui components are used as-is for the 80% of surface vocabulary that doesn't need brand customization (Card, Dialog, Sheet, DropdownMenu, Toast, Tabs, Avatar, Separator, Table). The contract: don't customize these beyond the tokens below.

Brand-layer-overridden components:

- **Button (primary variant)** — `{colors.accent}` fill, `{colors.accent-foreground}` text, `{rounded.md}` corner. Used for: Save redirect URL, Create campaign, Export data, Request location (interstitial). Other variants (secondary, ghost, destructive) inherit shadcn defaults.
- **Button (destructive variant)** — `{colors.destructive}` fill, `{colors.destructive-foreground}` text. Used only for: Delete campaign (with confirmation).
- **Input (URL field)** — `{colors.surface-raised}` background, `{colors.ink-primary}` text, `{colors.border-hairline}` border, `{rounded.md}` corner. Auto-validates HTTP/HTTPS URLs. Shows `{colors.destructive}` border + error text on invalid input.
- **Campaign card** — `{colors.surface-raised}` background, `{colors.border-hairline}` border, `{rounded.md}` corner. Contains: campaign name, truncated redirect URL (mono), scan count, last scan date, QR code thumbnail. Hover reveals an "Edit" quick-action.
- **Interstitial (redirect page)** — Flat surface, `{colors.surface-base}` background, `{colors.ink-primary}` text. Centered card with a brief message and the geolocation permission prompt. No nav, no chrome, no framework. Served by .NET backend as a Razor view.
- **Status pill** — `rounded/full`, `{colors.surface-raised}` background, `{colors.ink-secondary}` text. Used for: campaign status (active / deleted). Never uses accent color.

## Do's and Don'ts

| Do | Don't |
|---|---|
| Inherit shadcn defaults for everything not in the brand layer | Override shadcn's color tokens beyond `primary`, `accent`, and `destructive` |
| Use `{colors.accent}` only for primary actions (Save, Create, Export) | Use accent for secondary actions, state badges, or decoration |
| Use `{colors.destructive}` only for delete confirmation and error states | Use destructive for non-destructive actions |
| Keep the dashboard to a single column inside `max-w-3xl` | Wide multi-column tables or dashboards |
| Mono font for redirect URLs and campaign IDs | Decorative fonts for URLs or IDs |
| Inter font for all body and label text | Display serifs or decorative headings |
| Flat surfaces, no drop shadows for hierarchy | Shadows as a visual hierarchy device |
| Minimal interstitial — just the message and geolocation prompt | Chrome, nav, or framework on the interstitial page |
