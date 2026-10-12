# Task 03: i18n & RTL/LTR Localization Configuration

## 1. Task Overview & Target Component/Page
Implement complete internationalization (i18n) and bidirectional rendering (RTL for Arabic, LTR for English) using Next.js App Router dynamic routing `[locale]`. This includes URL path-based routing (`/ar`, `/en`), Next.js Edge Middleware for locale resolution, JSON dictionary loaders with full type safety, and automatic font switching (`Tajawal` for Arabic, `Inter` for English).

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── middleware.ts                         # Edge middleware for locale detection & rewriting
├── dictionaries/
│   ├── ar.json                           # Arabic string dictionary
│   └── en.json                           # English string dictionary
├── lib/
│   └── i18n/
│       ├── config.ts                     # Supported locales, default locale, types
│       └── get-dictionary.ts             # Server-side dictionary retrieval function
├── components/
│   └── ui/
│       └── locale-switcher.tsx           # Language toggle component
└── app/
    └── [locale]/
        ├── layout.tsx                    # Localized layout setting dir & font classes
        └── page.tsx                      # Localized landing page entry
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 i18n Configuration & Types (`lib/i18n/config.ts`)
```typescript
export const i18n = {
  defaultLocale: 'ar',
  locales: ['ar', 'en'] as const,
} as const;

export type Locale = (typeof i18n['locales'])[number];

export interface LocaleConfig {
  name: string;
  nativeName: string;
  dir: 'rtl' | 'ltr';
  fontClass: string;
}

export const localeDetails: Record<Locale, LocaleConfig> = {
  ar: {
    name: 'Arabic',
    nativeName: 'العربية',
    dir: 'rtl',
    fontClass: 'font-tajawal',
  },
  en: {
    name: 'English',
    nativeName: 'English',
    dir: 'ltr',
    fontClass: 'font-inter',
  },
};
```

### 3.2 Edge Middleware for Routing (`middleware.ts`)
The middleware intercepts requests, checks for a supported locale in the pathname, and rewrites or redirects based on cookies or default preferences.

```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { i18n } from '@/lib/i18n/config';

export function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;

  // Ignore static assets, images, API routes, and Next.js internal files
  if (
    [
      '/manifest.json',
      '/favicon.ico',
      '/robots.txt',
      '/sitemap.xml',
    ].includes(pathname) ||
    pathname.startsWith('/api') ||
    pathname.startsWith('/_next') ||
    pathname.match(/\.(.*)$/)
  ) {
    return NextResponse.next();
  }

  // Check if pathname has a supported locale prefix
  const pathnameIsMissingLocale = i18n.locales.every(
    (locale) => !pathname.startsWith(`/${locale}/`) && pathname !== `/${locale}`
  );

  if (pathnameIsMissingLocale) {
    const locale = request.cookies.get('NEXT_LOCALE')?.value || i18n.defaultLocale;

    return NextResponse.redirect(
      new URL(
        `/${locale}${pathname.startsWith('/') ? '' : '/'}${pathname}`,
        request.url
      )
    );
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

### 3.3 Dictionary Loader (`lib/i18n/get-dictionary.ts`)
Server-side lazy loading of dictionary files to maintain minimal initial bundle footprints.

```typescript
import 'server-only';
import type { Locale } from './config';

const dictionaries = {
  ar: () => import('@/dictionaries/ar.json').then((module) => module.default),
  en: () => import('@/dictionaries/en.json').then((module) => module.default),
};

export const getDictionary = async (locale: Locale) => {
  if (!dictionaries[locale]) {
    return dictionaries.ar();
  }
  return dictionaries[locale]();
};

