---
name: next-expo-tauri-feature-first-clean-architecture
description: Production-grade Universal Feature-First Clean Architecture for Next.js (Web), Expo (Mobile), and Tauri (Desktop) Monorepos with shared TypeScript Core and Database packages. Inspired by Flutter Clean Architecture and DDD principles.
---

# Next.js + Expo + Tauri Feature-First Clean Architecture

This skill provides comprehensive architectural guidelines, patterns, and best practices for building scalable, 100% DRY, and maintainable cross-platform monorepos targeting **Web (Next.js)**, **Mobile (Expo React Native)**, and **Desktop (Tauri)**.

---

## 🏛️ 1. High-Level Monorepo Structure

```text
root/
├── packages/
│   ├── core/                        # Pure TypeScript: Domain Models, Business Rules, Helpers, i18n
│   │   ├── src/
│   │   │   ├── types.ts             # Entities, Enums, Interfaces
│   │   │   ├── calculations.ts      # Pure business formulas (Cart, Discounts, Shipping)
│   │   │   ├── formatters.ts        # Currency, Date, Units
│   │   │   ├── constants.ts         # Business constants & thresholds
│   │   │   ├── permissions.ts       # RBAC & Role Permission Guards
│   │   │   ├── theme.ts             # Design System Tokens (colors, spacing, typography, elevation, z-index, animation)
│   │   │   └── i18n/                # Centralized translation dictionaries
│   │   └── package.json
│   │
│   ├── database/ (or supabase/)     # Shared Database Schema, Client Factory, Migrations, RLS
│   └── ui/                          # (Optional) Primitive cross-platform design tokens
│
├── apps/
│   ├── web/                         # Next.js App Router (Thin Controllers + Feature Components)
│   ├── mobile/                      # Expo / React Native (Flutter-style Clean Architecture)
│   └── desktop/                     # Tauri (Next.js / Vite frontend with Rust OS bridge)
```

---

## 🧩 2. Flutter Clean Architecture Mapping to TypeScript / React

For developers coming from Flutter Feature-First Clean Architecture:

| Flutter Clean Architecture Layer | TypeScript / React Universal Equivalent | Location |
| :--- | :--- | :--- |
| **Entities / Data Models** | Pure TypeScript Interfaces & Types | `packages/core/src/types.ts` |
| **Use Cases / Domain Logic** | Pure Functions (No React/UI hooks) | `packages/core/` or `features/{name}/domain/` |
| **Repositories / Data Sources** | API Clients, Supabase Singleton, Query Hooks | `packages/database/` or `features/{name}/data/` |
| **BLoC / Cubit / State** | Zustand Store with `persist` middleware | `features/{name}/store/use{Name}Store.ts` |
| **Pages / Screens** | Thin Route Controllers / Screens | `app/{route}/page.tsx` or `features/{name}/presentation/` |
| **Widgets / Sub-components** | Isolated Small UI Components (< 150 lines) | `features/{name}/components/` or `presentation/components/` |

---

## 💻 3. Web Architecture (Next.js App Router)

### Rule 1: Thin Route Controllers (< 30 Lines)
Never place business logic, large forms, or complex UI trees directly inside `app/*/page.tsx`.
`page.tsx` must only act as a route entry point wrapping the Feature View inside Layout (Navbar/Footer).

```tsx
// apps/web/app/cart/page.tsx (CORRECT: Thin Controller)
"use client";

import React from "react";
import { Navbar } from "../../src/shared/components/Navbar";
import { Footer } from "../../src/shared/components/Footer";
import { CartView } from "../../src/features/cart/components/CartView";

export default function CartPage() {
  return (
    <div className="page-shell">
      <Navbar />
      <main className="main-content">
        <CartView />
      </main>
      <Footer />
    </div>
  );
}
```

### Rule 2: Modular Feature Components
Break feature interfaces into focused, single-responsibility components:
```text
apps/web/src/features/cart/
├── store/
│   └── useCartStore.ts
└── components/
    ├── CartItemRow.tsx              # Single row render & quantity buttons
    ├── CartShippingForm.tsx         # Delivery method & map GPS picker
    ├── CartPaymentOptions.tsx       # Payment method selection
    ├── CartOrderSummary.tsx         # Totals, discounts, checkout CTA
    ├── CartSuccessView.tsx          # Order receipt & WhatsApp share
    └── CartView.tsx                 # Main orchestrator container
```

