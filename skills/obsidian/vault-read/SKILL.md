---
name: vault-read
description: Pull Obsidian vault content into the current session as context. Use when user asks to "read a note", "search the vault", "find in Obsidian", "check my notes", "look up", or "/vault-read".
---

# /vault-read

Read notes, search content, and explore structure in the Obsidian vault.

Default vault is `Hadrian`. Replace `vault=Hadrian` with the target vault name if different.

## Steps

1. **Determine what the user wants.** If not specified, ask:
   - A specific note name or path?
   - A keyword or topic to search for?
   - A folder to browse?

2. **Read a specific note** by name or path:

   ```bash
   obsidian vault=Hadrian read file="Note Name"
   # or by exact path:
   obsidian vault=Hadrian read path="folder/note.md"
   ```

3. **Search the vault** by keyword with surrounding context:

   ```bash
   obsidian vault=Hadrian search:context query="search term"
   ```

   For a quick file list without context:

   ```bash
   obsidian vault=Hadrian search query="search term"
   ```

   Optionally add `limit=N` to cap results if the vault is large.

4. **Browse a folder** — list files then read as needed:

   ```bash
   obsidian vault=Hadrian files folder="folder-name"
   # Then read individual notes from the results
   obsidian vault=Hadrian read path="folder-name/note.md"
   ```

5. **Orient within a long note** — show headings first:

   ```bash
   obsidian vault=Hadrian outline file="Note Name"
   ```

6. **Explore connections** — show what links to a note:

   ```bash
   obsidian vault=Hadrian backlinks file="Note Name"
   ```

7. **Discover vault structure** when unsure where things live:

   ```bash
   obsidian vault=Hadrian folders
   obsidian vault=Hadrian tags counts sort=count
   ```

8. **Output the content directly** so it can be used as context for the current task. Do not summarize unless the user asks for a summary.