export type Dictionary = Awaited<ReturnType<typeof getDictionary>>;
```

### 3.4 Arabic & English Dictionaries (`dictionaries/ar.json` & `dictionaries/en.json`)

**`dictionaries/ar.json`:**
```json
{
  "nav": {
    "home": "الرئيسية",
    "menu": "قائمة الطعام",
    "about": "عن المطعم",
    "reservations": "حجز طاولة",
    "contact": "تواصل معنا",
    "cart": "سلة المأكولات"
  },
  "hero": {
    "badge": "صيد البحر الطازج يومياً",
    "title": "أشهى المأكولات البحرية الفاخرة بطابع ساحلي أصيل",
    "subtitle": "نختار أجود أنواع الأسماك وثمار البحر مباشرة من الصيادين المحليين، ونحضرها بأيدي أمهر الطهاة لتجربة تذوق استثنائية.",
    "ctaMenu": "استكشف القائمة",
    "ctaReserve": "احجز طاولتك الآن",
    "stats": {
      "freshness": "طازج 100%",
      "dishes": "+45 طبق بحري",
      "rating": "4.9 تقييم العملاء"
    }
  },
  "menu": {
    "title": "قائمة المأكولات البحرية",
    "subtitle": "أطباق تم إعدادها بشغف مستوحاة من ثراء البحر المتوسط والخليج العربي",
    "all": "الكل",
    "addToCart": "إضافة للطلب",
    "currency": "ر.س",
    "specialBadge": "مميز",
    "spicyBadge": "حار"
  },
  "cart": {
    "title": "سلة الطلبات",
    "empty": "سلة المأكولات فارغة حالياً",
    "emptyDesc": "تصفح قائمتنا واختر وجباتك البحرية المفضلة لطلبها عبر واتساب",
    "total": "المجموع الإجمالي",
    "specialInstructions": "ملاحظات إضافية على الطلب (تتبيلة، طريقة الطهي...)",
    "sendWhatsApp": "إرسال الطلب عبر واتساب",
    "clearCart": "تفريغ السلة"
  },
  "reservation": {
    "badge": "تجربة طعام لا تُنسى",
    "title": "احجز طاولتك مع إطلالة بحرية",
    "subtitle": "يرجى تعبئة النموذج لحجز مقعدك، وسيتم تأكيد حجزك فوراً برسالة نصية أو مكالمة",
    "nameLabel": "الاسم الكامل",
    "phoneLabel": "رقم الجوال",
    "dateLabel": "تاريخ الحجز",
    "timeLabel": "وقت الحضور",
    "guestsLabel": "عدد الأفراد",
    "seatingLabel": "مكان الجلوس",
    "indoor": "الصالة الداخلية الفاخرة",
    "terrace": "الشرفة الخارجية",
    "seaView": "إطلالة البحر المباشرة",
    "notesLabel": "طلبات خاصة أو مناسبة خاصة",
    "submitBtn": "تأكيد الحجز الآن",
    "submitting": "جاري تأكيد الحجز...",
    "successMessage": "تم استلام طلب الحجز بنجاح! سنتواصل معك للتأكيد خلال دقائق."
  }
}
```

**`dictionaries/en.json`:**
```json
{
  "nav": {
    "home": "Home",
    "menu": "Menu",
    "about": "About",
    "reservations": "Reservations",
    "contact": "Contact",
    "cart": "Seafood Cart"
  },
  "hero": {
    "badge": "Fresh Catch Daily",
    "title": "Gourmet Ocean Dining & Coastal Mediterranean Flavors",
    "subtitle": "Directly sourced from local fishermen and crafted by master chefs for an unforgettable oceanic culinary adventure.",
    "ctaMenu": "Explore Menu",
    "ctaReserve": "Reserve a Table",
    "stats": {
      "freshness": "100% Fresh Catch",
      "dishes": "+45 Signature Dishes",
      "rating": "4.9 Guest Rating"
    }
  },
  "menu": {
    "title": "Fresh Seafood Menu",
    "subtitle": "Handcrafted culinary masterpieces inspired by coastal traditions and exotic marine spices",
    "all": "All",
    "addToCart": "Add to Order",
    "currency": "SAR",
    "specialBadge": "Chef Special",
    "spicyBadge": "Spicy"
  },
  "cart": {
    "title": "Your Seafood Order",
    "empty": "Your cart is currently empty",
    "emptyDesc": "Explore our menu and add delicious seafood specialties to place an instant WhatsApp order.",
    "total": "Total Amount",
    "specialInstructions": "Special notes or cooking preferences (grill level, spices...)",
    "sendWhatsApp": "Order via WhatsApp",
    "clearCart": "Clear Cart"
  },
  "reservation": {
    "badge": "Unforgettable Dining",
    "title": "Reserve Your Coastal Table",
    "subtitle": "Fill out the form below to secure your spot. We will confirm your reservation immediately.",
    "nameLabel": "Full Name",
    "phoneLabel": "Phone Number",
    "dateLabel": "Reservation Date",
    "timeLabel": "Time",
    "guestsLabel": "Number of Guests",
    "seatingLabel": "Seating Area",
    "indoor": "Luxury Indoor Hall",
    "terrace": "Outdoor Terrace",
    "seaView": "Direct Seafront View",
    "notesLabel": "Special Requests or Occasion",
    "submitBtn": "Confirm Table Reservation",
    "submitting": "Confirming Booking...",
    "successMessage": "Your table reservation request was received! We will contact you shortly to confirm."
  }
}
```

### 3.5 Localized Layout Wrapper (`app/[locale]/layout.tsx`)
```tsx
import type { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { inter, tajawal } from '@/styles/fonts';
import { i18n, localeDetails, type Locale } from '@/lib/i18n/config';
import { ThemeProvider } from '@/components/providers/theme-provider';

export function generateStaticParams() {
  return i18n.locales.map((locale) => ({ locale }));
}

export async function generateMetadata({
  params: { locale },
}: {
  params: { locale: Locale };
}): Promise<Metadata> {
  const isAr = locale === 'ar';
  return {
    title: isAr
      ? 'مطعم مروج البحر | مأكولات بحرية فاخرة وطازجة'
      : 'Marouj Al Bahr | Luxury Fresh Seafood Dining',
    description: isAr
      ? 'تجربة طعام بحرية استثنائية مع أشهى الأسماك وثمار البحر الطازجة يومياً.'
      : 'Exceptional coastal seafood culinary experience featuring daily fresh catch and seaside dining.',
  };
}

export default async function RootLocaleLayout({
  children,
  params: { locale },
}: {
  children: React.ReactNode;
  params: { locale: string };
}) {
  if (!i18n.locales.includes(locale as Locale)) {
    notFound();
  }

  const currentLocale = locale as Locale;
  const config = localeDetails[currentLocale];
  const fontVariable = currentLocale === 'ar' ? tajawal.variable : inter.variable;
  const fontClass = currentLocale === 'ar' ? tajawal.className : inter.className;

  return (
    <html
      lang={currentLocale}
      dir={config.dir}
      className={`${fontVariable} ${fontClass}`}
      suppressHydrationWarning
    >
      <body className="min-h-screen bg-background text-foreground antialiased selection:bg-ocean-500 selection:text-white transition-colors duration-300">
        <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### 3.6 Locale Switcher Component (`components/ui/locale-switcher.tsx`)
```tsx
'use client';

import * as React from 'react';
import { usePathname, useRouter } from 'next/navigation';
import { Globe } from 'lucide-react';
import { i18n, type Locale } from '@/lib/i18n/config';
import { cn } from '@/lib/utils';

export function LocaleSwitcher({ currentLocale }: { currentLocale: Locale }) {
  const pathname = usePathname();
  const router = useRouter();

  const toggleLocale = () => {
    const targetLocale: Locale = currentLocale === 'ar' ? 'en' : 'ar';
    const segments = pathname.split('/');
    segments[1] = targetLocale;
    const newPath = segments.join('/') || `/${targetLocale}`;
    
    // Set cookie for persistent preference
    document.cookie = `NEXT_LOCALE=${targetLocale}; path=/; max-age=31536000; SameSite=Lax`;
    router.push(newPath);
    router.refresh();
  };

  return (
    <button
      onClick={toggleLocale}
      className={cn(
        'inline-flex items-center gap-1.5 px-3 py-1.5 rounded-full border border-border text-sm font-semibold',
        'bg-card/80 text-foreground hover:bg-ocean-100 hover:text-ocean-700 dark:hover:bg-ocean-900/50 dark:hover:text-ocean-300',
        'transition-colors duration-200 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring'
      )}
      aria-label={`Switch language to ${currentLocale === 'ar' ? 'English' : 'العربية'}`}
    >
      <Globe className="h-4 w-4 text-ocean-500" />
      <span>{currentLocale === 'ar' ? 'EN' : 'عربي'}</span>
    </button>
  );
}
```

---

## 4. Acceptance Criteria
- [ ] Route URLs dynamically prefix with `/ar` and `/en` via `[locale]` segment.
- [ ] Edge Middleware automatically detects locale and enforces routing.
- [ ] Document attribute `dir="rtl"` applied for Arabic and `dir="ltr"` for English.
- [ ] Font classes dynamically toggle between `Tajawal` (Arabic) and `Inter` (English).
- [ ] Locale switcher toggles language smoothly and sets `NEXT_LOCALE` cookie.
- [ ] Dictionaries loaded on the server with zero client bundle overhead.
