## Type Inference and `z.infer`

### Step 1 — The Real Problem Zod Is Solving (Deeper View)

Without Zod, teams often end up with **two parallel definitions**:

TypeScript type (compile-time only):

	type User = {
	  email: string;
	  role: "user" | "admin";
	};

And separately, a runtime validator:

	const userSchema = z.object({
	  email: z.string().email(),
	  role: z.enum(["user", "admin"]),
	});

This creates serious problems:

- ❌ Two sources of truth
- ❌ Types and validation drift over time
- ❌ Bugs appear silently in production

Zod’s core philosophy is simple and strict:

**Schemas are the source of truth.  
Types must be derived — never duplicated.**

---

### Step 2 — What `z.infer` Actually Does

	type User = z.infer<typeof userSchema>;

This tells TypeScript:

- Look at the **runtime Zod schema**
- Extract its **TypeScript shape**
- Keep both perfectly in sync

Important facts:

- `z.infer` has **zero runtime cost**
- It runs **only at compile time**
- Types always reflect real validation rules

---

### Step 3 — Basic Inference Example

	const userSchema = z.object({
	  email: z.string().email(),
	  age: z.number().int().optional(),
	});

	type User = z.infer<typeof userSchema>;

Inferred type:

	{
	  email: string;
	  age?: number;
	}

This matches runtime behavior **exactly**:

- `email` is required
- `age` is optional
- `age` must be an integer if present

No guessing. No mismatch.

---

### Step 4 — Real Production Pattern: API Requests

#### Step 4.1 — Define the schema (boundary)

	const createUserSchema = z.object({
	  email: z.string().email(),
	  password: z.string().min(8),
	});

#### Step 4.2 — Infer the type

	type CreateUserInput = z.infer<typeof createUserSchema>;

#### Step 4.3 — Use in service layer

	function createUser(input: CreateUserInput) {
	  // input is already guaranteed valid
	}

Why this pattern works:

- 🔥 Validation happens at the boundary
- 🔥 Types are trusted everywhere else
- 🔥 No duplication anywhere

This is **how Zod is meant to be used**.

---

### Step 5 — Derived Schemas → Derived Types

Schemas are not static — they evolve.

	const updateUserSchema = createUserSchema.deepPartial();

Now infer again:

	type UpdateUserInput = z.infer<typeof updateUserSchema>;

Resulting type:

	{
	  email?: string;
	  password?: string;
	}

Key insight:

- You did NOT redefine fields
- You did NOT rewrite types
- Types adapt automatically

This is **professional-grade schema design**.

---

### Step 6 — Inference with Unions & Discriminated Unions

	const paymentSchema = z.discriminatedUnion("method", [
	  z.object({
	    method: z.literal("card"),
	    cardNumber: z.string(),
	  }),
	  z.object({
	    method: z.literal("upi"),
	    upiId: z.string(),
	  }),
	]);

	type Payment = z.infer<typeof paymentSchema>;

TypeScript now understands control flow:

	if (payment.method === "card") {
	  payment.cardNumber; // ✅
	}

No casting.  
No `as`.  
No unsafe checks.

---

### Step 7 — Schema-First Architecture

A clean project structure looks like this:

	schemas/
	  user.schema.ts
	  auth.schema.ts

	services/
	  user.service.ts

	routes/
	  user.route.ts

Workflow:

- Schemas define truth
- Types are inferred from schemas
- Routes validate input
- Services accept inferred types

This creates a **single, reliable source of truth**.

---

### Step 8 — Exporting Both Schema and Type

	export const userSchema = z.object({
	  id: z.string().uuid(),
	  email: z.string().email(),
	});

	export type User = z.infer<typeof userSchema>;

Benefits:

- Schema used for runtime validation
- Type used everywhere else
- Guaranteed sync forever

This is a **recommended production pattern**.

---

### Step 9 — Common Production Mistakes

#### ❌ Mistake 1 — Writing Types Manually

	type User = {
	  email: string;
	}; // ❌ drift risk

Always infer from schemas.

---

#### ❌ Mistake 2 — Using Type Assertions Instead of Validation

	const user = data as User; // ❌ unsafe

Inference does NOT validate data.  
Validation must happen first.

---

#### ❌ Mistake 3 — Inferring from Weak Schemas

	const schema = z.any();
	type T = z.infer<typeof schema>; // ❌ useless

Inference is only meaningful when schemas are precise.

---

### Step 10 — When NOT to Use `z.infer`

Rare cases:

- Public library APIs
- Very generic utility types

Even then, prefer schema-first when possible.

---

### Golden Rule

**Never write a TypeScript type that already exists in a Zod schema.**

Schemas validate at runtime.  
Types infer at compile time.  

That is the Zod way.
