## `merge` vs `intersection`

### Step 1 — The Real Problem: Multiple Concerns, One Payload

In real applications, request data often comes from **different concerns**:

- Authentication data
- Pagination
- Filters
- Metadata
- Shared headers or context

Example request payload:

	{
	  "email": "a@b.com",
	  "password": "secret",
	  "page": 1,
	  "pageSize": 20
	}

These fields belong to **different logical domains**, but they arrive together.

You should **not** model this as one giant schema from scratch.

Instead, Zod gives you tools to **compose schemas safely**.

---

### Step 2 — `.merge()` 

#### What `.merge()` Means

`.merge()` combines **two object schemas** into a **single object schema**.

Important rules from Zod:

- Works **only with `z.object()` schemas**
- Result is a new object schema
- If keys conflict, **the second schema wins**

#### Basic Example

	const authSchema = z.object({
	  email: z.string().email(),
	  password: z.string().min(8),
	});

	const paginationSchema = z.object({
	  page: z.number().int().positive(),
	  pageSize: z.number().int().max(100),
	});

	const combinedSchema = authSchema.merge(paginationSchema);

Resulting shape:

	{
	  email: string;
	  password: string;
	  page: number;
	  pageSize: number;
	}

This is the **recommended way** to compose API payloads.

---

### Step 3 — Key Conflict Behavior in `.merge()`

If both schemas define the same key, **the second schema overrides the first**.

Example:

	const a = z.object({ role: z.string() });
	const b = z.object({ role: z.literal("admin") });

	const result = a.merge(b);

Result:

	role → "admin"

This is **intentional behavior** in Zod and must be used carefully.

---

### Step 4 — `.intersection()` 

#### What `.intersection()` Means

`.intersection()` enforces **logical AND**:

> The value must satisfy **both schemas at the same time**.

Syntax:

	const schema = z.intersection(schemaA, schemaB);

Important characteristics:

- Works with **any schema type**
- Does **not merge schemas**
- Performs **dual validation**

---

### Step 5 — Object Intersection Example

	const hasId = z.object({
	  id: z.string(),
	});

	const hasTimestamp = z.object({
	  createdAt: z.date(),
	});

	const schema = z.intersection(hasId, hasTimestamp);

Validation rule:

- Object must have **both `id` and `createdAt`**
- Missing either → validation fails

This is **validation**, not object merging.

---

### Step 6 — Non-Object Intersection (Where `.merge()` Cannot Work)

`.merge()` only works for objects.

`.intersection()` works for **constraints**.

Example:

	const positive = z.number().positive();
	const integer = z.number().int();

	const positiveInt = z.intersection(positive, integer);

This enforces:

- Must be a number
- Must be positive
- Must be an integer

This is **impossible** with `.merge()`.

---

### Step 7 — `.merge()` vs `.intersection()`

| Feature | `.merge()` | `.intersection()` |
|------|-----------|------------------|
| Works on objects | Yes | Yes |
| Works on non-objects | No | Yes |
| Purpose | Object composition | Constraint composition |
| Key conflicts | Second schema wins | Must satisfy both or fail |
| Readability | High | Medium |
| Typical usage | APIs, payloads | Advanced validation |

Key idea:

- `.merge()` **builds objects**
- `.intersection()` **combines rules**

---

### Step 8 — Real Production Examples

#### Example A — API Request Composition (Use `.merge()`)

	const filterSchema = z.object({
	  query: z.string().optional(),
	});

	const paginationSchema = z.object({
	  page: z.number().int().positive(),
	});

	const requestSchema = filterSchema.merge(paginationSchema);

Why this is correct:

- Clean separation of concerns
- Readable schemas
- Easy to evolve

---

#### Example B — Numeric Constraints (Use `.intersection()`)

	const priceSchema = z.intersection(
	  z.number().positive(),
	  z.number().max(1000)
	);

This ensures **both conditions** are enforced.

---

#### Example C — Authenticated Request

	const authSchema = z.object({
	  userId: z.string().uuid(),
	});

	const payloadSchema = z.object({
	  data: z.string(),
	});

	const requestSchema = z.intersection(authSchema, payloadSchema);

Useful when schemas come from **different layers** of the system.

---

### Step 9 — TypeScript Inference

	type Request = z.infer<typeof requestSchema>;

Facts from Zod:

- `.merge()` produces **clean object types**
- `.intersection()` produces **intersection types** (`A & B`)
- Intersection types may be harder to work with in TypeScript

Best practice:

- Prefer `.merge()` for objects
- Use `.intersection()` only when needed

---

### Step 10 — Common Production Mistakes

#### ❌ Mistake 1 — Using `.intersection()` for Objects Unnecessarily

	z.intersection(a, b); // ❌ harder to read and reason about

Use `.merge()` instead.

---

#### ❌ Mistake 2 — Assuming `.merge()` Enforces Both Constraints

`.merge()` **overrides on conflict**.

If you need **both rules**, use `.intersection()`.

---

#### ❌ Mistake 3 — Intersecting Incompatible Schemas

	z.intersection(
	  z.string(),
	  z.number()
	); // ❌ impossible to satisfy

Always think logically.

---

### Golden Rule

- **Objects → use `.merge()`**
- **Rules / constraints → use `.intersection()`**

Choose composition tools intentionally, and your schemas will stay clean, predictable, and safe.
