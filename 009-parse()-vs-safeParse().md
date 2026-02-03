## parse() vs safeParse()

### 1. parse() and safeParse()

In Zod, **parse()** and **safeParse()** are the two main ways to validate data.

Both of them:

- Validate **runtime data**
- Use the **same schema**
- Apply **all constraints and refinements**

The **only difference** between them is **how they fail** when validation does not pass.

Understanding this difference is critical for writing safe production code.

---

### 2. parse() — Throws on Failure

Behavior:

	z.string().parse(123);

What happens step by step:

1. Zod receives the value
2. Zod validates it against the schema
3. Validation fails
4. ❌ Zod **throws a ZodError**

Example error:

	ZodError: Expected string, received number

Important point:

- `parse()` does **not** return a result object
- It either returns valid data **or throws**

---

### 3. Real-World Crash Example

Consider an API endpoint:

	app.post("/login", (req, res) => {
	  const data = loginSchema.parse(req.body); // ❌ dangerous
	  res.send("ok");
	});

What happens if the input is invalid?

- `parse()` throws an exception
- The request handler crashes
- The request fails abruptly
- In some setups, the **entire server process may crash**

This behavior is **not acceptable for user input**.

Users make mistakes. Servers must not crash because of them.

---

### 4. safeParse() — Never Throws

Behavior:

	const result = z.string().safeParse(123);

Returned value:

	{
	  success: false,
	  error: ZodError
	}

Key properties:

- No exception is thrown
- The program continues running
- You get full control over error handling

This makes `safeParse()` safe for untrusted input.

---

### 5. Safe API Example 

Correct way to validate user input:

	app.post("/login", (req, res) => {
	  const result = loginSchema.safeParse(req.body);

	  if (!result.success) {
	    return res.status(400).json({
	      error: "Invalid input"
	    });
	  }

	  const data = result.data;
	  res.send("ok");
	});

Why this is correct:

- Invalid input is handled gracefully
- No crashes
- Clean, predictable responses
- Server remains stable

This is how **production APIs should behave**.

---

### 6. When to Use parse() 

Use `parse()` **only** when invalid data indicates a **developer or configuration bug**.

Characteristics:

- Data is trusted
- Data is controlled by developers
- App should **fail fast** if data is wrong

Examples:

- Environment variables
- Configuration files
- Internal hardcoded values

Example:

	const env = envSchema.parse(process.env);

If environment variables are invalid:

- App crashes immediately
- Crash happens at startup
- This is **GOOD behavior**

Failing early prevents broken production deployments.

---

### 7. When to Use safeParse() 

Use `safeParse()` when data comes from **outside your control**:

- User input
- HTTP requests
- JSON.parse output
- Third-party APIs

Rule of thumb:

	If input is untrusted → use safeParse()

This keeps your application resilient.

---

### 8. Handling Errors Correctly

❌ Bad approach:

	res.json(result.error);

Why this is bad:

- Leaks internal validation details
- Exposes schema structure
- Potential security issue

✅ Good approach:

	res.status(400).json({
	  error: "Invalid request payload"
	});

Best practice:

- Log full `ZodError` internally
- Return minimal, user-friendly errors to clients

---

### 9. Pattern: Parse at Boundaries

Canonical Zod pattern:

	const result = schema.safeParse(input);

	if (!result.success) {
	  // handle error here
	}

	const data = result.data;
	// TRUST data from this point onward

Key idea:

- Validate **once**, at the boundary
- Trust validated data internally
- Do not re-validate in business logic

This keeps code clean and efficient.

---

### 10. Express Middleware Pattern

Reusable validation middleware:

	const validate = schema => (req, res, next) => {
	  const result = schema.safeParse(req.body);

	  if (!result.success) {
	    return res.status(400).json({
	      error: "Invalid input"
	    });
	  }

	  req.body = result.data;
	  next();
	};

Benefits:

- Centralized validation
- Cleaner route handlers
- Guaranteed type-safe `req.body`

This pattern scales well in real applications.

---

### 11. Golden Rule

	parse()     = crash on invalid data  
	safeParse() = controlled failure  

Final takeaway:

- `parse()` is for **bugs**
- `safeParse()` is for **users**
- Choosing correctly prevents production outages

Mastering this distinction is a major step toward writing safe, professional Zod-powered applications.
