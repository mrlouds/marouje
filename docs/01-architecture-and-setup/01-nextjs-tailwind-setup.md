# Task 01: Next.js 14+ App Router & Tailwind CSS Base Setup

## 1. Task Overview & Target Component/Page
Set up the core foundation for a high-performance, responsive bilingual Seafood Restaurant application using Next.js 14+ (App Router), TypeScript, and Tailwind CSS. This task establishes the directory topology, path aliases, base Tailwind styling with semantic color tokens, and Google Fonts integration (`Tajawal` for Arabic and `Inter` for English).

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── app/
│   ├── [locale]/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── favicon.ico
│   └── globals.css
├── public/
│   ├── images/
│   │   ├── hero/
│   │   └── menu/
│   └── icons/
├── styles/
│   └── fonts.ts
├── .env.example
├── .env.local
├── next.config.mjs
├── package.json
├── postcss.config.mjs
├── tailwind.config.ts
└── tsconfig.json
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Dependencies Installation
```bash
npm install next@latest react@latest react-dom@latest lucide-react clsx tailwind-merge next-themes zustand zod
npm install -D typescript @types/node @types/react @types/react-dom postcss tailwindcss autoprefixer
```

### 3.2 Font Configuration (`styles/fonts.ts`)
We use `next/font/google` with CSS variable definitions to swap typography seamlessly based on the active locale.

```typescript
import { Inter, Tajawal } from 'next/font/google';

export const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
  weight: ['400', '500', '600', '700', '800'],
});

export const tajawal = Tajawal({
  subsets: ['arabic'],
  display: 'swap',
  variable: '--font-tajawal',
  weight: ['400', '500', '700', '800'],
});
```

### 3.3 Tailwind CSS Configuration (`tailwind.config.ts`)
The color palette represents a premium seafood dining atmosphere: Deep Ocean Blues, Coastal Turquoise, Sea Foam accents, Warm Gold sands, and Pearl whites.

```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: ['class'],
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './src/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    container: {
      center: true,
      padding: '1.5rem',
      screens: {
        '2xl': '1360px',
      },
    },
    extend: {
      fontFamily: {
        sans: ['var(--font-sans)', 'sans-serif'],
        inter: ['var(--font-inter)', 'sans-serif'],
        tajawal: ['var(--font-tajawal)', 'sans-serif'],
      },
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        ocean: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0284c7',
          600: '#0369a1',
          700: '#075985',
          800: '#0c4a6e',
          900: '#082f49',
          950: '#031926', // Deep Abyss
        },
        coral: {
          400: '#fb7185',
          500: '#f43f5e',
          600: '#e11d48',
        },
        sand: {
          100: '#fef3c7',
          200: '#fde68a',
          400: '#fbbf24',
          500: '#d97706',
        },
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [],
};

export default config;
```

### 3.4 Global Styles & Design Tokens (`app/globals.css`)
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 204 33% 98%;
    --foreground: 205 60% 12%;

    --card: 0 0% 100%;
    --card-foreground: 205 60% 12%;

    --primary: 201 96% 32%;
    --primary-foreground: 210 40% 98%;

    --secondary: 199 89% 48%;
    --secondary-foreground: 205 60% 12%;

    --muted: 204 20% 92%;
    --muted-foreground: 205 20% 45%;

    --accent: 38 92% 50%;
    --accent-foreground: 205 60% 12%;

    --border: 204 20% 88%;
    --input: 204 20% 88%;
    --ring: 201 96% 32%;

    --radius: 0.75rem;
  }

  .dark {
    --background: 205 78% 6%;
    --foreground: 204 20% 94%;

    --card: 205 70% 9%;
    --card-foreground: 204 20% 94%;

    --primary: 199 89% 48%;
    --primary-foreground: 205 78% 6%;

    --secondary: 201 96% 32%;
    --secondary-foreground: 204 20% 94%;

    --muted: 205 40% 16%;
    --muted-foreground: 204 15% 65%;

    --accent: 38 92% 50%;
    --accent-foreground: 205 78% 6%;

    --border: 205 40% 18%;
    --input: 205 40% 18%;
    --ring: 199 89% 48%;
  }
}

@layer utilities {
  .glass-panel {
    @apply backdrop-blur-md bg-white/75 dark:bg-ocean-950/75 border border-white/20 dark:border-ocean-800/40;
  }
}
```

### 3.5 Class Name Helper (`lib/utils.ts`)
```typescript
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

---

## 4. Acceptance Criteria
- [ ] Next.js 14+ initialized with TypeScript, ESLint, and App Router structure.
- [ ] Tailwind CSS configured with custom oceanic color scheme and semantic tokens.
- [ ] Google Fonts (`Tajawal` and `Inter`) configured with Next.js font optimization and CSS variables.
- [ ] `lib/utils.ts` created with `cn()` utility for conditional class merging.
- [ ] Application compiles successfully with `npm run build` without font hydration issues.
