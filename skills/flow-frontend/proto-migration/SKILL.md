---
name: proto-migration
description: Fixes flow-frontend breakage after a proto bump — updates converters, server actions, and forms — distinguishing mechanical renames from semantic changes that must be escalated. Use when the user says "fix the proto breakage", "the buf bump broke types", "migrate after the proto change", "update converters for the new protos", or "/proto-migration". User-invoked only — it is the side-effecting post-bump companion to buf-bump, gated to match.
disable-model-invocation: true
---

# /proto-migration

The "fix the breakage" companion to `buf-bump`. After generated `@bufteam/*` packages change,
this restores the consuming code: `rpcToForm*` / `formToRpc*` converters, server actions, and
forms. The hard part is judgment — telling a mechanical rename (safe to apply) from a semantic
change (must be escalated). Applying a default or null-check to paper over a removed field hides
lost data; that is the silent-failure trap this skill exists to prevent.

Runs in **flow-frontend**, after a bump has already landed (via `buf-bump`). If the protos
aren't bumped yet, use `buf-bump` first.

## Steps

1. **Surface the full breakage.** Run `pnpm lint:tsc`. Capture every error — do not fix the
   first one and re-run blind. The error set is the migration's scope.

2. **Triage each error into one of two buckets:**

   **Mechanical** — apply directly:
   - Field renamed → update the field name in the converter and any direct proto usage.
   - Type widened, or a new *optional* field added → adjust the converter; default the new
     field only if the proto marks it optional and the form has a sensible empty value.
   - Enum value renamed (same meaning) → update the mapping.

   **Semantic — STOP and escalate to the user.** Do not invent a fix:
   - A field was **removed** — this is a backend contract change. A removed field means the
     data is gone; a null-check hides that. Report it.
   - An enum value was **dropped** — a previously valid state no longer exists; the UI handling
     it needs a product decision.
   - A field's **meaning or units changed**, or required↔optional flipped — semantics, not
     syntax. Escalate.

3. **Fix mechanical changes at the edges.** All proto↔form mapping lives in converters and
   server actions, never in components (the `protobuf` rule, installed at
   `.claude/rules/protobuf.md`; `/add-server-action`). Use `*Required` types from `hadrian_protoc-gen-es-required` for form
   validation. Re-check int64-as-string, RFC 3339 timestamp, and proto3-default handling — a
   bump can change which of these applies to a field.

4. **Add a canary for any workaround you couldn't avoid.** If you had to special-case
   something, add a positive assertion (test or runtime guard) that fails loudly when the
   contract shifts again — so the next bump can't silently re-break it. Mark load-bearing
   workarounds with a comment explaining why.

5. **Verify.** `pnpm lint:tsc` clean, then `pnpm test`. Update converter and form tests so they
   assert the new field shape through the real path, not the old one.

6. **Report.** List what you fixed mechanically, every semantic change you escalated (these
   block until the user decides), and any canary you added.
