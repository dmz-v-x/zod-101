## String Constraints in Zod

### 1. String Constraints in Zod 

(.min(), .max(), .regex(), and **email validation**)

This lesson answers a very real question:

“`z.string()` accepts empty strings — how do we enforce real rules?”

---

### 2. Why Constraints Exist

This is valid JavaScript:

	typeof "" === "string"; // true

So by default:

	z.string().parse("");

✅ passes

But in real applications:

- empty usernames ❌
- empty passwords ❌
- invalid emails ❌

➡️ Constraints express **business rules**, not just types.

Zod intentionally separates:
- **type validation** (string, number, boolean)
- **rule validation** (length, format, pattern)

---

### 3. `.min()` — Minimum Length

Basic usage:

	const usernameSchema = z.string().min(1);

Meaning:

- must be a string
- must have at least 1 character

	usernameSchema.parse("a"); // ✅
	usernameSchema.parse("");  // ❌

Custom error message (important in production):

	const usernameSchema = z.string().min(1, "Username is required");

Zod will now return:

	"Username is required"

instead of a generic error.

---

### 4. `.max()` — Maximum Length

	const usernameSchema = z.string().max(20);

Meaning:

- string length ≤ 20

	usernameSchema.parse("short");        // ✅
	usernameSchema.parse("a".repeat(30)); // ❌

You can combine `.min()` and `.max()`:

	import { z } from "zod";

	const usernameSchema = z
	  .string()
	  .min(3, "Username too short")
	  .max(20, "Username too long");

Order does not matter logically, but readability does.

---

### 5. Email Validation (UPDATED — Latest Zod Docs)

⚠️ **Important update according to the latest Zod documentation**

`z.string().email()` **still works**, but it is **deprecated** in Zod v4.

✅ **Recommended approach (official docs):**

Use the top-level email schema:

	const emailSchema = z.email();

Valid:

	emailSchema.parse("test@example.com"); // ✅

Invalid:

	emailSchema.parse("test");          // ❌
	emailSchema.parse("test@");         // ❌
	emailSchema.parse("@example.com");  // ❌

Custom error message:

	const emailSchema = z.email("Invalid email address");

Important clarifications (from Zod docs):

- This validates **format only**
- It does NOT verify domain existence
- It does NOT send emails

---

### 6. `.regex()` — Custom Patterns

Used when you need **custom rules**:

- passwords
- usernames
- access codes
- special formats

Example: password must contain a number

	const passwordSchema = z
	  .string()
	  .min(8, "Password too short")
	  .regex(/[0-9]/, "Password must contain a number");

Valid:

	"abc12345" // ✅

Invalid:

	"abcdefgh" // ❌

Zod evaluates **all constraints**, not just the first one.

---

### 7. Real-World Signup Schema (Incremental Build)

Let’s build this properly, step by step.

#### Step 1 — Start simple (NOT production-ready)

	import { z } from "zod";

	const signupSchema = z.object({
	  email: z.string(),
	  password: z.string(),
	});

This only checks types — **no real rules yet**.

---

#### Step 2 — Add email validation (UPDATED)

	import { z } from "zod";

	const signupSchema = z.object({
	  email: z.email("Invalid email"),
	  password: z.string(),
	});

This now validates email format correctly.

---

#### Step 3 — Add password rules

	import { z } from "zod";

	const signupSchema = z.object({
	  email: z.email("Invalid email"),
	  password: z
	    .string()
	    .min(8, "Password is too short")
	    .max(20, "Password is too long")
	    .regex(/[0-9]/, "Password must contain a number"),
	});

Now we have:

- real validation
- meaningful error messages
- runtime safety

---

### 8. Test with `safeParse`

	const result = signupSchema.safeParse({
	  email: "not-an-email",
	  password: "short",
	});

Result:

	{
	  success: false,
	  error: {
	    email: "Invalid email",
	    password: [
	      "Password is too short",
	      "Password must contain a number"
	    ]
	  }
	}

Multiple constraints → **multiple errors**

This is intentional and powerful.

---

### 9. Common Beginner Mistakes

#### Mistake 1 — Forgetting `.min(1)`

	z.string(); // accepts ""

Always ask:

“Should an empty string be allowed?”

---

#### Mistake 2 — Overusing `.regex()`

Bad:

	z.string().regex(/.+@.+\..+/);

Good (built-in, clearer intent):

	z.email();

Use Zod’s built-ins whenever they exist.

---

#### Mistake 3 — Frontend-only validation

Frontend validation ≠ backend safety.

Zod **must** run on the server.

---

### 10. Derived TypeScript Type (Best Practice)

	type SignupInput = z.infer<typeof signupSchema>;

Now you get:

- Runtime validation ✅
- Compile-time types ✅
- One source of truth ✅
- Zero drift between rules and types

---

### Final Takeaway

- `z.string()` validates type only
- Constraints enforce **business rules**
- Email validation should use `z.email()` (latest docs)
- Zod is strict by default — this is a feature
- Runtime validation + TypeScript inference is the winning combo
