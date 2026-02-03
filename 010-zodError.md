## ZodError

### 1. What Is a ZodError?

When Zod validation fails, it creates a **ZodError** — a structured error object you can inspect.

A **ZodError** is:

	import { z } from "zod";

	z.string().parse(123);
	
	This throws a ZodError

Zod throws this special error **only** when `.parse()` fails, allowing you to handle validation problems in a controlled way.

---

### 2. Why Zod Needs Its Own Error Type

Normal JavaScript errors are just text — they don’t explain:

	• Which field failed  
	• Why it failed  
	• What was expected vs. received  

Zod errors are **structured**, not just plain messages — and that structure helps you handle errors accurately.

---

### 3. Basic Shape of a ZodError

A ZodError has a `.issues` array. Roughly:

	{
		name: "ZodError",
		issues: [ ... ]
	}

The **issues** array is the important part — it contains every validation problem. 

---

### 4. The `issues` Array (Most Important)

Each entry in the `issues` array describes one failure:

For example:

	z.string().parse(123);

Produces an issue like:

	[
		{
			code: "invalid_type",
			expected: "string",
			received: "number",
			path: [],
			message: "Expected string, received number"
		}
	]

Zod collects detailed information about every validation problem.
---

### 5. Explanation of Issue Fields

**code**

Indicates the type of validation failure (e.g., `"invalid_type"`, `"too_small"`, `"invalid_string"`). :contentReference[oaicite:4]{index=4}

**expected / received**

Shows what Zod wanted vs. what it actually got. 

**path**

Shows where the error happened.

	path: []

Means the root value was invalid.

For nested objects:

	path: ["user", "email"]

Means `data.user.email` invalid. 

**message**

A human-readable explanation like:

	"Expected string, received number"

You can customize this message.

---

### 6. Object Example (Very Common)

	const schema = z.object({
		name: z.string(),
		age: z.number().min(18),
	});

	schema.parse({
		name: "Alex",
		age: 15,
	});

Produces something like:

	issues: [
		{
			code: "too_small",
			minimum: 18,
			type: "number",
			inclusive: true,
			path: ["age"],
			message: "Number must be greater than or equal to 18"
		}
	]

Zod reports detailed context for each issue. 

---

### 7. Multiple Errors at Once

Zod does **not stop at the first error**:

	schema.parse({
		name: 123,
		age: "young",
	});

Produces:

	issues: [
		{ path: ["name"], ... },
		{ path: ["age"], ... }
	]

This is helpful when validating forms or complex input. 

---

### 8. Catching a ZodError

To handle thrown errors safely:

	try {
	    schema.parse(data);
	} catch (err) {
	    if (err instanceof z.ZodError) {
	        console.log(err.issues);
	    }
	}

This pattern is recommended when using `.parse()`. 

---

### 9. `safeParse()` and ZodError

`.safeParse()` does **not throw** when validation fails.

	const result = schema.safeParse(data);

	if (!result.success) {
		console.log(result.error); // ZodError
	}

`result.error` is a `ZodError` instance and you still get detailed issues. 

---

### 10. Biggest Gotchas ⚠️

❌ **Forgetting try/catch with .parse()**  
→ App crashes unexpectedly.

❌ **Treating ZodError like a string**  
It’s an object with rich data.

❌ **Displaying raw developer error messages to users**  
Default messages are developer-oriented.

❌ **Assuming only one error occurs**  
Zod may return multiple issues at once.

---

### 11. Mental Model

Think of `ZodError` as:

> **A detailed report** of what went wrong with your data, not just “invalid”.  

This perspective helps you handle validations and improve UX.

---

### Final Takeaway

Zod errors are structured, predictable, and contain all the information needed to understand exactly what went wrong during validation — and this behavior matches the **latest official Zod documentation**.
