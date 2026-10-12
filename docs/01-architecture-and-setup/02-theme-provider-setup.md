# Task 02: Theme Provider & Dark/Light Mode Integration

## 1. Task Overview & Target Component/Page
Integrate `next-themes` to support smooth, flicker-free Light and Dark mode transitions across the entire application. The theme state must be stored in `localStorage`, sync seamlessly across browser tabs, and respect system preferences by default. The design system leverages nautical seafood color palettes tailored for day and night dining experiences.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── components/
│   └── providers/
│       └── theme-provider.tsx     # Client wrapper around next-themes
│   └── ui/
│       └── theme-toggle.tsx       # Reusable toggle button (Sun / Moon / System)
├── app/
│   ├── [locale]/
│   │   └── layout.tsx             # Root layout wrapping children with ThemeProvider
│   └── globals.css                # CSS variables and transition classes
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Theme Provider Implementation (`components/providers/theme-provider.tsx`)
Because `next-themes` relies on client hooks and local storage, it must be encapsulated inside a `'use client'` component.

```typescript
'use client';

import * as React from 'react';
import { ThemeProvider as NextThemesProvider } from 'next-themes';
import type { ThemeProviderProps } from 'next-themes';

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>;
}
```

### 3.2 HTML Document Setup (`app/[locale]/layout.tsx`)
To prevent the hydration mismatch warning from Next.js when `next-themes` injects the `class="dark"` attribute before hydration:
- Include `suppressHydrationWarning` on the `<html>` element.
- Configure `attribute="class"`, `defaultTheme="system"`, and `enableSystem={true}`.

```tsx
import { ThemeProvider } from '@/components/providers/theme-provider';

export default function LocaleLayout({
  children,
  params: { locale },
}: {
  children: React.ReactNode;
  params: { locale: string };
}) {
  return (
    <html lang={locale} dir={locale === 'ar' ? 'rtl' : 'ltr'} suppressHydrationWarning>
      <body className="min-h-screen bg-background text-foreground antialiased selection:bg-ocean-500 selection:text-white transition-colors duration-300">
        <ThemeProvider
          attribute="class"
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange={false}
        >
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### 3.3 Theme Toggle Component (`components/ui/theme-toggle.tsx`)
A resilient, accessible toggle button that handles client hydration gracefully to prevent initial render icon flickering.

```tsx
'use client';

import * as React from 'react';
import { useTheme } from 'next-themes';
import { Sun, Moon } from 'lucide-react';
import { cn } from '@/lib/utils';

interface ThemeToggleProps {
  className?: string;
}

export function ThemeToggle({ className }: ThemeToggleProps) {
  const { resolvedTheme, setTheme } = useTheme();
  const [mounted, setMounted] = React.useState(false);

  React.useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return (
      <div
        className={cn('h-10 w-10 rounded-full border border-border bg-card/60 p-2 opacity-50', className)}
        aria-hidden="true"
      />
    );
  }

  const isDark = resolvedTheme === 'dark';

  return (
    <button
      type="button"
      onClick={() => setTheme(isDark ? 'light' : 'dark')}
      className={cn(
        'relative inline-flex h-10 w-10 items-center justify-center rounded-full border border-border',
        'bg-card/80 text-foreground shadow-sm transition-all duration-300',
        'hover:bg-ocean-100 hover:text-ocean-700 dark:hover:bg-ocean-900/50 dark:hover:text-ocean-300',
        'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2',
        className
      )}
      aria-label={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
      title={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
    >
      <Sun
        className={cn(
          'h-5 w-5 transition-all duration-300',
          isDark ? 'rotate-90 scale-0 opacity-0' : 'rotate-0 scale-100 opacity-100 text-amber-500'
        )}
      />
      <Moon
        className={cn(
          'absolute h-5 w-5 transition-all duration-300',
          isDark ? 'rotate-0 scale-100 opacity-100 text-cyan-400' : '-rotate-90 scale-0 opacity-0'
        )}
      />
    </button>
  );
}
```

### 3.4 Smooth Transition Handling (`app/globals.css`)
Ensure color transitions do not lag or jump abruptly when switching themes:

```css
*,
*::before,
*::after {
  transition-property: background-color, border-color, color, fill, stroke;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
}
```

---

## 4. Acceptance Criteria
- [ ] `next-themes` installed and integrated with `ThemeProvider` client wrapper.
- [ ] `suppressHydrationWarning` added to the `<html>` element in root locale layout.
- [ ] Theme toggle switch works smoothly between light, dark, and system modes.
- [ ] No Flash of Unstyled Content (FOUC) or hydration mismatch errors in the console.
- [ ] Persistent state: Theme preference survives page reloads and cross-tab navigation.
