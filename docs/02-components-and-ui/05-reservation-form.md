# Task 08: Table Reservation Form with Next.js Server Actions

## 1. Task Overview & Target Component/Page
Build a robust, fully validated Table Reservation Section. The client form features interactive seating preference selection (Indoor Luxury, Terrace, Seaside View), guest counter, date/time pickers, client-side input validation, and secure handling via Next.js 14+ Server Actions (`reserveTableAction`) with real-time success/error notifications.

---

## 2. Next.js File Structure & Location
```
seafood-restaurant/
├── components/
│   └── reservation/
│       ├── reservation-section.tsx    # Section wrapper
│       └── reservation-form.tsx       # Client form component with useFormState/useFormStatus
├── app/
│   └── actions/
│       └── reservation-action.ts      # Server Action with Zod schema validation
├── lib/
│   └── validation/
│       └── reservation-schema.ts      # Shared Zod validation schema
```

---

## 3. Detailed Specifications & Code Architecture Guidelines

### 3.1 Zod Validation Schema (`lib/validation/reservation-schema.ts`)
```typescript
import { z } from 'zod';

export const reservationSchema = z.object({
  fullName: z.string().min(3, { message: 'Full name must be at least 3 characters long' }),
  phone: z.string().regex(/^([+]?[\d\s-]{8,15})$/, { message: 'Invalid phone number format' }),
  date: z.string().min(1, { message: 'Please select a reservation date' }),
  time: z.string().min(1, { message: 'Please select a reservation time' }),
  guests: z.coerce.number().min(1).max(20, { message: 'Guest count must be between 1 and 20' }),
  seatingArea: z.enum(['indoor', 'terrace', 'seaView'], {
    errorMap: () => ({ message: 'Please select a valid seating area' }),
  }),
  specialRequests: z.string().max(300).optional(),
});

export type ReservationFormData = z.infer<typeof reservationSchema>;

export interface ActionResponse {
  success: boolean;
  message: string;
  errors?: Record<string, string[]>;
}
```

### 3.2 Server Action (`app/actions/reservation-action.ts`)
```typescript
'use server';

import { reservationSchema, type ActionResponse } from '@/lib/validation/reservation-schema';

export async function createReservationAction(
  prevState: ActionResponse | null,
  formData: FormData
): Promise<ActionResponse> {
  // Extract raw form entries
  const rawData = {
    fullName: formData.get('fullName'),
    phone: formData.get('phone'),
    date: formData.get('date'),
    time: formData.get('time'),
    guests: formData.get('guests'),
    seatingArea: formData.get('seatingArea'),
    specialRequests: formData.get('specialRequests'),
  };

  // Validate against Zod schema
  const validationResult = reservationSchema.safeParse(rawData);

  if (!validationResult.success) {
    const errorMap = validationResult.error.flatten().fieldErrors;
    return {
      success: false,
      message: 'Validation failed. Please review your entries.',
      errors: errorMap,
    };
  }

  const validData = validationResult.data;

  try {
    // Simulate database insertion or CRM webhook (e.g. Supabase, Prisma, or Resend Email notification)
    await new Promise((resolve) => setTimeout(resolve, 1000));

    console.log('Successfully recorded table reservation:', validData);

    return {
      success: true,
      message: 'Reservation booked successfully! Our team will contact you shortly.',
    };
  } catch (error) {
    console.error('Reservation processing error:', error);
    return {
      success: false,
      message: 'An unexpected error occurred while booking. Please try again.',
    };
  }
}
```

