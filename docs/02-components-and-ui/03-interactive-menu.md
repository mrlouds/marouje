# Task 06: Interactive Seafood Menu with Category Filtering

## 1. Task Overview & Target Component/Page
Implement the interactive seafood menu section featuring category filter pills, responsive grid cards, localized dish titles/descriptions/pricing, dietary badges (e.g. `Spicy`, `Chef's Special`, `Gluten-Free`), and direct integration with the Zustand cart store with immediate visual feedback when an item is added.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── components/
│   └── menu/
│       ├── menu-section.tsx         # Main interactive menu component
│       ├── category-tabs.tsx        # Horizontal scrolling category filter pills
│       ├── menu-card.tsx            # Single seafood dish card with Add-to-Cart
│       └── dietary-badge.tsx        # Micro-badges for spicy, chef special, etc.
├── data/
│   └── menu-items.ts                # Bilingual catalog dataset
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Menu Section Component (`components/menu/menu-section.tsx`)
```tsx
'use client';

import * as React from 'react';
import { type Locale } from '@/lib/i18n/config';
import { type Dictionary } from '@/lib/i18n/get-dictionary';
import { menuCategories, menuItems, type MenuItem } from '@/data/menu-items';
import { CategoryTabs } from './category-tabs';
import { MenuCard } from './menu-card';

interface MenuSectionProps {
  locale: Locale;
  dict: Dictionary;
}

export function MenuSection({ locale, dict }: MenuSectionProps) {
  const [selectedCategory, setSelectedCategory] = React.useState<string>('all');

  const filteredItems = React.useMemo(() => {
    if (selectedCategory === 'all') return menuItems;
    return menuItems.filter((item) => item.categoryId === selectedCategory);
  }, [selectedCategory]);

  return (
    <section id="menu" className="py-20 bg-muted/30 scroll-mt-20">
      <div className="container">
        {/* Section Header */}
        <div className="text-center max-w-2xl mx-auto mb-12">
          <span className="text-xs sm:text-sm font-bold uppercase tracking-widest text-ocean-600 dark:text-ocean-400">
            {locale === 'ar' ? 'طازج من البحر' : 'Fresh From The Sea'}
          </span>
          <h2 className="text-3xl sm:text-4xl font-extrabold tracking-tight mt-2 text-foreground">
            {dict.menu.title}
          </h2>
          <p className="mt-3 text-muted-foreground text-sm sm:text-base">
            {dict.menu.subtitle}
          </p>

          {/* Category Tabs */}
          <div className="mt-8 flex justify-center">
            <CategoryTabs
              categories={menuCategories}
              selectedCategory={selectedCategory}
              onSelectCategory={setSelectedCategory}
              locale={locale}
              allLabel={dict.menu.all}
            />
          </div>
        </div>

        {/* Menu Grid */}
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 sm:gap-8">
          {filteredItems.map((item) => (
            <MenuCard
              key={item.id}
              item={item}
              locale={locale}
              dict={dict}
            />
          ))}
        </div>
      </div>
    </section>
  );
}
```

### 3.2 Category Filter Tabs (`components/menu/category-tabs.tsx`)
```tsx
'use client';

import * as React from 'react';
import { type Locale } from '@/lib/i18n/config';
import { type MenuCategory } from '@/data/menu-items';
import { cn } from '@/lib/utils';

interface CategoryTabsProps {
  categories: MenuCategory[];
  selectedCategory: string;
  onSelectCategory: (id: string) => void;
  locale: Locale;
  allLabel: string;
}

export function CategoryTabs({
  categories,
  selectedCategory,
  onSelectCategory,
  locale,
  allLabel,
}: CategoryTabsProps) {
  return (
    <div className="flex items-center gap-2 overflow-x-auto pb-2 max-w-full no-scrollbar">
      {/* 'All' Tab */}
      <button
        onClick={() => onSelectCategory('all')}
        className={cn(
          'px-5 py-2.5 rounded-full text-xs sm:text-sm font-bold whitespace-nowrap transition-all duration-200',
          selectedCategory === 'all'
            ? 'bg-ocean-600 text-white shadow-md shadow-ocean-600/30'
            : 'bg-card border border-border text-foreground hover:bg-muted'
        )}
      >
        {allLabel}
      </button>

      {/* Dynamic Categories */}
      {categories.map((category) => {
        const isSelected = selectedCategory === category.id;
        return (
          <button
            key={category.id}
            onClick={() => onSelectCategory(category.id)}
            className={cn(
              'px-5 py-2.5 rounded-full text-xs sm:text-sm font-bold whitespace-nowrap transition-all duration-200',
              isSelected
                ? 'bg-ocean-600 text-white shadow-md shadow-ocean-600/30'
                : 'bg-card border border-border text-foreground hover:bg-muted'
            )}
          >
            {category.name[locale]}
          </button>
        );
      })}
    </div>
  );
}
```

