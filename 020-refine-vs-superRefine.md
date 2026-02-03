## `refine` vs `superRefine`

### Step 1 — Why Refinements Exist

So far, the Zod rules you’ve learned were mostly:

- Local (one field at a time)
- Independent
- Structural

But real business rules are often:

- Cross-field
- Conditional
- Contextual

Example:

	password === confirmPassword

This **cannot** be expressed with `z.string()` or field-level rules alone.

This is exactly why **refinements** exist in Zod.

---

### Step 2 — `.refine()`

**What `.refine()` means**

> “Validate this value using a custom boolean condition.”

Basic example (single value):

	const evenNumberSchema = z
	  .number()
	  .refine(n => n % 2 === 0, {
	    message: "Number must be even",
	  });

Valid:

	2   // ✅

Invalid:

	3   // ❌

Here:

- The value is checked at runtime
- Returning `false` fails validation
- A custom error message is attached

---

### Step 3 — `.refine()` on Objects 

**Password confirmation example**

	const signupSchema = z
	  .object({
	    password: z.string().min(8),
	    confirmPassword: z.string(),
	  })
	  .refine(
	    data => data.password === data.confirmPassword,
	    {
	      message: "Passwords do not match",
	      path: ["confirmPassword"],
	    }
	  );

Key points:

- `data` is the **entire object**
- `path` controls **where the error appears**
- This is a **cross-field rule**

This pattern is extremely common in real applications.

---

### Step 4 — Real-World Example: Date Range Validation

	const dateRangeSchema = z
	  .object({
	    startDate: z.date(),
	    endDate: z.date(),
	  })
	  .refine(
	    data => data.endDate > data.startDate,
	    {
	      message: "End date must be after start date",
	      path: ["endDate"],
	    }
	  );

This enforces:

- Both dates must exist
- End date must come after start date

Very common in booking systems, reports, and analytics APIs.

---

### Step 5 — Limitations of `.refine()`

`.refine()` has limitations:

- It can emit **only one issue per refinement**
- It is not ideal for complex logic
- It cannot easily report multiple errors at once

For advanced validation logic → **use `.superRefine()`**.

---

### Step 6 — `.superRefine()` (Advanced & Powerful)

**What `.superRefine()` means**

> “Take full control over validation and manually add issues.”

It gives you a `ctx` object that lets you push **multiple validation errors** with full control.

---

### Step 7 — `.superRefine()` Basic Example

	const schema = z.object({
	  password: z.string(),
	  confirmPassword: z.string(),
	}).superRefine((data, ctx) => {
	  if (data.password !== data.confirmPassword) {
	    ctx.addIssue({
	      code: z.ZodIssueCode.custom,
	      message: "Passwords do not match",
	      path: ["confirmPassword"],
	    });
	  }
	});

Key differences from `.refine()`:

- You manually add issues
- You can add **multiple issues**
- You control error placement precisely

---

### Step 8 — Real Production Example: Conditional Rules

**Scenario**

- If role = `admin` → `adminCode` is required
- If role = `user` → `adminCode` must NOT exist

    	const userSchema = z.object({
    	  role: z.enum(["user", "admin"]),
    	  adminCode: z.string().optional(),
    	}).superRefine((data, ctx) => {
    	  if (data.role === "admin" && !data.adminCode) {
    	    ctx.addIssue({
    	      code: z.ZodIssueCode.custom,
    	      message: "adminCode is required for admins",
    	      path: ["adminCode"],
    	    });
    	  }

	  if (data.role === "user" && data.adminCode) {
	    ctx.addIssue({
	      code: z.ZodIssueCode.custom,
	      message: "adminCode is not allowed for users",
	      path: ["adminCode"],
	    });
	  }
	});

This **cannot** be modeled with basic schemas alone.

---

### Step 9 — PATCH API Rule: “At Least One Field Required”

Very common in production PATCH endpoints.

	const updateProfileSchema = z
	  .object({
	    displayName: z.string().optional(),
	    bio: z.string().optional(),
	  })
	  .superRefine((data, ctx) => {
	    if (!data.displayName && !data.bio) {
	      ctx.addIssue({
	        code: z.ZodIssueCode.custom,
	        message: "At least one field must be provided",
	      });
	    }
	  });

This prevents no-op updates like:

	{}

---

### Step 10 — Async Refinements

Both `.refine()` and `.superRefine()` **can be async**.

Example:

	z.string().refine(async value => {
	  return await isEmailAvailable(value);
	});

⚠️ Important rules:

- Async refinements require:
  - `parseAsync()`
  - `safeParseAsync()`
- Synchronous `parse()` will NOT wait for async checks
- Use async refinements carefully (DB calls, latency)

---

### Step 11 — TypeScript Inference

Refinements:

- Do **not** change inferred types
- Affect **runtime validation only**

	type User = z.infer<typeof userSchema>;

The type remains the same — only runtime behavior changes.

---

### Step 12 — Common Production Mistakes

❌ Mistake 1 — Scattering business logic everywhere  
Centralize complex rules inside schemas.

❌ Mistake 2 — Forgetting `path`  
Without a path, errors attach to the root → poor UX.

❌ Mistake 3 — Overusing async refinements  
Validate structure first, then do async checks if needed.

---

### Golden Rule

- Use `.refine()` for **simple validation rules**
- Use `.superRefine()` for **complex, multi-field, conditional rules**

This is the correct, production-grade way to handle advanced validation in Zod.
