# Task 10: Localized Seafood Menu Data Schema & Catalog

## 1. Task Overview & Target Component/Page
Design the data architecture and TypeScript schemas for the seafood restaurant catalog. This includes bilingual category definitions (`en` & `ar`), item attributes, pricing, dietary flags (`isGlutenFree`, `isSpicy`, `isChefSpecial`), and an initial production-ready dataset representing authentic coastal seafood offerings.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── data/
│   └── menu-items.ts                # TypeScript catalog dataset & categories
├── types/
│   └── menu.ts                      # Core domain types for categories & dishes
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Domain Types Schema (`types/menu.ts`)
```typescript
export interface LocalizedString {
  en: string;
  ar: string;
}

export type CategorySlug = 
  | 'appetizers'
  | 'catch-of-the-day'
  | 'signature-platters'
  | 'soups-and-salads'
  | 'beverages';

export interface MenuCategory {
  id: CategorySlug;
  name: LocalizedString;
  description?: LocalizedString;
}

export interface MenuItem {
  id: string;
  categoryId: CategorySlug;
  name: LocalizedString;
  description: LocalizedString;
  price: number;
  image: string;
  weightGram?: number;
  calories?: number;
  isChefSpecial?: boolean;
  isSpicy?: boolean;
  isGlutenFree?: boolean;
  isAvailable: boolean;
}
```