### 3.3 Client Reservation Form (`components/reservation/reservation-form.tsx`)
```tsx
'use client';

import * as React from 'react';
import { useFormState, useFormStatus } from 'react-dom';
import { Calendar, Users, Clock, CheckCircle2, AlertCircle, Sparkles } from 'lucide-react';
import { createReservationAction } from '@/app/actions/reservation-action';
import { type Locale } from '@/lib/i18n/config';
import { type Dictionary } from '@/lib/i18n/get-dictionary';
import { cn } from '@/lib/utils';

function SubmitButton({ label, submittingLabel }: { label: string; submittingLabel: string }) {
  const { pending } = useFormStatus();

  return (
    <button
      type="submit"
      disabled={pending}
      className={cn(
        'w-full inline-flex items-center justify-center gap-2 py-4 px-6 rounded-xl font-bold text-white',
        'bg-ocean-600 hover:bg-ocean-700 shadow-lg shadow-ocean-600/30 transition-all duration-200',
        'disabled:opacity-60 disabled:cursor-not-allowed hover:-translate-y-0.5'
      )}
    >
      <Sparkles className="h-5 w-5" />
      <span>{pending ? submittingLabel : label}</span>
    </button>
  );
}

export function ReservationForm({ locale, dict }: { locale: Locale; dict: Dictionary }) {
  const [state, formAction] = useFormState(createReservationAction, null);
  const formRef = React.useRef<HTMLFormElement>(null);

  React.useEffect(() => {
    if (state?.success) {
      formRef.current?.reset();
    }
  }, [state]);

  const seatingOptions = [
    { id: 'seaView', label: dict.reservation.seaView },
    { id: 'terrace', label: dict.reservation.terrace },
    { id: 'indoor', label: dict.reservation.indoor },
  ];

  return (
    <form ref={formRef} action={formAction} className="space-y-6">
      {/* Toast Feedback Banner */}
      {state?.message && (
        <div
          className={cn(
            'p-4 rounded-xl flex items-start gap-3 text-sm font-semibold border',
            state.success
              ? 'bg-emerald-50 dark:bg-emerald-950/40 text-emerald-800 dark:text-emerald-300 border-emerald-300'
              : 'bg-red-50 dark:bg-red-950/40 text-red-800 dark:text-red-300 border-red-300'
          )}
        >
          {state.success ? (
            <CheckCircle2 className="h-5 w-5 shrink-0 text-emerald-600" />
          ) : (
            <AlertCircle className="h-5 w-5 shrink-0 text-red-600" />
          )}
          <p>{state.success ? dict.reservation.successMessage : state.message}</p>
        </div>
      )}

      {/* Guest Name & Phone */}
      <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <div>
          <label className="block text-xs font-bold text-foreground mb-1.5">
            {dict.reservation.nameLabel} *
          </label>
          <input
            type="text"
            name="fullName"
            required
            className="w-full px-4 py-3 rounded-xl border border-border bg-card text-foreground focus:ring-2 focus:ring-ocean-500 focus:outline-none text-sm"
          />
          {state?.errors?.fullName && (
            <span className="text-xs text-red-500 mt-1 block">{state.errors.fullName[0]}</span>
          )}
        </div>

        <div>
          <label className="block text-xs font-bold text-foreground mb-1.5">
            {dict.reservation.phoneLabel} *
          </label>
          <input
            type="tel"
            name="phone"
            placeholder="+966 50 000 0000"
            required
            className="w-full px-4 py-3 rounded-xl border border-border bg-card text-foreground focus:ring-2 focus:ring-ocean-500 focus:outline-none text-sm"
          />
          {state?.errors?.phone && (
            <span className="text-xs text-red-500 mt-1 block">{state.errors.phone[0]}</span>
          )}
        </div>
      </div>

      {/* Date, Time & Number of Guests */}
      <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
        <div>
          <label className="block text-xs font-bold text-foreground mb-1.5">
            {dict.reservation.dateLabel} *
          </label>
          <input
            type="date"
            name="date"
            required
            className="w-full px-4 py-3 rounded-xl border border-border bg-card text-foreground focus:ring-2 focus:ring-ocean-500 focus:outline-none text-sm"
          />
        </div>

        <div>
          <label className="block text-xs font-bold text-foreground mb-1.5">
            {dict.reservation.timeLabel} *
          </label>
          <input
            type="time"
            name="time"
            required
            className="w-full px-4 py-3 rounded-xl border border-border bg-card text-foreground focus:ring-2 focus:ring-ocean-500 focus:outline-none text-sm"
          />
        </div>

        <div>
          <label className="block text-xs font-bold text-foreground mb-1.5">
            {dict.reservation.guestsLabel} *
          </label>
          <input
            type="number"
            name="guests"
            defaultValue={2}
            min={1}
            max={20}
            required
            className="w-full px-4 py-3 rounded-xl border border-border bg-card text-foreground focus:ring-2 focus:ring-ocean-500 focus:outline-none text-sm"
          />
        </div>
      </div>

      {/* Seating Preference Radios */}
      <div>
        <label className="block text-xs font-bold text-foreground mb-2">
          {dict.reservation.seatingLabel} *
        </label>
        <div className="grid grid-cols-1 sm:grid-cols-3 gap-3">
          {seatingOptions.map((opt, idx) => (
            <label
              key={opt.id}
              className="flex items-center gap-3 p-3.5 rounded-xl border border-border bg-card hover:border-ocean-500 cursor-pointer transition-colors"
            >
              <input
                type="radio"
                name="seatingArea"
                value={opt.id}
                defaultChecked={idx === 0}
                className="text-ocean-600 focus:ring-ocean-500 h-4 w-4"
              />
              <span className="text-xs font-semibold text-foreground">{opt.label}</span>
            </label>
          ))}
        </div>
      </div>

      {/* Special Notes / Dietary Requests */}
      <div>
        <label className="block text-xs font-bold text-foreground mb-1.5">
          {dict.reservation.notesLabel}
        </label>
        <textarea
          name="specialRequests"
          rows={3}
          className="w-full px-4 py-3 rounded-xl border border-border bg-card text-foreground focus:ring-2 focus:ring-ocean-500 focus:outline-none text-sm resize-none"
        />
      </div>

      {/* Action Button */}
      <SubmitButton
        label={dict.reservation.submitBtn}
        submittingLabel={dict.reservation.submitting}
      />
    </form>
  );
}
```

---

## 4. Acceptance Criteria
- [ ] Table reservation form built using React Server Actions with `useFormStatus` and `useFormState`.
- [ ] Zod schema validates names, phone numbers, guest counts, and dates.
- [ ] Form displays inline field validation errors and toast alert feedback.
- [ ] Successful submission resets form state.
- [ ] Layout is responsive across all devices and adapts direction for Arabic and English.
