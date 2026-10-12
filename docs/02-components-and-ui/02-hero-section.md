# Task 05: Hero Section with Localized Visuals & Dual CTAs

## 1. Task Overview & Target Component/Page
Design and implement a visually captivating, high-conversion Hero section. It showcases an atmospheric coastal dining presentation, localized promotional badge ("Fresh Catch Daily"), fluid typography with bilingual headline and description, dual localized call-to-actions ("Explore Menu" and "Reserve a Table"), and key social-proof indicators.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── components/
│   └── home/
│       ├── hero-section.tsx       # Main hero container
│       ├── hero-badge.tsx         # Catch of the Day pill badge
│       └── hero-stats.tsx         # Social proof metrics card
├── public/
│   └── images/
│       └── hero/
│           ├── hero-bg.webp       # Optimized oceanic atmosphere background
│           └── hero-dish.webp     # High-resolution signature seafood platter
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Hero Section Component (`components/home/hero-section.tsx`)
```tsx
import Image from 'next/image';
import Link from 'next/link';
import { ArrowLeft, ArrowRight, UtensilsCrossed, Calendar, Fish, Star, Sparkles } from 'lucide-react';
import { type Locale } from '@/lib/i18n/config';
import { type Dictionary } from '@/lib/i18n/get-dictionary';

interface HeroSectionProps {
  locale: Locale;
  dict: Dictionary;
}

export function HeroSection({ locale, dict }: HeroSectionProps) {
  const isAr = locale === 'ar';
  const ArrowIcon = isAr ? ArrowLeft : ArrowRight;

  return (
    <section id="home" className="relative min-h-[92vh] flex items-center pt-28 pb-16 overflow-hidden">
      {/* Background Decorative Gradients & Oceanic Glow */}
      <div className="absolute inset-0 pointer-events-none overflow-hidden">
        <div className="absolute -top-40 -right-40 w-96 h-96 bg-ocean-400/20 dark:bg-ocean-600/15 rounded-full blur-3xl" />
        <div className="absolute top-1/2 -left-40 w-96 h-96 bg-secondary/15 dark:bg-secondary/10 rounded-full blur-3xl" />
      </div>

      <div className="container relative z-10 grid lg:grid-cols-12 gap-12 items-center">
        {/* Left Column: Copy & Actions */}
        <div className="lg:col-span-7 flex flex-col items-start gap-6 text-start">
          {/* Fresh Catch Daily Badge */}
          <div className="inline-flex items-center gap-2 px-4 py-2 rounded-full border border-ocean-200 dark:border-ocean-800 bg-ocean-50/80 dark:bg-ocean-950/80 text-ocean-700 dark:text-ocean-300 text-xs sm:text-sm font-bold shadow-sm">
            <Fish className="h-4 w-4 text-ocean-500 animate-pulse" />
            <span>{dict.hero.badge}</span>
            <span className="flex h-2 w-2 rounded-full bg-emerald-500" />
          </div>

          {/* Main Headline */}
          <h1 className="text-4xl sm:text-5xl lg:text-6xl font-extrabold tracking-tight leading-[1.15] text-foreground">
            {dict.hero.title}
          </h1>

          {/* Supporting Subtitle */}
          <p className="text-base sm:text-lg text-muted-foreground max-w-2xl leading-relaxed">
            {dict.hero.subtitle}
          </p>

          {/* Dual CTAs */}
          <div className="flex flex-wrap items-center gap-4 pt-2 w-full sm:w-auto">
            <a
              href="#menu"
              className="w-full sm:w-auto inline-flex items-center justify-center gap-2 px-7 py-3.5 rounded-xl font-bold text-white bg-ocean-600 hover:bg-ocean-700 dark:bg-ocean-500 dark:hover:bg-ocean-600 shadow-lg shadow-ocean-600/25 transition-all duration-200 hover:-translate-y-0.5 active:translate-y-0"
            >
              <UtensilsCrossed className="h-5 w-5" />
              <span>{dict.hero.ctaMenu}</span>
              <ArrowIcon className="h-4 w-4" />
            </a>

            <a
              href="#reservations"
              className="w-full sm:w-auto inline-flex items-center justify-center gap-2 px-7 py-3.5 rounded-xl font-bold border border-border bg-card/80 hover:bg-muted text-foreground transition-all duration-200 hover:-translate-y-0.5"
            >
              <Calendar className="h-5 w-5 text-ocean-500" />
              <span>{dict.hero.ctaReserve}</span>
            </a>
          </div>

          {/* Social Proof Stats */}
          <div className="grid grid-cols-3 gap-4 pt-6 border-t border-border/60 w-full max-w-lg">
            <div className="flex flex-col">
              <span className="text-xl sm:text-2xl font-black text-ocean-600 dark:text-ocean-400">
                100%
              </span>
              <span className="text-xs sm:text-sm text-muted-foreground font-medium">
                {dict.hero.stats.freshness}
              </span>
            </div>
            <div className="flex flex-col">
              <span className="text-xl sm:text-2xl font-black text-ocean-600 dark:text-ocean-400">
                +45
              </span>
              <span className="text-xs sm:text-sm text-muted-foreground font-medium">
                {dict.hero.stats.dishes}
              </span>
            </div>
            <div className="flex flex-col">
              <span className="text-xl sm:text-2xl font-black text-amber-500 flex items-center gap-1">
                4.9 <Star className="h-4 w-4 fill-amber-500" />
              </span>
              <span className="text-xs sm:text-sm text-muted-foreground font-medium">
                {dict.hero.stats.rating}
              </span>
            </div>
          </div>
        </div>

        {/* Right Column: Hero Visual Asset */}
        <div className="lg:col-span-5 relative flex items-center justify-center">
          <div className="relative w-full aspect-square max-w-md lg:max-w-none">
            {/* Glow halo */}
            <div className="absolute inset-4 rounded-full bg-gradient-to-tr from-ocean-500/20 to-coral-500/20 blur-2xl transform scale-95" />

            {/* Platter Frame */}
            <div className="relative h-full w-full rounded-3xl overflow-hidden border border-white/20 dark:border-ocean-700/40 shadow-2xl glass-panel p-3">
              <div className="relative w-full h-full rounded-2xl overflow-hidden bg-ocean-950/20">
                <Image
                  src="/images/hero/hero-dish.webp"
                  alt={isAr ? 'أطباق مأكولات بحرية طازجة' : 'Signature Fresh Seafood Platter'}
                  fill
                  priority
                  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 40vw"
                  className="object-cover transition-transform duration-700 hover:scale-105"
                />

                {/* Floating Special Dish Badge */}
                <div className="absolute bottom-4 inset-x-4 glass-panel p-3.5 rounded-xl shadow-lg flex items-center justify-between">
                  <div className="flex items-center gap-3">
                    <div className="p-2 rounded-lg bg-amber-500/10 text-amber-500">
                      <Sparkles className="h-5 w-5" />
                    </div>
                    <div>
                      <p className="text-xs text-muted-foreground font-medium">
                        {isAr ? 'طبق الشيف اليومي' : "Chef's Special"}
                      </p>
                      <p className="text-sm font-bold text-foreground">
                        {isAr ? 'استاكوزا مشوية بالزبدة والأعشاب' : 'Grilled Herb Butter Lobster'}
                      </p>
                    </div>
                  </div>
                  <span className="font-extrabold text-ocean-600 dark:text-ocean-400 text-sm">
                    {isAr ? '195 ر.س' : '195 SAR'}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
```

---

## 4. Acceptance Criteria
- [ ] Hero headline and descriptive text rendered responsively across phone, tablet, and widescreen.
- [ ] Both call-to-action buttons smoothly scroll to `#menu` and `#reservations`.
- [ ] Arrow icons in buttons change direction automatically based on Arabic (left) vs English (right).
- [ ] Hero visual image configured with Next.js `Image` using `priority` to achieve high LCP performance.
- [ ] Floating Chef's special highlight card renders with proper glassmorphism design tokens.
