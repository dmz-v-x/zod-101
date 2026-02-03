## `transform` vs `preprocess`

### Step 1 — Why Validation Alone Is Not Enough

In real applications, raw input is messy and inconsistent.

Example input:

	{
	  "email": "  TEST@Example.com ",
	  "age": "20",
	  "name": "   Himanshu   "
	}

Validation alone can only say:

- ❌ Invalid
- ❌ Wrong type

But production applications need **more than rejection**. They need:

- Trimming
- Normalization
- Type coercion
- Shaping raw input into clean domain data

That is exactly what **transformations** in Zod are designed for.

---

### Step 2 — `.transform()` 

**Mental model (important)**

`.transform()` runs **only after validation succeeds**.

Order of execution:

1. Validate
2. Transform

Basic example:

	const trimmedString = z
	  .string()
	  .transform(val => val.trim());

	trimmedString.parse("  hello  ");

Result:

	"hello"

Key points:

- Input must already be valid
- Invalid input never reaches `.transform()`
- `.transform()` is for shaping valid data

---

### Step 3 — Real-World Example: Normalizing Email

	const emailSchema = z
	  .string()
	  .email()
	  .transform(email => email.toLowerCase().trim());

Input:

	"  TEST@Example.COM  "

Output:

	"test@example.com"

This combines:

- Validation (email format)
- Normalization (lowercase + trim)

All in one place.

---

### Step 4 — `.transform()` Can Change the Output Type

This is **very important**.

	const lengthSchema = z
	  .string()
	  .transform(str => str.length);

	const result = lengthSchema.parse("hello");

Result:

	5

TypeScript inference:

	type Result = z.infer<typeof lengthSchema>;

Resulting type:

	number

Important takeaway:

- Schema input type ≠ schema output type
- Zod tracks this correctly in TypeScript

---

### Step 5 — `.preprocess()`

**Mental model (CRITICAL)**

`.preprocess()` runs **before validation**.

Use it when:

- Input type is wrong
- But convertible
- And you want to accept it

Basic example: string → number

	const ageSchema = z.preprocess(
	  val => Number(val),
	  z.number().int().positive()
	);

Input:

	"20"

Process:

- preprocess → 20
- validate → positive integer

Output:

	20

---

### Step 6 — `.preprocess()` vs `z.coerce.*`

Zod provides built-in coercion helpers:

- z.coerce.number()
- z.coerce.boolean()
- z.coerce.date()

Example:

	const ageSchema = z.coerce.number().int().positive();

This is effectively:

- preprocess + validation combined

Rule of thumb:

- Use `z.coerce.*` for simple conversions
- Use `.preprocess()` for custom logic

---

### Step 7 — Real-World Production Examples

#### A) Form Input Cleanup

	const formSchema = z.object({
	  name: z.string().transform(v => v.trim()),
	  email: z.string().email().transform(v => v.toLowerCase()),
	});

#### B) Query Parameters (Always Strings)

	const querySchema = z.object({
	  page: z.coerce.number().int().positive(),
	  pageSize: z.coerce.number().int().max(100),
	});

#### C) Boolean Flags from Strings

	const flagSchema = z.coerce.boolean();

Accepts:

	"true", "false", 1, 0

---

### Step 8 — Chaining `.transform()`

	const schema = z
	  .string()
	  .transform(v => v.trim())
	  .transform(v => v.toUpperCase());

Transforms execute **in order**, left to right.

---

### Step 9 — `.transform()` and `.refine()` Order (IMPORTANT)

Order matters.

	z.string()
	  .transform(v => v.trim())
	  .refine(v => v.length > 0);

Execution flow:

1. Validate string
2. Transform (trim)
3. Refine (check length)

This is ideal for:

- Cleaning input
- Then enforcing business rules

---

### Step 10 — Common Production Mistakes

❌ Mistake 1 — Using `.transform()` to fix invalid types

	z.string().transform(Number); // ❌

This fails because validation happens first.

Use:

- `.preprocess()`
- or `z.coerce.*`

---

❌ Mistake 2 — Heavy logic inside `.transform()`

Transformations should be:

- Deterministic
- Fast
- Pure

No database calls.
No network requests.

---

❌ Mistake 3 — Forgetting output type changes

Transformations affect inferred types.

Always be aware of the output shape.

---

### Golden Rules

- `preprocess` = fix **input shape**
- `transform` = shape **output data**

Validation protects correctness.
Transformations produce clean, usable domain values.
