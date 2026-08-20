# Size Mode Tabs Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace always-visible shoe size UI + separate bespoke custom liquid with a `Size:` accordion: Ready To Wear | Bespoke (both closed by default, mutually exclusive).

**Architecture:** New `snippets/size-mode-tabs.liquid` owns tab chrome, bespoke panel, CSS, and JS. `product-option.liquid` marks size option content for show/hide and renders the snippet. Remove the old bespoke block from `templates/product.json`.

**Tech Stack:** Shopify Liquid, vanilla JS, Minimog variant-picker markup

## Global Constraints

- Theme: `traack-global-shopify` only
- Header copy: `Size:` + tabs `Ready To Wear` and `Bespoke`
- Default: both panels closed
- Mutual exclusivity: RTW open closes Bespoke and vice versa
- Active tab stays open until the other tab is clicked
- Bespoke cart props: `properties[Custom Size / Measurement]`, `properties[Contact (Email / WhatsApp)]`, `properties[Size Type]=Fully Bespoke`
- No commit unless user asks

---

### Task 1: Create `size-mode-tabs.liquid`

**Files:**
- Create: `snippets/size-mode-tabs.liquid`

**Interfaces:**
- Consumes: `section`, `product_form_id` (optional; defaults to `product-form-{{ section.id }}`)
- Produces: `#size-mode-tabs-{{ section.id }}` with tabs + bespoke panel; JS toggles `[data-size-mode-content]` inside the size option for this section

- [ ] **Step 1: Create snippet** with markup, styles, and script (full file in implementation)

- [ ] **Step 2: Manual check** — file exists under `snippets/`

---

### Task 2: Wire `product-option.liquid`

**Files:**
- Modify: `snippets/product-option.liquid`

**Interfaces:**
- When `is_size`: force label title `Size`, hide `.option-label--selected`, add `data-size-mode-content` + default `display:none` on content, render `size-mode-tabs` once after label (before content)

- [ ] **Step 1: Update all picker-type branches** (`dropdown`, `color`, `image`, `button`, `else`) consistently for `is_size`

- [ ] **Step 2: Verify** only size options get tabs; non-size options unchanged

---

### Task 3: Remove old bespoke custom liquid from `product.json`

**Files:**
- Modify: `templates/product.json`

- [ ] **Step 1: Remove** block `custom_liquid_nRitj4` from `sections.main.blocks` and from `block_order`

- [ ] **Step 2: Keep** Fitting Guide block `custom_liquid_4R8rW8`

- [ ] **Step 3: Validate** JSON parses

---

### Task 4: Verify against spec checklist

- [ ] Load product page: `Size: Ready To Wear | Bespoke`; no size grid; no bespoke form
- [ ] RTW → size grid; Bespoke closed
- [ ] Bespoke → form + info; size grid hidden
- [ ] Switch tabs → exclusive
- [ ] Bespoke ATC without fields → alert
- [ ] Fitting Guide still works