---

## 📱 4. Mobile Architecture (Expo / React Native)

```text
apps/mobile/src/
├── core/                            # Shared Platform Layer
│   ├── lib/supabase.ts              # Supabase singleton & persistent storage
│   ├── storage/storage.ts           # Cross-platform AsyncStorage adapter
│   ├── store/useAppStore.ts         # Global preferences (Theme, Language)
│   └── components/                  # Shared Layout (Navbar, BottomNav, HeroBanner, FilterBar)
│
└── features/                        # Domain Features
    ├── auth/
    │   ├── store/useAuthStore.ts
    │   └── presentation/AccountView.tsx
    ├── cart/
    │   ├── domain/cartCalculations.ts
    │   ├── store/useCartStore.ts
    │   └── presentation/
    │       ├── components/
    │       │   ├── CartItemRow.tsx
    │       │   ├── ShippingSelector.tsx
    │       │   ├── PaymentMethodPicker.tsx
    │       │   └── CartPriceSummary.tsx
    │       └── CartModal.tsx
    ├── pos/
    │   ├── domain/posUtils.ts       # Receipt formatting & ESC/POS text
    │   ├── store/usePOSStore.ts
    │   └── presentation/
    │       ├── components/
    │       │   ├── POSCartList.tsx
    │       │   ├── POSPaymentModal.tsx
    │       │   └── POSReceiptModal.tsx
    │       └── POSView.tsx
    └── orders/
        ├── store/useOrderStore.ts
        └── presentation/OrdersView.tsx
```

---

## 🖥️ 5. Desktop Architecture (Tauri)

Tauri leverages the same web frontend components while providing native desktop OS integrations (hardware thermal printing, local file access, offline SQLite, auto-updates):

```text
apps/desktop/
├── src-tauri/                       # Rust Backend Engine
│   ├── src/
│   │   ├── main.rs                  # OS Window & IPC Handlers
│   │   └── printer.rs               # Native Direct ESC/POS USB/Serial Printing
│   ├── Cargo.toml
│   └── tauri.conf.json
│
└── src/ (or linked to Next.js SSG / Vite)
    └── features/                    # Reuses identical feature components from Web & Core
```

---

## 🔄 6. The Golden DRY Rules

1. **Pure Business Formulas Live in `@monorepo/core`**:
   Never calculate discounts, tax, shipping thresholds, or prices differently in Web, Mobile, or Desktop.
2. **State Decoupling**:
   Keep Zustand stores feature-scoped (`useCartStore`, `useAuthStore`, `usePOSStore`, `useOrderStore`). Avoid giant monolithic global stores.
3. **Pure Re-export Bridges**:
   When refactoring legacy files, replace old paths with clean 1-line re-exports:
   ```ts
   // apps/mobile/src/components/CartModal.tsx
   export * from "../features/cart/presentation/CartModal";
   ```
4. **Zero-any Strict TypeScript**:
   Enforce complete types across all boundary layers, props, and store states.

---

## 🔍 7. Verification Checklist

Before considering any task complete:
- [ ] Run `pnpm check-types` across all monorepo packages (Exit Code 0).
- [ ] Verify `app/*/page.tsx` files are thin controllers (< 35 lines).
- [ ] Verify no business logic is duplicated between Web, Mobile, and Desktop.
- [ ] Confirm git status and commit safety rules.

---

## 🎨 8. Design System & Theme Tokens (`packages/core/src/theme.ts`)

All design tokens live in a **single, platform-agnostic** `theme.ts` inside `packages/core`.
Both Web (Next.js) and Mobile (Expo) consume this file directly — no duplication of values.

### Token Map

