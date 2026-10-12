# Bilingual Seafood Restaurant Landing Page - Master Execution Plan

Welcome to the engineering architecture and task specification guide for the high-performance, bilingual (Arabic & English) **Seafood Restaurant Landing Page** built on Next.js 14+ (App Router), TypeScript, Tailwind CSS, and Zustand.

---

## Architecture Blueprint & Folder Tree

```
seafood-restaurant/
├── docs/
│   ├── 01-architecture-and-setup/
│   │   ├── 01-nextjs-tailwind-setup.md
│   │   ├── 02-theme-provider-setup.md
│   │   └── 03-i18n-rtl-configuration.md
│   ├── 02-components-and-ui/
│   │   ├── 01-header-and-navigation.md
│   │   ├── 02-hero-section.md
│   │   ├── 03-interactive-menu.md
│   │   ├── 04-cart-drawer-whatsapp.md
│   │   └── 05-reservation-form.md
│   └── 03-data-and-state/
│       ├── 01-zustand-cart-store.md
│       └── 02-menu-data-schema.md
├── app/
│   ├── actions/
│   │   └── reservation-action.ts
│   ├── [locale]/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── favicon.ico
│   └── globals.css
├── components/
│   ├── cart/
│   │   ├── cart-drawer.tsx
│   │   └── whatsapp-order-btn.tsx
│   ├── home/
│   │   ├── hero-section.tsx
│   │   └── hero-badge.tsx
│   ├── layout/
│   │   ├── header.tsx
│   │   ├── mobile-nav-drawer.tsx
│   │   └── cart-trigger-btn.tsx
│   ├── menu/
│   │   ├── menu-section.tsx
│   │   ├── category-tabs.tsx
│   │   └── menu-card.tsx
│   ├── providers/
│   │   └── theme-provider.tsx
│   ├── reservation/
│   │   ├── reservation-form.tsx
│   │   └── reservation-section.tsx
│   └── ui/
│       ├── locale-switcher.tsx
│       └── theme-toggle.tsx
├── data/
│   └── menu-items.ts
├── dictionaries/
│   ├── ar.json
│   └── en.json
├── lib/
│   ├── i18n/
│   │   ├── config.ts
│   │   └── get-dictionary.ts
│   ├── store/
│   │   ├── cart-store.ts
│   │   └── use-cart-hydration.ts
│   ├── utils.ts
│   ├── validation/
│   │   └── reservation-schema.ts
│   └── whatsapp/
│       └── generate-order-link.ts
├── styles/
│   └── fonts.ts
├── types/
│   └── menu.ts
├── middleware.ts
├── next.config.mjs
├── package.json
└── tailwind.config.ts
```

---

## Phased Implementation Sequence

| Phase | Task Specification | Key Focus |
|---|---|---|
| **Phase 1** | `01-nextjs-tailwind-setup.md` | Core framework, oceanic color system, Google Fonts (`Tajawal` & `Inter`). |
| **Phase 1** | `02-theme-provider-setup.md` | `next-themes` integration, dark/light CSS variables, zero-flicker hydration. |
| **Phase 1** | `03-i18n-rtl-configuration.md` | Dynamic `[locale]` routes, Edge Middleware, JSON dictionaries, RTL/LTR layout. |
| **Phase 2** | `01-zustand-cart-store.md` | Zustand store, localStorage persistence, hydration guard. |
| **Phase 2** | `02-menu-data-schema.md` | Bilingual menu items schema, categories, mock seafood catalog. |
| **Phase 3** | `01-header-and-navigation.md` | Sticky glassmorphism header, responsive mobile drawer, cart badge counter. |
| **Phase 3** | `02-hero-section.md` | Catch-of-the-day badge, bilingual CTAs, responsive LCP optimization. |
| **Phase 3** | `03-interactive-menu.md` | Real-time category filtering, product cards, optimistic add-to-cart. |
| **Phase 3** | `04-cart-drawer-whatsapp.md` | Slide-over drawer, quantity controls, encoded WhatsApp order builder. |
| **Phase 3** | `05-reservation-form.md` | Table reservation with Zod validation and Next.js Server Action. |
