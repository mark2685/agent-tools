# Root Cause Tracing

Technique for tracing bugs backward through the call stack to find the original trigger.

## When to Use

Use when an error occurs deep in the call stack and the immediate error location is not the root cause. Common in this repo when:
- Proto type mismatches surface in form components but originate in `rpcToForm*` converters
- Server action errors surface in the UI but originate in RPC client configuration
- Test failures surface in assertions but originate in mock setup

## The Technique: Trace Backward

Start at the error and work backward through each call site:

### Step 1: Identify the Bad Value
At the error location, what value is wrong? Is it `undefined`, the wrong type, stale, or malformed?

### Step 2: Where Does It Come From?
Look at the function signature — which parameter or variable holds the bad value? Trace it to the call site.

### Step 3: Repeat Until You Find the Source
Keep tracing up the call chain. At each level ask: "Is this the layer that introduced the bad value, or did it receive it from above?"

### Step 4: Fix at the Source
Fix the bug where the bad value originates, not where it surfaces.

## Example: Proto Type Mismatch

```
Error: Cannot read property 'value' of undefined
  at QuoteLineItemDetails.tsx:45     ← symptom
```

**Trace backward:**
1. `QuoteLineItemDetails.tsx:45` — accessing `lineItem.deliverySchedule.value` → `deliverySchedule` is undefined
2. `QuoteLineItemDetails` receives `lineItem` from `QuoteLineItemsTables.tsx` → check how it maps the data
3. `QuoteLineItemsTables` gets data from `rpcToFormQuoteLine()` → check the converter
4. `rpcToFormQuoteLine()` maps `response.deliverySchedule` → the proto response field was renamed in latest buf-bump
5. **Root cause**: Proto field name changed, converter not updated

**Fix**: Update `rpcToFormQuoteLine()` to use the new field name — not a null check at the component level.

## Common Trace Paths in This Repo

### RPC to UI Data Flow
```
Server Action (_lib/actions/*.actions.ts)
  → Connect RPC client (lib/proto/)
    → Proto response type (@bufteam/*)
      → rpcToForm* converter
        → Form state (React Hook Form)
          → Component render
```

### Form to RPC Submission Flow
```
Form input (component)
  → React Hook Form state
    → Zod validation (schema)
      → formToRpc* converter
        → Server Action
          → Connect RPC client
            → Backend service
```

### Test Failure Flow
```
Test assertion (*.test.tsx)
  → Component render (testing-library)
    → Hook/state initialization
      → Mock setup (@hadrian-mtv/vitest-utils)
        → Mock data shape vs. actual interface
```
