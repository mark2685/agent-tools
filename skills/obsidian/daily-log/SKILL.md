---
name: daily-log
description: Append an entry to today's Obsidian daily note. Use when user says "log this", "add to daily note", "daily log", "jot this down", or "/daily-log".
---

# /daily-log

Append a bullet-list entry to today's daily note in Obsidian.

Default vault is `Hadrian`. Replace `vault=Hadrian` with the target vault name if different.

## Steps

1. **Determine what to log.** If the user says "log this" without specifying content, summarize the most recent significant action in the current Claude Code session (e.g., "Created vault-write skill for obsidian project bundle"). Present the proposed summary and ask for confirmation before appending.

2. **Format as a bullet item** matching the existing daily note style:
   - Top-level items: `- Item text`
   - Sub-items: `\t- Sub-item text`
   - Use plain text, not wikilinks or formatting, unless the user explicitly provides them

   Examples:
   - Single item: `- Reviewed PR for capacity demand endpoint`
   - Nested: `- Domain-Driven Design:\n\t- Chapter 3 — model integrity patterns`

3. **Append to today's daily note:**

   ```bash
   obsidian vault=Hadrian daily:append content="\n- Entry text here"
   ```

   Lead with `\n` to ensure a blank line separator from existing content.

   For nested items:

   ```bash
   obsidian vault=Hadrian daily:append content="\n- Parent item:\n\t- Sub-item one\n\t- Sub-item two"
   ```

4. **Read back to confirm:**

   ```bash
   obsidian vault=Hadrian daily:read
   ```

5. Report the updated daily note content to the user.
