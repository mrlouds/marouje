# Task 09: Zustand Cart Store Architecture & Hydration Safety

## 1. Task Overview & Target Component/Page
Architect and implement the global client-side Cart state using **Zustand**. The store includes item manipulation (`addItem`, `removeItem`, `updateQuantity`, `clearCart`), drawer visibility toggling, derived selectors (`totalItems`, `totalPrice`), and persistence via `localStorage` with a hydration-safe helper hook to prevent Next.js SSR hydration mismatches.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── lib/
│   └── store/
│       ├── cart-store.ts            # Zustand store definition with persist middleware
│       └── use-cart-hydration.ts    # Hydration helper hook to prevent client/server mismatch
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Cart Store Interfaces & Implementation (`lib/store/cart-store.ts`)
```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

export interface CartItem {
  id: string;
  name: string;
  price: number;
  image: string;
  quantity: number;
  categoryName?: string;
}

export interface CartState {
  items: CartItem[];
  isOpen: boolean;

  // Actions
  addItem: (item: Omit<CartItem, 'quantity'> & { quantity?: number }) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  openCart: () => void;
  closeCart: () => void;
  toggleCart: () => void;

  // Computed / Selectors
  totalItems: () => number;
  totalPrice: () => number;
}

export const useCartStore = create<CartState>()(
  persist(
    (set, get) => ({
      items: [],
      isOpen: false,

      addItem: (newItem) => {
        const { items } = get();
        const existingIndex = items.findIndex((item) => item.id === newItem.id);

        if (existingIndex > -1) {
          const updatedItems = [...items];
          updatedItems[existingIndex].quantity += newItem.quantity || 1;
          set({ items: updatedItems, isOpen: true });
        } else {
          set({
            items: [...items, { ...newItem, quantity: newItem.quantity || 1 }],
            isOpen: true,
          });
        }
      },

      removeItem: (id) => {
        set({
          items: get().items.filter((item) => item.id !== id),
        });
      },

      updateQuantity: (id, quantity) => {
        if (quantity <= 0) {
          get().removeItem(id);
          return;
        }

        set({
          items: get().items.map((item) =>
            item.id === id ? { ...item, quantity } : item
          ),
        });
      },

      clearCart: () => {
        set({ items: [] });
      },

      openCart: () => set({ isOpen: true }),
      closeCart: () => set({ isOpen: false }),
      toggleCart: () => set((state) => ({ isOpen: !state.isOpen })),

      totalItems: () => {
        return get().items.reduce((total, item) => total + item.quantity, 0);
      },

      totalPrice: () => {
        return get().items.reduce((total, item) => total + item.price * item.quantity, 0);
      },
    }),
    {
      name: 'seafood-restaurant-cart-storage',
      storage: createJSONStorage(() => localStorage),
      // Only persist the cart items, not transient UI states like isOpen
      partialize: (state) => ({ items: state.items }),
    }
  )
);
```

### 3.2 Hydration Safe Store Hook (`lib/store/use-cart-hydration.ts`)
Next.js SSR renders HTML before `localStorage` is accessible on the client. To eliminate hydration mismatches when rendering badges or item lists, we supply a custom hook:

```typescript
'use client';

import { useState, useEffect } from 'react';

export function useCartHydration<T, F>(
  store: (callback: (state: T) => unknown) => unknown,
  callback: (state: T) => F
) {
  const result = store(callback) as F;
  const [data, setData] = useState<F>();

  useEffect(() => {
    setData(result);
  }, [result]);

  return data;
}
```

---

## 4. Acceptance Criteria
- [ ] Zustand store tracks `items`, `quantity`, `totalItems`, and `totalPrice`.
- [ ] `persist` middleware synchronizes cart items to `localStorage`.
- [ ] Drawer state (`isOpen`) remains ephemeral and does not persist across sessions.
- [ ] Zero Next.js hydration warning or mismatch when accessing persistent cart state.
- [ ] Safe decrementing: Quantity reduction to 0 deletes the item cleanly.
