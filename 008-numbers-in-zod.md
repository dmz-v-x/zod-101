## Numbers in Zod

### 1. Why Numbers Are Dangerous

Working with numbers in real apps can be tricky because:

- JSON transports numbers *and* strings (e.g., `"1"`)  
- HTML forms send number fields as strings  
- JavaScript has **NaN** and **Infinity**  
- Negative or out-of-range values can sneak in  

So when you expect a number, you must validate it explicitly — relying on TypeScript types alone won’t protect your app at runtime.

---

### 2. `z.number()` — Numbers in Zod

In Zod, the basic numeric schema is:

    const n = z.number();

This schema accepts **any finite numeric value**, and rejects things that aren’t valid numbers:

Valid:

- `10`
- `3.14`

Invalid (Zod will throw or return an error):

- `"10"`  — string (even if numeric)
- `NaN`   — not a valid number
- `Infinity` / `-Infinity` — Zod rejects them by default

Zod’s built-in number schema protects your program from invalid values and edge cases. 

---

### 3. `.int()` — Integer Only

Use `.int()` when you want only whole numbers:

    const ageSchema = z.number().int();

Valid:

    18 // ✅

Invalid:

    18.5 // ❌

This is especially useful for things like:
- age
- counts
- page numbers

Zod enforces this constraint at runtime. 

---

### 4. `.min()` and `.max()` — Range Limits

When numbers need a range, Zod lets you define boundaries:

    const ageSchema = z
      .number()
      .int()
      .min(0)   // age cannot be negative
      .max(120) // age realistically capped

Valid:

    25

Invalid:

    -1
    150

You define your bounds based on your business rules — Zod enforces them at runtime. 

---

### 5. Positivity Helpers

Zod includes helpers for common numeric patterns:

- `.positive()` → must be greater than 0  
- `.nonnegative()` → must be ≥ 0

Examples:

    z.number().positive();     // > 0
    z.number().nonnegative();  // ≥ 0

These helpers are surface-level shortcuts for common validations. 

---

### 6. When to Use What

| Use case      | Schema                                 |
|---------------|----------------------------------------|
| Age           | `.int().min(0)`                        |
| Price         | `.number().positive()`                 |
| Quantity      | `.number().int().positive()`           |
| Offset        | `.number().nonnegative()`              |

Think in terms of *intent* — Zod helps you express rules, not just types.

---

### 7. Real-World Examples

#### Pagination

    const paginationSchema = z.object({
      page: z.number().int().positive(),
      pageSize: z.number().int().min(1).max(100),
    });

This ensures valid pagination parameters every time.

---

#### Product Pricing

    const productSchema = z.object({
      name: z.string(),
      price: z.number().positive(),
    });

Your API won’t accept negative or zero pricing by mistake.

---

#### Quantity Update

    const updateQuantitySchema = z.object({
      quantity: z.number().int().nonnegative(),
    });

Perfect for stock or item counts.

---

### 8. Important Production Gotcha — Strings from Input

If you receive:

    { "page": "1" }

This will **fail** with `z.number()` because the input is a string, not a number.

This behavior is **good** — strict schemas block invalid data early.

Later you can use coercion (`z.coerce.number()`), but strict validation first is safer. 

---

### 9. NaN & Infinity (Hidden Dangers)

Zod rejects:

- `NaN`
- `Infinity`
- `-Infinity`

These values behave like numbers in JavaScript but lead to bugs in calculations. Zod blocks them by default, keeping your app safer. 

---

### 10. Custom Messages (Production-Ready)

You can provide your own error messages:

    const priceSchema = z
      .number()
      .positive("Price must be greater than zero");

This yields clear, user-friendly messages instead of generic ones.

---

### 11. Common Mistakes (Important)

❌ **Mistake 1 — Not restricting ranges**

    z.number(); // ❌ allows any number

Always ask:

> “What values are actually valid here?”

---

❌ **Mistake 2 — Using floats for money**

Zod validates numeric types — it does **not fix floating-point math issues**.

In production, handle money as:
- integers (e.g., cents)
- or use a decimal library

Zod will validate but not correct precision behavior.

---

❌ **Mistake 3 — Assuming frontend already validated**

Never trust just client-side checks.

Backend validation must re-validate data on every request.

---

### 12. Golden Rule

**Every number needs a rule.**

Numbers are simple, but wrong assumptions lead to bugs.  
Use Zod’s number constraints to express your intent clearly and enforce it safely at runtime.

---

### Final Takeaway

Zod’s `z.number()` plus its constraint methods (`.int()`, `.min()`, `.max()`, `.positive()`, `.nonnegative()`) help you validate numeric data cleanly and reliably.  

