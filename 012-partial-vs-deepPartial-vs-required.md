## partial vs deepPartial vs required

### 1. Why Update APIs Are Tricky

In real applications, **CREATE** and **UPDATE** APIs behave very differently.

**CREATE (POST)**  
The user must send **everything**.

	{
	  "email": "a@b.com",
	  "password": "secret",
	  "profile": {
	    "displayName": "Alex"
	  }
	}

**UPDATE (PATCH)**  
The user sends **only what changed**.

	{
	  "profile": {
	    "displayName": "New name"
	  }
	}

If you reuse the **create schema** for updates, validation fails.

➡️ The solution is **derived schemas**, not copied schemas.

---

### 2. `.partial()` — Make All Top-Level Fields Optional

Start with a **base schema**.  
This is your **single source of truth** for CREATE.

	const userSchema = z.object({
	  email: z.string().email(),
	  password: z.string().min(8),
	  profile: z.object({
	    displayName: z.string(),
	    bio: z.string().nullable(),
	  }),
	});

This schema is correct for **POST /create**.

---

#### Apply `.partial()`

	const updateUserSchema = userSchema.partial();

What `.partial()` does:

- Makes **every top-level field optional**
- **Does NOT affect nested objects**

---

#### What Is Now Valid

	{}                            // ✅
	{ email: "new@b.com" }        // ✅
	{ profile: { displayName: "Alex" } } // ❌

❗ **Why does the last one fail?**

Because:

- `profile` itself is optional
- BUT if `profile` exists, it must still be **complete**

---

### 3. Why `.partial()` Is NOT Enough for PATCH APIs

Zoom in on the nested schema:

	profile: z.object({
	  displayName: z.string(),
	  bio: z.string().nullable(),
	});

After `.partial()`:

- `profile` → optional
- `profile.displayName` → **still required**
- `profile.bio` → **still required**

This breaks real PATCH behavior.

PATCH APIs need **nested optionality**, not just top-level optionality.

---

### 4. `.deepPartial()` — Recursive Optionality

The correct schema for PATCH is:

	const updateUserSchema = userSchema.deepPartial();

What `.deepPartial()` does:

- Makes **all fields optional**
- Recursively
- At every depth

---

#### What Is Now Valid

	{}                                   // ✅
	{ email: "new@b.com" }               // ✅
	{ profile: { displayName: "Alex" } } // ✅
	{ profile: {} }                      // ✅

This matches how PATCH APIs work in the real world.

---

### 5. `.required()` — Undo Optionality

Sometimes you want:

- Start with the base schema
- Make it partial
- Then **force specific fields back to required**

Example: email must always exist.

	const updateSchema = userSchema
	  .partial()
	  .required({ email: true });

Result:

- `email` → required
- Everything else → optional

Use this carefully.  
It’s powerful but easy to misuse.

---

### 6. Real PATCH Endpoint

	const updateUserSchema = userSchema.deepPartial();

	app.patch("/user", (req, res) => {
	  const result = updateUserSchema.safeParse(req.body);

	  if (!result.success) {
	    return res.status(400).json({
	      error: "Invalid update payload"
	    });
	  }

	  const updates = result.data;
	  // updates is deeply partial

	  // Apply only provided fields
	  updateUserInDB(updates);

	  res.send("updated");
	});

This approach is:

- Safe
- Predictable
- Scales cleanly as schemas grow

---

### 7. TypeScript Inference

	type UpdateUserInput = z.infer<typeof updateUserSchema>;

The inferred type becomes:

	{
	  email?: string;
	  password?: string;
	  profile?: {
	    displayName?: string;
	    bio?: string | null;
	  };
	}

This type **exactly matches PATCH behavior**.

No manual typing. No drift.

---

### 8. Common Production Mistakes

❌ **Mistake 1 — Copy-pasting schemas**

	const updateSchema = z.object({
	  email: z.string().optional(),
	  password: z.string().optional(),
	  ...
	});

This:

- Duplicates logic
- Causes drift
- Breaks over time

Always derive schemas.

---

❌ **Mistake 2 — Using `.partial()` instead of `.deepPartial()`**

This causes subtle nested update bugs that are hard to detect.

---

❌ **Mistake 3 — Allowing empty updates silently**

Often you want to block this:

	{} // ❌ no-op update

Later, this is solved using `.refine()`.

---

### Golden Rule

**POST → full schema**  
**PATCH → deepPartial schema**  
**Never duplicate schemas**

This rule alone prevents an entire class of production bugs.
