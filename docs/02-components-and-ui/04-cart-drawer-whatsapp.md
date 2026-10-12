# Task 07: Slide-Over Cart Drawer & WhatsApp Order Generator

## 1. Task Overview & Target Component/Page
Construct an accessible, responsive Slide-Over Cart Drawer connected to the client-side Zustand store. It supports dynamic item quantity adjustments, item removal, customer culinary notes (e.g. grilling style or spices), total calculation, and direct generation of a pre-formatted bilingual WhatsApp order link that immediately opens WhatsApp with the customer's full cart contents.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── components/
│   └── cart/
│       ├── cart-drawer.tsx          # Main slide-over drawer modal
│       ├── cart-item-row.tsx        # Individual cart item with (+ / -) controls
│       └── whatsapp-order-btn.tsx   # Formatter and CTA button for WhatsApp
├── lib/
│   └── whatsapp/
│       └── generate-order-link.ts   # Order URL generator utility
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 WhatsApp Order Message Generator (`lib/whatsapp/generate-order-link.ts`)
This utility takes cart items, customer notes, and current locale, formatting them into an elegant WhatsApp message with URL encoding.

```typescript
import { type CartItem } from '@/lib/store/cart-store';
import { type Locale } from '@/lib/i18n/config';

interface OrderPayload {
  items: CartItem[];
  totalPrice: number;
  notes?: string;
  locale: Locale;
  restaurantWhatsAppNumber: string; // e.g. "966500000000"
}

export function generateWhatsAppOrderLink({
  items,
  totalPrice,
  notes,
  locale,
  restaurantWhatsAppNumber,
}: OrderPayload): string {
  const isAr = locale === 'ar';
  const currency = isAr ? 'ر.س' : 'SAR';

  // Construct message header
  const header = isAr
    ? '🌊 *طلب جديد من مطعم مروج البحر*'
    : '🌊 *New Order - Marouj Al Bahr Seafood*';

  const separator = '--------------------------------';

  // Format line items
  const itemsList = items
    .map((item, index) => {
      const itemSubtotal = item.price * item.quantity;
      return `${index + 1}. *${item.name}*\n   ${item.quantity}x @ ${item.price} ${currency} = *${itemSubtotal} ${currency}*`;
    })
    .join('\n\n');

  // Format total
  const totalLine = isAr
    ? `💰 *المجموع الإجمالي:* ${totalPrice} ${currency}`
    : `💰 *Grand Total:* ${totalPrice} ${currency}`;

  // Format notes if provided
  const notesLine = notes
    ? isAr
      ? `📝 *ملاحظات خاصة:* ${notes}`
      : `📝 *Special Instructions:* ${notes}`
    : '';

  const footer = isAr
    ? 'يرجى تأكيد الطلب وتحديد وقت الاستلام أو موقع التوصيل. شكراً لكم!'
    : 'Please confirm my order and advise estimated preparation/delivery time. Thank you!';

  const rawMessage = [header, separator, itemsList, separator, totalLine, notesLine, separator, footer]
    .filter(Boolean)
    .join('\n');

  const encodedMessage = encodeURIComponent(rawMessage);

  return `https://wa.me/${restaurantWhatsAppNumber}?text=${encodedMessage}`;
}
```

### 3.2 Slide-Over Cart Drawer (`components/cart/cart-drawer.tsx`)
```tsx
'use client';

import * as React from 'react';
import Image from 'next/image';
import { X, Trash2, Plus, Minus, ShoppingBag, MessageSquareQuote } from 'lucide-react';
import { useCartStore } from '@/lib/store/cart-store';
import { type Locale } from '@/lib/i18n/config';
import { type Dictionary } from '@/lib/i18n/get-dictionary';
import { generateWhatsAppOrderLink } from '@/lib/whatsapp/generate-order-link';
import { cn } from '@/lib/utils';

interface CartDrawerProps {
  locale: Locale;
  dict: Dictionary;
}

const RESTAURANT_WHATSAPP = process.env.NEXT_PUBLIC_WHATSAPP_NUMBER || '966500000000';

