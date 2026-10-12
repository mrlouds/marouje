# Task 04: Sticky Header & Navigation with Mobile Drawer

## 1. Task Overview & Target Component/Page
Build a high-performance, responsive sticky navigation header with glassmorphism backdrop blur. The header houses the restaurant brand identity, desktop navigation links, responsive mobile drawer, language switcher (`EN/AR`), theme toggle (`Dark/Light`), and an interactive cart button with an animated counter reflecting the Zustand cart store.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── components/
│   ├── layout/
│   │   ├── header.tsx             # Main sticky navbar component
│   │   ├── nav-links.tsx          # Localized desktop navigation links
│   │   ├── mobile-nav-drawer.tsx  # Mobile off-canvas menu drawer
│   │   └── cart-trigger-btn.tsx   # Header cart badge button
│   └── ui/
│       ├── theme-toggle.tsx
│       └── locale-switcher.tsx
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Header Component (`components/layout/header.tsx`)
```tsx
'use client';

import * as React from 'react';
import Link from 'next/link';
import { Waves, Menu, X } from 'lucide-react';
import { type Locale } from '@/lib/i18n/config';
import { type Dictionary } from '@/lib/i18n/get-dictionary';
import { ThemeToggle } from '@/components/ui/theme-toggle';
import { LocaleSwitcher } from '@/components/ui/locale-switcher';
import { CartTriggerBtn } from './cart-trigger-btn';
import { MobileNavDrawer } from './mobile-nav-drawer';
import { cn } from '@/lib/utils';

interface HeaderProps {
  locale: Locale;
  dict: Dictionary;
}

export function Header({ locale, dict }: HeaderProps) {
  const [isScrolled, setIsScrolled] = React.useState(false);
  const [mobileMenuOpen, setMobileMenuOpen] = React.useState(false);

  React.useEffect(() => {
    const handleScroll = () => {
      setIsScrolled(window.scrollY > 20);
    };
    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  const navLinks = [
    { href: `#home`, label: dict.nav.home },
    { href: `#menu`, label: dict.nav.menu },
    { href: `#reservations`, label: dict.nav.reservations },
    { href: `#contact`, label: dict.nav.contact },
  ];

  return (
    <>
      <header
        className={cn(
          'fixed top-0 inset-x-0 z-40 transition-all duration-300',
          isScrolled
            ? 'glass-panel shadow-md py-3'
            : 'bg-transparent py-5'
        )}
      >
        <div className="container flex items-center justify-between">
          {/* Brand Logo */}
          <Link
            href={`/${locale}`}
            className="flex items-center gap-2 group font-bold text-xl sm:text-2xl text-foreground"
          >
            <span className="flex items-center justify-center h-10 w-10 rounded-xl bg-ocean-600 text-white shadow-md shadow-ocean-600/30 group-hover:scale-105 transition-transform duration-200">
              <Waves className="h-6 w-6 animate-pulse" />
            </span>
            <div className="flex flex-col leading-tight">
              <span className="font-extrabold tracking-tight text-ocean-700 dark:text-ocean-300">
                {locale === 'ar' ? 'مروج البحر' : 'Marouj Al Bahr'}
              </span>
              <span className="text-[10px] tracking-wider uppercase text-muted-foreground font-semibold">
                {locale === 'ar' ? 'مأكولات بحرية فاخرة' : 'Seafood Dining'}
              </span>
            </div>
          </Link>

          {/* Desktop Nav Items */}
          <nav className="hidden md:flex items-center gap-8 text-sm font-semibold">
            {navLinks.map((item) => (
              <a
                key={item.href}
                href={item.href}
                className="text-foreground/80 hover:text-ocean-600 dark:hover:text-ocean-400 transition-colors"
              >
                {item.label}
              </a>
            ))}
          </nav>

          {/* Action Center: Cart, Theme, Locale, Hamburger */}
          <div className="flex items-center gap-2 sm:gap-3">
            <CartTriggerBtn />
            <ThemeToggle className="hidden sm:inline-flex" />
            <LocaleSwitcher currentLocale={locale} />

            {/* Mobile Hamburger Button */}
            <button
              onClick={() => setMobileMenuOpen(true)}
              className="md:hidden inline-flex items-center justify-center h-10 w-10 rounded-full border border-border bg-card/80 text-foreground"
              aria-label="Open mobile navigation"
            >
              <Menu className="h-5 w-5" />
            </button>
          </div>
        </div>
      </header>

      {/* Mobile Off-canvas Drawer */}
      <MobileNavDrawer
        isOpen={mobileMenuOpen}
        onClose={() => setMobileMenuOpen(false)}
        links={navLinks}
        locale={locale}
      />
    </>
  );
}
```

### 3.2 Cart Trigger Badge (`components/layout/cart-trigger-btn.tsx`)
Connects to the Zustand cart store to show an animated badge with the current quantity.

```tsx
'use client';

