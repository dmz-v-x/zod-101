## Installing Zod

### 0. What Zod *Really* Is

Zod is a **runtime schema validation library** for TypeScript and JavaScript.  
It runs **in your application code**, not just at compile time. Zod exists to let you validate external data safely when your app runs. 

That means:

- Zod runs *at runtime*
- It must be **installed as a normal dependency**
- It is bundled or shipped with your app
- Your production build still needs Zod available in `node_modules`

---

### 1. Step 1: Install Zod

To install Zod in your project directory, run this:

	npm install zod

You can also use:

	yarn add zod  
	pnpm add zod

✔ This downloads Zod  
✔ Adds it to `node_modules`  
✔ Adds it to your `package.json`

#### ❌ Do *not* install Zod as a dev dependency

	npm install -D zod   # ❌ WRONG

Why?

Because Zod doesn’t just help at development time — it **runs when your code executes**. If you install it only as a dev dependency, your production environment may not have it available after deployment, causing runtime errors. Zod is a **true runtime library**. 

---

### 2. Step 2 — Import Zod

In your TypeScript or JavaScript file:

	import { z } from "zod";

Here:

- `z` is the Zod namespace
- Everything you build starts with `z.something()`

Examples:

- `z.string()`  
- `z.number()`  
- `z.object()`  

These create validation schemas that can parse and check real data. 

---

### 3. Step 3 — Smallest Runnable Zod Example

Create a file, e.g., `index.ts`:

	import { z } from "zod";

	const schema = z.string();

	console.log(schema.parse("hello"));

If you run this code, you’ll see:

	hello

Now change it to:

	console.log(schema.parse(123));

❌ You’ll get a runtime error, because:

- Zod enforces correctness at runtime  
- It *throws* when input doesn’t match schema  

This behavior is intentional and helps catch bugs early.

---

### 4. Step 4 — Why `.parse()` Throwing Errors Matters

Zod’s `.parse()` method **throws errors** when validation fails.

This is good because:

- It immediately stops incorrect code
- It ensures only valid data continues
- It makes misuse obvious during development

Later, you’ll learn about `.safeParse()` — a safer alternative that returns a result object instead of throwing.

As a rule:

- Use `.parse()` when invalid data is a *bug*
- Use `.safeParse()` when validating *external user input*

Zod enforces runtime correctness by design. 

---

### 5. Node + Express + Zod in Production

Imagine a typical Node + Express + TypeScript app.

Install dependencies:

	npm install express zod  
	npm install -D typescript @types/node @types/express

Then import Zod normally:

	import { z } from "zod";

Zod will work in:

- Development  
- Production  
- Docker containers  
- Serverless environments  
- Edge runtimes  

No special bundling or configuration required.

---

### 6. Production Build (TypeScript → JavaScript)

When you run something like:

	npm run build

TypeScript will:

- Compile `.ts` → `.js`
- Keep all Zod-related code intact

Zod doesn’t disappear after compilation — it stays as regular JavaScript in your build output:

	const { z } = require("zod");

Because Zod runs at runtime, your production server *must* include it in `node_modules`.

---

### 7. Does Zod Affect Production Performance?

Short answer: **very minimal impact** when used correctly.

Zod only runs when you explicitly call methods like:

- `.parse()`  
- `.safeParse()`  

It does *not* run automatically or in the background. There are:

- No global performance penalties
- No background hooks
- No runtime overhead outside your validation calls

💡 Production tips

- Validate only at boundaries (e.g., API entry points)
- Don’t validate the same data repeatedly
- Avoid running heavy async refinements in tight loops