### 3.3 Menu Card Component (`components/menu/menu-card.tsx`)
```tsx
'use client';

import * as React from 'react';
import Image from 'next/image';
import { Plus, Check, Flame, Award } from 'lucide-react';
import { type Locale } from '@/lib/i18n/config';
import { type Dictionary } from '@/lib/i18n/get-dictionary';
import { type MenuItem } from '@/data/menu-items';
import { useCartStore } from '@/lib/store/cart-store';
import { cn } from '@/lib/utils';

interface MenuCardProps {
  item: MenuItem;
  locale: Locale;
  dict: Dictionary;
}

export function MenuCard({ item, locale, dict }: MenuCardProps) {
  const addItem = useCartStore((state) => state.addItem);
  const [justAdded, setJustAdded] = React.useState(false);

  const handleAddToCart = () => {
    addItem({
      id: item.id,
      name: item.name[locale],
      price: item.price,
      image: item.image,
      categoryName: item.categoryId,
    });

    setJustAdded(true);
    setTimeout(() => setJustAdded(false), 1200);
  };

  return (
    <div className="group rounded-2xl bg-card border border-border/70 overflow-hidden shadow-sm hover:shadow-xl transition-all duration-300 flex flex-col justify-between hover:-translate-y-1">
      {/* Product Image & Badges */}
      <div className="relative aspect-[4/3] w-full overflow-hidden bg-ocean-950/20">
        <Image
          src={item.image}
          alt={item.name[locale]}
          fill
          sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
          className="object-cover transition-transform duration-500 group-hover:scale-105"
        />

        {/* Dietary and Highlights Badges */}
        <div className="absolute top-3 inset-x-3 flex items-center justify-between pointer-events-none">
          <div className="flex flex-wrap gap-1.5">
            {item.isChefSpecial && (
              <span className="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-[11px] font-bold bg-amber-500 text-white shadow-md">
                <Award className="h-3 w-3" />
                {dict.menu.specialBadge}
              </span>
            )}
            {item.isSpicy && (
              <span className="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-[11px] font-bold bg-red-600 text-white shadow-md">
                <Flame className="h-3 w-3" />
                {dict.menu.spicyBadge}
              </span>
            )}
          </div>

          {/* Calorie or Prep Time Badge */}
          {item.calories && (
            <span className="px-2 py-0.5 rounded-md text-[10px] font-semibold bg-black/60 backdrop-blur-sm text-white">
              {item.calories} {locale === 'ar' ? 'سعرة' : 'kcal'}
            </span>
          )}
        </div>
      </div>

      {/* Card Content & Action */}
      <div className="p-5 flex-1 flex flex-col justify-between">
        <div>
          <div className="flex items-baseline justify-between gap-2 mb-2">
            <h3 className="font-bold text-lg text-foreground group-hover:text-ocean-600 transition-colors">
              {item.name[locale]}
            </h3>
            <span className="font-black text-ocean-600 dark:text-ocean-400 whitespace-nowrap text-lg">
              {item.price} <span className="text-xs font-semibold">{dict.menu.currency}</span>
            </span>
          </div>

          <p className="text-xs sm:text-sm text-muted-foreground line-clamp-2 leading-relaxed">
            {item.description[locale]}
          </p>
        </div>

        {/* Action Button */}
        <div className="mt-5 pt-4 border-t border-border/50 flex items-center justify-between">
          <span className="text-xs font-semibold text-muted-foreground uppercase tracking-wider">
            {item.weightGram ? `${item.weightGram}g` : ''}
          </span>

          <button
            onClick={handleAddToCart}
            className={cn(
              'inline-flex items-center gap-1.5 px-4 py-2 rounded-xl text-xs sm:text-sm font-bold transition-all duration-200',
              justAdded
                ? 'bg-emerald-600 text-white'
                : 'bg-ocean-600 hover:bg-ocean-700 text-white shadow-md shadow-ocean-600/20 active:scale-95'
            )}
            aria-label={`Add ${item.name[locale]} to cart`}
          >
            {justAdded ? (
              <>
                <Check className="h-4 w-4" />
                <span>{locale === 'ar' ? 'تمت الإضافة' : 'Added!'}</span>
              </>
            ) : (
              <>
                <Plus className="h-4 w-4" />
                <span>{dict.menu.addToCart}</span>
              </>
            )}
          </button>
        </div>
      </div>
    </div>
  );
}
```

---

## 4. Acceptance Criteria
- [ ] Category tabs dynamically filter the dish catalog with zero page reload.
- [ ] Mobile category bar has horizontal scrolling without layout overflow.
- [ ] Dishes display localized names, descriptions, prices, and dietary micro-badges.
- [ ] Clicking "Add to Cart" pushes the item to the Zustand store and renders optimistic confirmation ("Added!").
- [ ] Image assets render smoothly with Next.js image optimization.
