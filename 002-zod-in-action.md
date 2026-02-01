## Zod in Action

### 1.1 User signup API (WITHOUT Zod)

You define a TypeScript type (compile-time only).

	user.ts
	export type SignupInput = {
	  email: string;
	  age: number;
	};

Looks safe ✅  
You expect `email` to be a string and `age` to be a number.

---

### 1.2 Express server using that type

	import express from "express";
	import { SignupInput } from "./user";

	const app = express();
	app.use(express.json());

	app.post("/signup", (req, res) => {
	  const data: SignupInput = req.body;

	  console.log(data.email.toLowerCase());
	  console.log(data.age + 1);

	  res.send("ok");
	});

	app.listen(3000);

Important:

- `data: SignupInput` is ONLY for TypeScript
- It does NOT validate `req.body`
- No runtime checks are happening

---

### 1.3 Bad client request (REAL WORLD)

Someone sends this HTTP request:

	{
	  "email": 123,
	  "age": "twenty"
	}

---

### 1.4 What TypeScript does

❌ TypeScript does NOT check this at runtime  
❌ TypeScript is already gone after compilation  
❌ No error is thrown  

---

### 1.5 Runtime explosion 💥

When this line runs:

	data.email.toLowerCase();

Actual value becomes:

	123.toLowerCase(); // 💥 CRASH

Error:

	TypeError: data.email.toLowerCase is not a function

🚨 This error happens:

- At runtime
- In production
- After deployment
- Possibly for real users

---

### 1.6 CORE REASON 

TypeScript:

- Exists only at compile time
- Is erased at runtime
- Does NOT validate external data

External data includes:

- `req.body`
- `JSON.parse`
- `fetch` responses
- `process.env`

External data must ALWAYS be validated at runtime

---

### 2. Why TypeScript cannot protect you here

Example with `JSON.parse` (no server involved):

	type User = {
	  name: string;
	};

	const user: User = JSON.parse('{"name": 123}');

	console.log(user.name.toUpperCase());

TypeScript says: ✅ NO ERROR  
Runtime says: 💥 CRASH  

Why?

- `JSON.parse` returns `any`
- You assigned it to `User`
- TypeScript trusts you blindly

---

### 3. The WRONG way people try to “fix” it

❌ This does NOTHING:

	const data = req.body as SignupInput;

❌ This also does NOTHING:

	function signup(data: SignupInput) {}
	signup(req.body);

These are **type assertions**, not validation.

They silence TypeScript — they do NOT protect your app.

---

### 4. Same scenario — WITH Zod

---

### 4.1 Define schema (runtime truth)

	import { z } from "zod";

	const signupSchema = z.object({
	  email: z.email(),
	  age: z.number(),
	});

This schema:

- Exists at runtime
- Actively checks values
- Knows shape + constraints

---

### 4.2 Use schema BEFORE trusting data

	app.post("/signup", (req, res) => {
	  const result = signupSchema.safeParse(req.body);

	  if (!result.success) {
	    return res.status(400).json({
	      error: result.error.format(),
	    });
	  }

	  const data = result.data;

	  console.log(data.email.toLowerCase());
	  console.log(data.age + 1);

	  res.send("ok");
	});

Now the code is safe.

---

### 4.3 What is safeParse?

`safeParse` is a Zod method that validates data against a schema **without throwing errors**.

It returns a result object:

- Success case:
	{ success: true, data }
- Failure case:
	{ success: false, error }

This allows you to handle invalid input safely and predictably.

---

### 4.4 What is result.success?

`result.success` is a boolean returned by `safeParse`.

- `true` → data matches the schema
- `false` → validation failed

This is how you branch logic safely.

---

### 4.5 Where does result.error.format() come from?

- `result.error` is a `ZodError`
- `.format()` converts errors into a structured, readable object
- Ideal for API responses

Example structure:

	{
	  "email": { "_errors": ["Expected string, received number"] },
	  "age": { "_errors": ["Expected number, received string"] }
	}

---

### 4.6 What is result.data?

- `result.data` exists only when validation succeeds
- It contains fully validated, type-safe data
- Guaranteed to match the schema

This means:

- `data.email` is a valid email string
- `data.age` is a valid number
- No runtime crashes

---

### 4.7 Same bad request again

	{
	  "email": 123,
	  "age": "twenty"
	}

What happens now?

✅ Zod catches it  
✅ No crash  
✅ Controlled error response  

---

### 5. TypeScript + Zod together (BEST PRACTICE)

Now derive the TypeScript type from the Zod schema:

	type SignupInput = z.infer<typeof signupSchema>;

Why this is huge:

- Runtime validation 
- Compile-time typing 
- ONE source of truth
- Impossible for schema & type to drift

---

### Final takeaway

- TypeScript provides compile-time safety
- Zod provides runtime safety
- External data is always untrusted
- Zod complements TypeScript — it does not replace it
