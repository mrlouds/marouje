# مطعم مروج البحر | Marouj Al Bahr Seafood Restaurant 🌊

> **Bilingual (Arabic / English) High-Performance Seafood Restaurant Landing Page & Ordering System.**
> Built with Next.js 14+ (App Router), TypeScript, Tailwind CSS, and Zustand.

---

## 🌟 Key Features

- **🌐 Full Bilingual Support (i18n):**
  - Arabic (RTL) with typography powered by Google Font `Tajawal`.
  - English (LTR) with typography powered by Google Font `Inter`.
- **🌗 Theme System:** Seamless Light / Dark mode toggle with persistent state and custom ocean palette tokens.
- **🦐 Interactive Seafood Menu:**
  - Dynamic category filtering (Catch of the Day, Signature Platters, Seafood Appetizers, Soups & Salads, Beverages).
  - Dietary and chef recommendation badges (Spicy, Chef's Special).
- **🛍️ Slide-Over Cart Drawer:**
  - Dynamic cart management (item counters, additions, deletions, kitchen notes).
  - **Instant WhatsApp Order Generator**: Encodes the cart contents, subtotal, customer cooking notes, and sends it directly via WhatsApp Web/App.
- **📅 Table Reservation Section:**
  - Seating preference selection (Seafront view, Outdoor terrace, Indoor hall).
  - Form validation with feedback banners.

---

## 📁 Project Structure

```
├── docs/                           # Complete architectural task specifications (.md)
│   ├── 01-architecture-and-setup/  # Setup, Next.js, Tailwind, next-themes, i18n
│   ├── 02-components-and-ui/       # Header, Hero, Menu, Cart Drawer, Reservations
│   └── 03-data-and-state/          # Zustand store, schema & catalog dataset
├── index.html                      # Standalone interactive landing page preview
├── preview.html                    # Local preview file with full feature-set
├── .gitignore                      # Git ignore patterns
└── README.md                       # Project overview & documentation
```

---

## 🚀 Instant Preview

You can open `index.html` or `preview.html` directly in any web browser to explore all features interactively.

---

## 📖 Architecture & Development Specs

Detailed modular task files for developing or extending the Next.js App Router codebase are organized under the [`docs/`](./docs/) directory.