export function CartDrawer({ locale, dict }: CartDrawerProps) {
  const { items, isOpen, closeCart, updateQuantity, removeItem, clearCart, totalPrice } =
    useCartStore();

  const [customerNotes, setCustomerNotes] = React.useState('');

  // Lock body scroll when drawer is open
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

  const total = totalPrice();
  const whatsappUrl = generateWhatsAppOrderLink({
    items,
    totalPrice: total,
    notes: customerNotes,
    locale,
    restaurantWhatsAppNumber: RESTAURANT_WHATSAPP,
  });

  return (
    <div className="fixed inset-0 z-50 overflow-hidden">
      {/* Backdrop */}
      <div
        className="fixed inset-0 bg-black/60 backdrop-blur-sm transition-opacity duration-300"
        onClick={closeCart}
      />

      {/* Slide-over Content Container */}
      <div
        className={cn(
          'fixed inset-y-0 max-w-full flex',
          locale === 'ar' ? 'left-0' : 'right-0'
        )}
      >
        <div className="w-screen max-w-md bg-card shadow-2xl flex flex-col justify-between border-s border-border">
          {/* Drawer Header */}
          <div className="p-6 border-b border-border flex items-center justify-between">
            <div className="flex items-center gap-2">
              <ShoppingBag className="h-5 w-5 text-ocean-600 dark:text-ocean-400" />
              <h2 className="text-lg font-bold text-foreground">{dict.cart.title}</h2>
              {items.length > 0 && (
                <span className="px-2 py-0.5 rounded-full text-xs font-semibold bg-ocean-100 text-ocean-700 dark:bg-ocean-900/60 dark:text-ocean-300">
                  {items.length}
                </span>
              )}
            </div>

            <button
              onClick={closeCart}
              className="p-2 rounded-full hover:bg-muted text-muted-foreground hover:text-foreground transition-colors"
              aria-label="Close cart"
            >
              <X className="h-5 w-5" />
            </button>
          </div>

          {/* Cart Item List */}
          <div className="flex-1 overflow-y-auto p-6 space-y-4">
            {items.length === 0 ? (
              <div className="h-full flex flex-col items-center justify-center text-center py-12">
                <div className="h-20 w-20 rounded-full bg-ocean-50 dark:bg-ocean-950 flex items-center justify-center text-ocean-500 mb-4">
                  <ShoppingBag className="h-10 w-10 stroke-[1.5]" />
                </div>
                <h3 className="text-base font-bold text-foreground">{dict.cart.empty}</h3>
                <p className="text-xs sm:text-sm text-muted-foreground max-w-xs mt-1">
                  {dict.cart.emptyDesc}
                </p>
              </div>
            ) : (
              items.map((item) => (
                <div
                  key={item.id}
                  className="flex items-center gap-4 p-3 rounded-xl border border-border/60 bg-muted/20"
                >
                  <div className="relative h-16 w-16 rounded-lg overflow-hidden shrink-0 bg-ocean-950/20">
                    <Image
                      src={item.image}
                      alt={item.name}
                      fill
                      className="object-cover"
                    />
                  </div>

                  <div className="flex-1 min-w-0">
                    <h4 className="font-bold text-sm text-foreground truncate">{item.name}</h4>
                    <p className="text-xs font-semibold text-ocean-600 dark:text-ocean-400">
                      {item.price} {dict.menu.currency}
                    </p>

                    {/* Quantity controls */}
                    <div className="flex items-center gap-2 mt-2">
                      <button
                        onClick={() => updateQuantity(item.id, item.quantity - 1)}
                        className="p-1 rounded-md border border-border hover:bg-muted text-foreground transition-colors"
                        aria-label="Decrease quantity"
                      >
                        <Minus className="h-3 w-3" />
                      </button>
                      <span className="text-xs font-bold px-1.5">{item.quantity}</span>
                      <button
                        onClick={() => updateQuantity(item.id, item.quantity + 1)}
                        className="p-1 rounded-md border border-border hover:bg-muted text-foreground transition-colors"
                        aria-label="Increase quantity"
                      >
                        <Plus className="h-3 w-3" />
                      </button>
                    </div>
                  </div>

                  <button
                    onClick={() => removeItem(item.id)}
                    className="p-2 text-muted-foreground hover:text-red-500 transition-colors"
                    aria-label={`Remove ${item.name}`}
                  >
                    <Trash2 className="h-4 w-4" />
                  </button>
                </div>
              ))
            )}
          </div>

          {/* Drawer Footer & Checkout Action */}
          {items.length > 0 && (
            <div className="p-6 border-t border-border bg-card space-y-4">
              {/* Optional Kitchen Notes */}
              <div>
                <label className="text-xs font-bold text-foreground mb-1 block">
                  {dict.cart.specialInstructions}
                </label>
                <textarea
                  value={customerNotes}
                  onChange={(e) => setCustomerNotes(e.target.value)}
                  placeholder={
                    locale === 'ar'
                      ? 'مثال: تسوية الشواء، بدون بهارات حارة...'
                      : 'e.g. Well done grill, low salt, extra lemon...'
                  }
                  rows={2}
                  className="w-full text-xs p-2.5 rounded-lg border border-border bg-background focus:outline-none focus:ring-2 focus:ring-ocean-500 resize-none"
                />
              </div>

              {/* Total Calculation */}
              <div className="flex items-center justify-between text-base font-bold">
                <span>{dict.cart.total}</span>
                <span className="text-xl font-black text-ocean-600 dark:text-ocean-400">
                  {total} {dict.menu.currency}
                </span>
              </div>

              {/* Order via WhatsApp Button */}
              <a
                href={whatsappUrl}
                target="_blank"
                rel="noopener noreferrer"
                className="w-full inline-flex items-center justify-center gap-2.5 py-3.5 px-4 rounded-xl font-bold text-white bg-emerald-600 hover:bg-emerald-700 shadow-lg shadow-emerald-600/25 transition-all duration-200 active:scale-[0.99]"
              >
                <MessageSquareQuote className="h-5 w-5" />
                <span>{dict.cart.sendWhatsApp}</span>
              </a>

              {/* Clear Cart Button */}
              <button
                onClick={clearCart}
                className="w-full text-center text-xs text-muted-foreground hover:text-red-500 transition-colors pt-1"
              >
                {dict.cart.clearCart}
              </button>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}
```

---

## 4. Acceptance Criteria
- [ ] Cart drawer slides in smoothly from the correct side based on RTL (left) and LTR (right).
- [ ] Quantity counter allows incrementing, decrementing, and removing items.
- [ ] Subtotal and Grand Total calculate accurately.
- [ ] Kitchen notes field preserves custom user inputs.
- [ ] WhatsApp CTA builds a well-formed, URL-encoded string that launches WhatsApp Web or app with complete order details.
- [ ] Clean empty state displays when no items are present.
