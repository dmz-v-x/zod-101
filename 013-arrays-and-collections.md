## Arrays & Collections

### 1. What `z.array()` Really Validates

	const numbersSchema = z.array(z.number());

This schema means:

- The value **must be an array**
- **Every element** in the array must be a number

Valid:

	[1, 2, 3]
	[]

Invalid:

	[1, "2"]
	"not an array"

Important point:

- **Empty arrays are valid by default** in Zod

This default is intentional and must be restricted explicitly if not desired.

---

### 2. Array Item Validation (VERY IMPORTANT)

Zod validates **each array element individually**.

	const idsSchema = z.array(z.string().uuid());

Invalid array:

	["good-uuid", "bad-value"] // ❌

Zod error paths include the **array index**:

	[1] → "Invalid uuid"

This makes debugging extremely clear and precise, especially for large payloads.

---

### 3. Array Length Constraints

Zod provides built-in constraints for array length.

**Minimum length**

	z.array(z.string()).min(1);

Means: at least 1 item

**Maximum length**

	z.array(z.string()).max(5);

Means: at most 5 items

**Combine both**

	z.array(z.string()).min(1).max(5);

Use these to express business rules clearly.

---

### 4. `.nonempty()` (Important Clarification)

	z.array(z.string()).nonempty();

Behavior-wise:

- Requires **at least one element**
- Same runtime behavior as `.min(1)`

However (important for TypeScript):

- `.nonempty()` returns a **non-empty array type**
- This affects TypeScript inference

So the correct mental model is:

- `.min(1)` → runtime constraint only
- `.nonempty()` → runtime constraint **plus stronger TS typing**

---

### 5. Real-World Examples

#### Bulk Delete Endpoint

	const bulkDeleteSchema = z.object({
	  ids: z.array(z.string().uuid()).nonempty(),
	});

Rejects:

	{ "ids": [] }

Ensures at least one ID is provided and all are valid UUIDs.

---

#### Tags on a Post

	const postSchema = z.object({
	  title: z.string(),
	  tags: z.array(z.string()).max(5),
	});

Limits the number of tags to prevent abuse.

---

#### Pagination Results

	const resultSchema = z.object({
	  items: z.array(
	    z.object({
	      id: z.string(),
	      name: z.string(),
	    })
	  ),
	  total: z.number().int(),
	});

Validates list responses returned from APIs.

---

#### Nested Arrays (Very Common)

	const orderSchema = z.object({
	  items: z.array(
	    z.object({
	      productId: z.string(),
	      quantity: z.number().int().positive(),
	    })
	  ).nonempty(),
	});

Every item is validated **deeply** and correctly.

---

#### TypeScript Inference

	type Order = z.infer<typeof orderSchema>;

Produces:

	{
	  items: {
	    productId: string;
	    quantity: number;
	  }[];
	}

The inferred type exactly matches validated data.

---

### 6. Common Production Mistakes

#### ❌ Mistake 1 — Forgetting `.nonempty()`

	z.array(z.string()); // ❌ allows []

Many APIs require **at least one element**.

---

#### ❌ Mistake 2 — Validating the Array but Not Its Items

	z.array(z.any()); // ❌ useless

This throws away all safety and defeats the purpose of Zod.

---

#### ❌ Mistake 3 — Allowing Huge Arrays

Always think about DoS risks.

	z.array(z.string()).max(100);

Explicit limits protect your system.

---

### 7. Combining Arrays with `.deepPartial()`

For PATCH APIs:

	const updateSchema = orderSchema.deepPartial();

Allows:

	{ items: [{ quantity: 5 }] }

This is valid Zod behavior.

Caution:

- Partial arrays can be tricky
- Often better handled with dedicated update endpoints

---

### 8. Golden Rule

**Arrays validate twice:**

- Once as a list
- Once per element

If either level fails, validation fails.

This is the key mental model to remember.
