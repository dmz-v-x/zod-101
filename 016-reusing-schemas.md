## Reusing Schemas

### Step 1 — The Duplication Problem

You start with a simple base schema:

	import { z } from "zod";

	const baseUserSchema = z.object({
	  email: z.string().email(),
	  password: z.string().min(8),
	});

Later in the application lifecycle, you need multiple variants:

- Admin user
- Public (API response) user
- Database user
- Internal service user

❌ A common mistake is **copy-pasting schemas** and modifying them.

This is where bugs begin.

---

### Step 2 — Why Copy-Paste Is Dangerous

Example of copy-paste:

	const adminSchema = z.object({
	  email: z.string().email(),
	  password: z.string().min(8),
	  role: z.literal("admin"),
	});

The problem:

- Password rule changes → must update everywhere
- Someone forgets → schemas drift
- Bugs appear silently
- Security issues creep in

Zod is designed to **prevent this exact problem**.

---

### Step 3 — `.extend()`

Instead of copying, **derive**.

	const adminUserSchema = baseUserSchema.extend({
	  role: z.literal("admin"),
	});

What this gives you:

- Base schema reused
- Admin schema builds on top
- Single source of truth
- Changes propagate safely

This is the **intended Zod pattern**.

---

### Step 4 — Important Correction: Removing Fields Properly

❌ This is **NOT correct**:

	const publicUserSchema = baseUserSchema.extend({
	  password: z.undefined(),
	});

Why this is wrong:

- `z.undefined()` means the key **must exist and be undefined**
- It does NOT mean “the field must not exist”
- JSON APIs usually omit sensitive fields entirely

This would reject:

	{ "email": "a@b.com" }

And accept:

	{ "email": "a@b.com", "password": undefined }

That is not what you want.

---

### Step 5 — Correct Way to Remove Fields: `.omit()`

Zod provides `.omit()` for this exact use case.

	const publicUserSchema = baseUserSchema.omit({
	  password: true,
	});

What this does:

- Removes `password` entirely
- Runtime validation matches intent
- TypeScript type excludes the field
- Prevents accidental leaks

---

### Step 6 — Real-World User Lifecycle Example

#### Step 6.1 — User Input (Signup)

	const userInputSchema = z.object({
	  email: z.string().email(),
	  password: z.string().min(8),
	});

---

#### Step 6.2 — Database User

	const dbUserSchema = userInputSchema.extend({
	  id: z.string().uuid(),
	  createdAt: z.date(),
	});

Adds persistence-only fields while preserving validation rules.

---

#### Step 6.3 — API Response User

	const apiUserSchema = dbUserSchema.omit({
	  password: true,
	});

This ensures:

- Password is never exposed
- Schema enforces safety
- No accidental leaks in refactors

---

### Step 7 — TypeScript Inference

	type ApiUser = z.infer<typeof apiUserSchema>;

Result:

- `email` → present
- `id` → present
- `createdAt` → present
- `password` → does not exist

Runtime rules and TypeScript types stay perfectly aligned.

---

### Step 8 — `.extend()` vs `.merge()`

- `.extend()`  
  Derive from one base object  
  Add or override fields  

- `.merge()`  
  Combine two object schemas  

Use `.extend()` for **hierarchies**  
Use `.merge()` for **composition**

---

### Step 9 — Common Production Mistakes

#### ❌ Mistake 1 — Redefining Base Schemas

Never redefine shared fields.

Always derive.

---

#### ❌ Mistake 2 — Overriding With Incompatible Meaning

	baseUserSchema.extend({
	  age: z.string(), // ❌ breaks domain logic
	});

Overrides should be **intentional and meaningful**.

---

#### ❌ Mistake 3 — Assuming `.extend()` Adds Strictness

- `.extend()` **inherits strictness**
- Only if the base schema was strict
- It does NOT automatically lock schemas down

Be explicit when strictness matters.

---

### Golden Rule

Write schemas like classes:

	Base → Derived

Reuse, extend, and omit —  
**never copy-paste schemas**.
