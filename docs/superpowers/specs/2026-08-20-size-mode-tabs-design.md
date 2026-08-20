# Size Mode Tabs (Ready To Wear / Bespoke) — Design

**Date:** 2026-08-20  
**Theme:** `traack-global-shopify` (Minimog OS 2.0)  
**Status:** Pending user review of this written spec

## Problem

On the product page, size UI currently shows:

- Label like `Shoe size: 38` with the size grid always visible
- A separate **Size: Bespoke** custom-liquid block (panel open by default, disables size options)

We need a single **Size:** control with two mutually exclusive modes.

## Goals

1. Label prefix stays **`Size:`**
2. Two tabs/buttons: **Ready To Wear** and **Bespoke**
3. Both mode contents are **hidden by default**
4. Clicking a tab shows that mode’s content and **closes the other**
5. Preserve existing bespoke cart properties / validation behavior

## Non-goals

- Changing Fitting Guide popup
- Renaming Shopify product option values in Admin
- Porting this to `shopify-a0kq2u-xk` in this pass

## UX

```
Size:   [ Ready To Wear ]   [ Bespoke ]
```

| State | Ready To Wear content | Bespoke content |
|-------|------------------------|-----------------|
| Default (neither) | Hidden (size swatch/button grid) | Hidden (form + info box) |
| Ready To Wear active | Visible size grid | Hidden; clear bespoke fields / Size Type |
| Bespoke active | Hidden size grid | Visible form + dashed info box; set Size Type |

Re-clicking the active tab: keep open until the other tab is chosen (no collapse-to-empty on second click), unless we later decide otherwise.

Visual style: match current dark theme (white border buttons, same type scale as existing bespoke toggle).

## Behavior details

### Ready To Wear

- Shows the theme’s existing size option content (`.m-product-option--content` for the Size option).
- Normal variant selection / Add to cart.
- Clears bespoke `properties[Size Type]`, custom size, and contact when switching to RTW.

### Bespoke

- Shows existing bespoke fields:
  - Custom size / measurement textarea
  - Email / WhatsApp input
  - Hidden `properties[Size Type]` = `Fully Bespoke` when active
  - Hint + dashed wooden-last info copy (current copy)
- Size grid stays hidden (not merely disabled).
- On submit while Bespoke active: require size + contact (same as today).

## Technical approach

**Recommended:** new snippet + thin integration; remove oversized bespoke custom liquid from `templates/product.json`.

### Files

| File | Change |
|------|--------|
| `snippets/size-mode-tabs.liquid` | **Create** — tab UI, CSS, JS (accordion + form property handling) |
| `snippets/product-option.liquid` | For size options: render tabs after label; mark content panel for show/hide |
| `templates/product.json` | Remove `custom_liquid_nRitj4` (Fully Bespoke Size Toggle v3) block from `block_order` / `blocks` |
| `snippets/cart-drawer-item.liquid` / `cart-line-item.liquid` | Optional follow-up: hide Size line when `Size Type == Fully Bespoke` (as in a0kq2u). Not required for first ship if cart already acceptable. |

### Integration sketch

1. In `product-option.liquid`, when `is_size`:
   - Change visible label title to fixed **`Size`** for the tab row (or keep option label as Size via theme setting).
   - Do **not** show selected value in the main tab header (`Shoe size: 38` → no dynamic `: 38` next to Size).
   - Render `{% render 'size-mode-tabs', section: section, product_form_id: product_form_id %}`.
   - Wrap or target `.m-product-option--content` with a data attribute / class so JS can toggle `display`.

2. Snippet JS:
   - Find size option field for this section.
   - Default: hide RTW content + hide bespoke panel; neither tab `is-active` (or both inactive styling).
   - RTW click → show size content, hide bespoke, clear bespoke required/props.
   - Bespoke click → hide size content, show bespoke panel, set Size Type, set required on fields.
   - Ensure property inputs live inside `#product-form-{{ section.id }}` (move wrap into form if needed, as current custom liquid does).

### Label wording

- Static chrome: **`Size:`**
- Tab 1 label: **`Ready To Wear`**
- Tab 2 label: **`Bespoke`**

Selected size value may still update inside the RTW panel only (optional); it must not appear as `Size: 38` in the header row.

## Testing

1. Product page load: only `Size: Ready To Wear | Bespoke`; no size buttons; no bespoke form.
2. Click Ready To Wear: size grid appears; bespoke hidden.
3. Select a size + Add to cart: normal variant cart line.
4. Click Bespoke: size grid hides; bespoke form + info visible; RTW inactive.
5. Add to cart without fields: blocked with alert.
6. Add to cart with fields: line includes bespoke properties.
7. Switch RTW → Bespoke → RTW: exclusive panels; no dual-open state.
8. Fitting Guide still works.

## Open decisions (locked for this spec)

- Mutual exclusivity: **yes**
- Default: **both closed**
- Second click on active tab: **stay open** (switch only via other tab)
- Scope: **traack-global-shopify only**