### 3.2 Catalog Dataset (`data/menu-items.ts`)
```typescript
import { type MenuCategory, type MenuItem } from '@/types/menu';

export const menuCategories: MenuCategory[] = [
  {
    id: 'catch-of-the-day',
    name: {
      ar: 'صيد اليوم الطازج',
      en: 'Catch of the Day',
    },
  },
  {
    id: 'signature-platters',
    name: {
      ar: 'أطباقنا المميزة',
      en: 'Signature Platters',
    },
  },
  {
    id: 'appetizers',
    name: {
      ar: 'المقبلات البحرية',
      en: 'Seafood Appetizers',
    },
  },
  {
    id: 'soups-and-salads',
    name: {
      ar: 'الشوربات والسلطات',
      en: 'Soups & Salads',
    },
  },
  {
    id: 'beverages',
    name: {
      ar: 'المشروبات المنعشة',
      en: 'Beverages',
    },
  },
];

export const menuItems: MenuItem[] = [
  {
    id: 'lobster-thermidor',
    categoryId: 'signature-platters',
    name: {
      ar: 'استاكوزا ثيرميدور فاخرة',
      en: 'Royal Lobster Thermidor',
    },
    description: {
      ar: 'استاكوزا بحرية طازجة محشوة بصلصة الكريمة الغنية وجبن الغرويير والأعشاب العطرية، تقدم مع أرز الزعفران.',
      en: 'Fresh lobster tail baked with a rich cream and Gruyère cheese sauce, aromatic tarragon, served with saffron rice.',
    },
    price: 240,
    image: '/images/menu/lobster-thermidor.webp',
    weightGram: 650,
    calories: 780,
    isChefSpecial: true,
    isSpicy: false,
    isGlutenFree: false,
    isAvailable: true,
  },
  {
    id: 'grilled-sea-bass',
    categoryId: 'catch-of-the-day',
    name: {
      ar: 'سمك قاروص مشوي على الفحم',
      en: 'Charcoal Grilled Sea Bass',
    },
    description: {
      ar: 'سمك قاروص طازج متبل بالليمون وزيت الزيتون البكر وثوم المزارع والكمون الساحلي.',
      en: 'Fresh Mediterranean sea bass marinated with virgin olive oil, garlic, lemon zest, and coastal cumin.',
    },
    price: 135,
    image: '/images/menu/grilled-sea-bass.webp',
    weightGram: 500,
    calories: 460,
    isChefSpecial: true,
    isSpicy: false,
    isGlutenFree: true,
    isAvailable: true,
  },
  {
    id: 'crispy-calamari',
    categoryId: 'appetizers',
    name: {
      ar: 'كالاماري مقرمش بالليمون والفلفل',
      en: 'Crispy Salt & Pepper Calamari',
    },
    description: {
      ar: 'حلقات الحبار المقرمشة الذهبية مع صلصة التارتار المعدة منزلياً وشريحة ليمون طازجة.',
      en: 'Tender golden-fried squid rings dusted with sea salt, served with house-made tartar sauce and lemon wedges.',
    },
    price: 65,
    image: '/images/menu/calamari.webp',
    weightGram: 280,
    calories: 390,
    isChefSpecial: false,
    isSpicy: false,
    isGlutenFree: false,
    isAvailable: true,
  },
  {
    id: 'dynamite-shrimp',
    categoryId: 'appetizers',
    name: {
      ar: 'جمبري ديناميت الحار',
      en: 'Spicy Dynamite Shrimp',
    },
    description: {
      ar: 'جمبري مقرمش ومغمس بصلصة الداينامايت الكريمية الحارة مع رشة بصل أخضر وبذور السمسم.',
      en: 'Crispy batter-fried prawns tossed in our signature creamy spicy sriracha sauce and toasted sesame.',
    },
    price: 75,
    image: '/images/menu/dynamite-shrimp.webp',
    weightGram: 320,
    calories: 520,
    isChefSpecial: false,
    isSpicy: true,
    isGlutenFree: false,
    isAvailable: true,
  },
  {
    id: 'seafood-paella',
    categoryId: 'signature-platters',
    name: {
      ar: 'بايلا ثمار البحر الأندلسية',
      en: 'Coastal Seafood Paella',
    },
    description: {
      ar: 'أرز بومبا الإسباني بالزعفران، بلح البحر، الجمبري الجامبو، والكالاماري المطبوخ على نار هادئة.',
      en: 'Authentic saffron Bomba rice simmered with black mussels, jumbo tiger prawns, and tender baby squid.',
    },
    price: 185,
    image: '/images/menu/paella.webp',
    weightGram: 700,
    calories: 840,
    isChefSpecial: true,
    isSpicy: false,
    isGlutenFree: true,
    isAvailable: true,
  },
  {
    id: 'sayadieh-fish',
    categoryId: 'signature-platters',
    name: {
      ar: 'صيادية السمك الشرقية الأصيلة',
      en: 'Traditional Fish Sayadieh',
    },
    description: {
      ar: 'أرز الصيادية البني المعطر بالبصل المكرمل والبهارات الساحلية، مغطى بفيليه الهامور المحمر والمكسرات المحمصة.',
      en: 'Spiced aromatic caramelized onion rice topped with golden hamour fillet, toasted pine nuts, and tahini sauce.',
    },
    price: 120,
    image: '/images/menu/sayadieh.webp',
    weightGram: 550,
    calories: 690,
    isChefSpecial: false,
    isSpicy: false,
    isGlutenFree: false,
    isAvailable: true,
  },
  {
    id: 'bouillabaisse-soup',
    categoryId: 'soups-and-salads',
    name: {
      ar: 'شوربة بويابيس الفرنسية',
      en: 'Classic Bouillabaisse Soup',
    },
    description: {
      ar: 'حساء السمك الفرنسي الغني مع الشمر والطماطم وثمار البحر المشكلة، يقدم مع خبز التوست وصلصة الروي.',
      en: 'Rich Provencal seafood stew infused with saffron, fennel, assorted shellfish, and crusty baguette with rouille.',
    },
    price: 85,
    image: '/images/menu/bouillabaisse.webp',
    weightGram: 450,
    calories: 340,
    isChefSpecial: false,
    isSpicy: false,
    isGlutenFree: true,
    isAvailable: true,
  },
  {
    id: 'mint-lemonade',
    categoryId: 'beverages',
    name: {
      ar: 'عصير الليمون بالنعناع الطازج',
      en: 'Fresh Mint Lemonade',
    },
    description: {
      ar: 'عصير منعش من الليمون الطبيعي وأوراق النعناع الطازجة والثلج المجروش.',
      en: 'Refreshing blend of freshly squeezed lemons, organic garden mint, and crushed ice.',
    },
    price: 25,
    image: '/images/menu/mint-lemonade.webp',
    calories: 120,
    isChefSpecial: false,
    isSpicy: false,
    isGlutenFree: true,
    isAvailable: true,
  },
];
```

---

## 4. Acceptance Criteria
- [ ] TypeScript interfaces represent category entities and bilingual menu dish models.
- [ ] Rich mock catalog provides dishes for all categories (Catch of the day, Platters, Soups, Beverages, Appetizers).
- [ ] Schema accommodates dietary flags (`isChefSpecial`, `isSpicy`, `isGlutenFree`).
- [ ] Bilingual text structure supports simple access (`item.name[locale]`).
