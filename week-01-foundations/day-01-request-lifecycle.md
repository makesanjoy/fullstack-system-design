# Week 1 · Day 1 — End-to-End Request Lifecycle

**Stack:** React → HTTPS / Load Balancer → Node.js + Express → PostgreSQL
**Example:** User clicks **Add to Cart**.
**Status:** ✅ Completed

## Request lifecycle

```text
User clicks Add to Cart
  → React sends POST /api/cart/items over HTTPS
  → Load balancer routes to Node.js / Express
  → Backend authenticates, validates, and applies business rules
  → PostgreSQL inserts or updates the cart item
  → Backend returns success or error response
  → React updates UI on confirmed success
```

1. **React:** Use `fetch` or Axios to send an HTTP request.
2. **Load balancer:** Routes the request to a healthy backend server (when present).
3. **Express:** Authenticate the user, validate `productId` and `quantity`, and check business rules.
4. **PostgreSQL:** Persist the cart update; handle errors and constraints.
5. **Response:** Return an appropriate status and JSON. React updates state or displays an error.

## HTTP methods

| Method | Typical use | Idempotent by HTTP semantics? |
| --- | --- | --- |
| GET | Read resource | Yes |
| POST | Create or execute action | Not by default |
| PUT | Replace resource | Yes |
| PATCH | Partially update resource | Not by default |
| DELETE | Delete resource | Yes |

A specific POST or PATCH implementation **can** be idempotent. For example, setting quantity to 2 repeatedly differs from incrementing quantity by 1 repeatedly.

## Example request

```http
POST /api/cart/items
Content-Type: application/json
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{"productId":101,"quantity":1}
```

Frontend code (simplified):

```js
const operationKey = crypto.randomUUID(); // Generate once per user action
const response = await fetch("/api/cart/items", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Idempotency-Key": operationKey,
  },
  body: JSON.stringify({ productId: 101, quantity: 1 }),
});
if (!response.ok) throw new Error("Unable to add to cart");
const cart = await response.json();
// Update React state using confirmed server data.
```

If retrying **the same action**, reuse `operationKey`. A new intentional action gets a new key. The key must be preserved while the operation is unresolved.

## Failure handling and idempotency

- Database temporarily unavailable: return an appropriate server error (often `503 Service Unavailable`), log failure, and do not claim success.
- Database commits but response is lost: the frontend does not know whether the update happened; a naive retry may double-increment quantity.
- Backend stores idempotency keys scoped to the caller and request, and coordinates key recording and the cart write **atomically**. A retry using the same key returns the previous result rather than repeating the mutation.
- Unique database constraints are useful safeguards but **do not replace** idempotency for all operations (e.g., quantity increments).
- Disabling a button reduces accidental duplicates but does not provide reliable backend protection.
- Never trust client-provided price or user identity without server-side verification.

## Interview-ready answer

> React sends an HTTPS POST request when Add to Cart is clicked. A load balancer forwards it to Node.js/Express, which authenticates the user, validates data, and applies business rules. PostgreSQL commits the cart update; the API responds with the updated result, and React updates the UI. If a network failure happens after the write, retries should use the same idempotency key to avoid duplicating the action.

## Quick revision checks

1. Why must validation and authorization happen on the backend?
2. Which HTTP methods are idempotent by default? Can POST be made idempotent?
3. What if the database is down?
4. What if the database commits but the HTTP response is lost?
5. Who generates the idempotency key, and when must it be reused?

## Day 1 mastery notes

- Fundamentals quiz: **5/5 correct**
- End-to-end flow explanation: **passed**; remember to name the HTTP method, database result, and error handling
- Failure/idempotency: **passed**; remember that a duplicate database error is not guaranteed
- **Day 2 must start only after explicit approval.**
