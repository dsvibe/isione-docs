# Style Guide

Design decisions that apply across all IsiOne components (web, Android, iOS, hardware).

## Colours

### Status Colours

Task status is communicated through colour across all platforms (web, mobile, hardware LEDs).

| Status | Colour | Hex | Usage |
|--------|--------|-----|-------|
| OK | <span style="display:inline-block;width:14px;height:14px;background:#4ade80;border-radius:50%;vertical-align:middle"></span> | `#4ade80` | Task completed within schedule |
| Pending | <span style="display:inline-block;width:14px;height:14px;background:#a78bfa;border-radius:50%;vertical-align:middle"></span> | `#a78bfa` | Task due but not yet overdue |
| Overdue | <span style="display:inline-block;width:14px;height:14px;background:#f87171;border-radius:50%;vertical-align:middle"></span> | `#f87171` | Task past its due time |

### Accent

| Name | Colour | Hex | Usage |
|------|--------|-----|-------|
| Primary | <span style="display:inline-block;width:14px;height:14px;background:#646cff;border-radius:50%;vertical-align:middle"></span> | `#646cff` | Links, active states, primary buttons |
| Primary hover | <span style="display:inline-block;width:14px;height:14px;background:#535bf2;border-radius:50%;vertical-align:middle"></span> | `#535bf2` | Hover state for primary elements |

### Feedback

| Name | Colour | Hex | Usage |
|------|--------|-----|-------|
| Success | <span style="display:inline-block;width:14px;height:14px;background:#4ade80;border-radius:50%;vertical-align:middle"></span> | `#4ade80` | Confirmation, complete actions |
| Warning | <span style="display:inline-block;width:14px;height:14px;background:#fbbf24;border-radius:50%;vertical-align:middle"></span> | `#fbbf24` | Warnings, caution states |
| Error | <span style="display:inline-block;width:14px;height:14px;background:#f87171;border-radius:50%;vertical-align:middle"></span> | `#f87171` | Errors, destructive actions |

### Neutrals (Dark Theme)

| Name | Colour | Hex | Usage |
|------|--------|-----|-------|
| Background | <span style="display:inline-block;width:14px;height:14px;background:#1a1a1a;border-radius:50%;vertical-align:middle;border:1px solid #444"></span> | `#1a1a1a` | Page/card backgrounds |
| Surface | <span style="display:inline-block;width:14px;height:14px;background:#2a2a2a;border-radius:50%;vertical-align:middle;border:1px solid #444"></span> | `#2a2a2a` | Elevated elements (menus, dropdowns) |
| Border | <span style="display:inline-block;width:14px;height:14px;background:#333333;border-radius:50%;vertical-align:middle;border:1px solid #444"></span> | `#333333` | Dividers, card borders |
| Text primary | <span style="display:inline-block;width:14px;height:14px;background:#ffffff;border-radius:50%;vertical-align:middle"></span> | `#ffffff` | Main text |
| Text secondary | <span style="display:inline-block;width:14px;height:14px;background:#aaaaaa;border-radius:50%;vertical-align:middle"></span> | `#aaaaaa` | Descriptions, secondary info |
| Text muted | <span style="display:inline-block;width:14px;height:14px;background:#888888;border-radius:50%;vertical-align:middle"></span> | `#888888` | Tertiary text, placeholders |

## Task Identity

These elements identify tasks to users and must be consistent across all platforms.

### Icons

All task icons use [Lucide](https://lucide.dev/), an open-source icon library.

### Task Label Font

The font used for task names/headings across all platforms.

**[Nunito](https://fonts.google.com/specimen/Nunito)** - A sans-serif with rounded terminals, friendly and legible at small sizes.

- Use **Medium (500)** or **Semi-Bold (600)** weight
- Semi-Bold recommended for 3D printed hardware labels

## Platform-Specific Styles

Fonts for body text, UI elements, and other non-task content are defined per-platform in their respective documentation.