| Token Group | Export | Web Usage | Mobile (RN) Usage |
| :--- | :--- | :--- | :--- |
| **Colors (Light)** | `THEME_LIGHT_COLORS` | CSS custom properties / inline styles | `StyleSheet.create` values |
| **Colors (Dark)** | `THEME_DARK_COLORS` | `data-theme="dark"` overrides | Dark mode `StyleSheet` |
| **Color getter** | `getThemeColors(mode)` | Call with `"light"` or `"dark"` | Same |
| **Spacing (CSS)** | `THEME.spacing` | `gap: THEME.spacing.md` | — |
| **Spacing (num)** | `THEME.spacingNum` | — | `padding: THEME.spacingNum.base` |
| **Border Radius (CSS)** | `THEME.radius` | `borderRadius: THEME.radius.lg` | — |
| **Border Radius (num)** | `THEME.radiusNum` | — | `borderRadius: THEME.radiusNum.lg` |
| **Font Size (num)** | `THEME.fontSizeNum` | `fontSize: THEME.fontSizeNum.md` | Same |
| **Font Weight (CSS)** | `THEME.fontWeight` | `fontWeight: THEME.fontWeight.bold` | — |
| **Font Weight (num)** | `THEME.fontWeightNum` | — | `fontWeight: THEME.fontWeightNum.bold` |
| **Line Height (CSS)** | `THEME.lineHeight` | `lineHeight: THEME.lineHeight.normal` | — |
| **Line Height (num)** | `THEME.lineHeightNum` | — | Multiply with `fontSize` |
| **Font Families** | `THEME.fontFamily` | `.sans`, `.mono` via CSS variable | `.regular` / `.bold` (expo-font names) |
| **Elevation (RN)** | `THEME.elevation` | — | Spread: `...THEME.elevation.md` |
| **Shadows (CSS)** | `THEME.shadows` / `getThemeColors().shadows` | `boxShadow: THEME.shadows.lg` | — |
| **Z-Index** | `THEME.zIndex` | `zIndex: THEME.zIndex.modal` | `zIndex` in `StyleSheet` |
| **Animation** | `THEME.animation` | `transition: ${THEME.animation.duration.normal} ${THEME.animation.easing.spring}` | `Animated.timing({ duration: THEME.animation.durationMs.normal })` |
| **Icon Size (num)** | `THEME.iconSizeNum` | `size={THEME.iconSizeNum.xl}` | Same |
| **Breakpoints** | `THEME.breakpoints` | `@media (min-width: ${THEME.breakpoints.tablet}px)` | `useWindowDimensions()` comparisons |

---

### Color Usage

```ts
// packages/core/src/theme.ts
import { getThemeColors, THEME } from "@yourmonorepo/core";

// Get correct palette for current mode
const colors = getThemeColors(isDark ? "dark" : "light");

// Examples
colors.bgMain            // page background
colors.toserba.primary   // main brand green
colors.thrift.primary    // thrift brand indigo
colors.accent.danger     // error red
colors.shadows.lg        // CSS box-shadow string (mode-correct)
```

---

### Typography Scale

```text
fontSizeNum:  micro(9) xs(11) sm(12) base(14) md(16) lg(18) xl(22) xxl(26) xxxl(32) display(40) hero(52)
fontWeightNum: regular(400) medium(500) semiBold(600) bold(700) extraBold(800) black(900)
lineHeightNum: tight(1.15) snug(1.3) normal(1.5) relaxed(1.65) loose(1.8)
```

---

### Elevation — React Native Only

```ts
// Spread directly onto a View style:
import { THEME } from "@yourmonorepo/core";

<View style={[styles.card, THEME.elevation.md]} />

// Available levels: none | xs | sm | md | lg | xl | modal
```

---

### Z-Index Layering

```text
base(0) → raised(10) → dropdown(100) → sticky(200) → overlay(300)
→ modal(400) → popover(500) → toast(600) → tooltip(700) → max(9999)
```

```ts
// Web
style={{ zIndex: THEME.zIndex.modal }}

// React Native
StyleSheet.create({ container: { zIndex: THEME.zIndex.dropdown } })
```

---

### Animation / Motion

```ts
// CSS transition (Web / Tauri)
const transition = `all ${THEME.animation.duration.normal} ${THEME.animation.easing.spring}`;

// React Native Animated
Animated.timing(animValue, {
  toValue: 1,
  duration: THEME.animation.durationMs.normal,
  useNativeDriver: true,
}).start();

// Presets: micro | standard | enter | exit
const { duration, easing } = THEME.animation.preset.enter;
```

---

### Exported TypeScript Types

```ts
import type {
  ThemeColors,        // typeof THEME_LIGHT_COLORS
  ThemeTokens,        // typeof THEME (full token object)
  ElevationLevel,     // "none" | "xs" | "sm" | "md" | "lg" | "xl" | "modal"
  ZIndexLevel,        // "base" | "raised" | "dropdown" | ... | "max"
  AnimationDuration,  // "instant" | "fast" | "normal" | "slow" | "xSlow"
} from "@yourmonorepo/core";
```