import * as React from 'react';
import { ShoppingBag } from 'lucide-react';
import { useCartStore } from '@/lib/store/cart-store';
import { cn } from '@/lib/utils';

export function CartTriggerBtn() {
  const [mounted, setMounted] = React.useState(false);
  const totalItems = useCartStore((state) => state.totalItems());
  const toggleCart = useCartStore((state) => state.toggleCart);

  React.useEffect(() => {
    setMounted(true);
  }, []);

  return (
    <button
      onClick={toggleCart}
      className="relative inline-flex items-center justify-center h-10 w-10 rounded-full border border-border bg-card/80 text-foreground shadow-sm hover:bg-ocean-100 dark:hover:bg-ocean-900/50 hover:text-ocean-600 dark:hover:text-ocean-400 transition-all duration-200"
      aria-label="Open Cart"
    >
      <ShoppingBag className="h-5 w-5" />
      {mounted && totalItems > 0 && (
        <span
          key={totalItems}
          className={cn(
            'absolute -top-1 -right-1 flex h-5 min-w-[1.25rem] items-center justify-center rounded-full bg-coral-500 px-1 text-[11px] font-bold text-white shadow-md',
            'animate-bounce'
          )}
        >
          {totalItems}
        </span>
      )}
    </button>
  );
}
```

### 3.3 Mobile Nav Drawer (`components/layout/mobile-nav-drawer.tsx`)
```tsx
'use client';

import * as React from 'react';
import { X } from 'lucide-react';
import { ThemeToggle } from '@/components/ui/theme-toggle';
import { type Locale } from '@/lib/i18n/config';
import { cn } from '@/lib/utils';

interface MobileNavDrawerProps {
  isOpen: boolean;
  onClose: () => void;
  links: { href: string; label: string }[];
  locale: Locale;
}

export function MobileNavDrawer({
  isOpen,
  onClose,
  links,
  locale,
}: MobileNavDrawerProps) {
  React.useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = '';
    }
    return () => {
      document.body.style.overflow = '';
    };
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 z-50 md:hidden">
      {/* Backdrop */}
      <div
        className="fixed inset-0 bg-black/60 backdrop-blur-sm transition-opacity"
        onClick={onClose}
      />

      {/* Drawer */}
      <div
        className={cn(
          'fixed inset-y-0 w-3/4 max-w-sm bg-card p-6 shadow-2xl transition-transform duration-300 flex flex-col justify-between',
          locale === 'ar' ? 'right-0' : 'left-0'
        )}
      >
        <div>
          <div className="flex items-center justify-between pb-6 border-b border-border">
            <span className="font-bold text-lg text-ocean-700 dark:text-ocean-300">
              {locale === 'ar' ? 'مروج البحر' : 'Marouj Al Bahr'}
            </span>
            <button
              onClick={onClose}
              className="p-2 rounded-full hover:bg-muted text-muted-foreground hover:text-foreground"
            >
              <X className="h-5 w-5" />
            </button>
          </div>

          <nav className="mt-8 flex flex-col gap-4">
            {links.map((link) => (
              <a
                key={link.href}
                href={link.href}
                onClick={onClose}
                className="text-lg font-medium text-foreground hover:text-ocean-600 transition-colors py-2 border-b border-border/40"
              >
                {link.label}
              </a>
            ))}
          </nav>
        </div>

        <div className="pt-6 border-t border-border flex items-center justify-between">
          <span className="text-sm text-muted-foreground font-medium">
            {locale === 'ar' ? 'المظهر' : 'Theme'}
          </span>
          <ThemeToggle />
        </div>
      </div>
    </div>
  );
}
```

---

## 4. Acceptance Criteria
- [ ] Header sticks to the top with glassmorphism blur and reduces padding when scrolling down.
- [ ] Responsive navigation works smoothly across mobile, tablet, and desktop breakpoints.
- [ ] Mobile drawer opens smoothly with a backdrop blur and prevents body scroll while active.
- [ ] Cart trigger button updates badge count optimistically based on Zustand state.
- [ ] RTL/LTR alignment adapts properly for mobile drawer slide direction.
