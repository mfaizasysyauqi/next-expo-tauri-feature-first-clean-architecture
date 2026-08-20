# Next.js + Expo + Tauri Feature-First Clean Architecture

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Next.js](https://img.shields.io/badge/Next.js-15-black?logo=next.js)](https://nextjs.org/)
[![Expo](https://img.shields.io/badge/Expo-React%20Native-000020?logo=expo)](https://expo.dev/)
[![Tauri](https://img.shields.io/badge/Tauri-Desktop-FFC131?logo=tauri)](https://tauri.app/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript)](https://www.typescriptlang.org/)

Production-grade **Universal Feature-First Clean Architecture** for monorepos targeting **Web (Next.js)**, **Mobile (Expo React Native)**, and **Desktop (Tauri)** with a shared TypeScript Core and Database layer. Inspired by Flutter Clean Architecture and Domain-Driven Design (DDD) principles.

---

## 🏛️ High-Level Monorepo Structure

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

## 🧩 Flutter Clean Architecture Mapping to TypeScript / React

For developers transitioning between Flutter and modern TypeScript/React:

| Flutter Clean Architecture Layer | TypeScript / React Universal Equivalent | Location |
| :--- | :--- | :--- |
| **Entities / Data Models** | Pure TypeScript Interfaces & Types | `packages/core/src/types.ts` |
| **Use Cases / Domain Logic** | Pure Functions (No React/UI hooks) | `packages/core/` or `features/{name}/domain/` |
| **Repositories / Data Sources** | API Clients, Supabase Singleton, Query Hooks | `packages/database/` or `features/{name}/data/` |
| **BLoC / Cubit / State** | Zustand Store with `persist` middleware | `features/{name}/store/use{Name}Store.ts` |
| **Pages / Screens** | Thin Route Controllers / Screens | `app/{route}/page.tsx` or `features/{name}/presentation/` |
| **Widgets / Sub-components** | Isolated Small UI Components (< 150 lines) | `features/{name}/components/` or `presentation/components/` |

---

## 💻 Web Architecture (Next.js App Router)

### Rule 1: Thin Route Controllers (< 30 Lines)
Never place business logic, large forms, or complex UI trees directly inside `app/*/page.tsx`. `page.tsx` must only act as a route entry point wrapping the Feature View inside Layout (Navbar/Footer).

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

## 📱 Mobile Architecture (Expo / React Native)

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

## 🖥️ Desktop Architecture (Tauri)

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

## 🔄 The Golden DRY Rules

1. **Pure Business Formulas Live in `@monorepo/core`**: Never calculate discounts, tax, shipping thresholds, or prices differently in Web, Mobile, or Desktop.
2. **State Decoupling**: Keep Zustand stores feature-scoped (`useCartStore`, `useAuthStore`, `usePOSStore`, `useOrderStore`). Avoid giant monolithic global stores.
3. **Pure Re-export Bridges**: When refactoring legacy files, replace old paths with clean 1-line re-exports:
   ```ts
   // apps/mobile/src/components/CartModal.tsx
   export * from "../features/cart/presentation/CartModal";
   ```
4. **Zero-any Strict TypeScript**: Enforce complete types across all boundary layers, props, and store states.

---

## 🎨 Unified Design System Tokens (`packages/core/src/theme.ts`)

All design tokens live in a **single, platform-agnostic** `theme.ts` inside `packages/core`. Both Web (Next.js) and Mobile (Expo) consume this file directly:

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
| **Elevation (RN)** | `THEME.elevation` | — | Spread: `...THEME.elevation.md` |
| **Shadows (CSS)** | `THEME.shadows` | `boxShadow: THEME.shadows.lg` | — |
| **Z-Index** | `THEME.zIndex` | `zIndex: THEME.zIndex.modal` | `zIndex` in `StyleSheet` |
| **Animation** | `THEME.animation` | `transition: ${THEME.animation.duration.normal}` | `Animated.timing(...)` |

---

## 🤖 Installation as Agent Skill (Claude / Antigravity / Cursor)

Copy the directory or `SKILL.md` to your agent customization directory:

- **Antigravity / Gemini Global**: `~/.gemini/config/skills/next-expo-tauri-feature-first-clean-architecture/`
- **Project Specific**: `.agents/skills/next-expo-tauri-feature-first-clean-architecture/`
- **Claude Code**: `.claude/skills/next-expo-tauri-feature-first-clean-architecture/`

---

## 📄 License

MIT © [Muhammad Faiz Asy Syauqi](https://github.com/mfaizasysyauqi)
