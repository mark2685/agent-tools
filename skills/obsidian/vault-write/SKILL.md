---
name: vault-write
description: Create or update notes in the Obsidian vault. Use when user asks to "create a note", "write to Obsidian", "add a note", "update a note", "save this to Obsidian", or "/vault-write".
---

# /vault-write

Create new notes or update existing notes in the Obsidian vault. All content must follow the Obsidian Flavored Markdown rule.

Default vault is `Hadrian`. Replace `vault=Hadrian` with the target vault name if different.

## Steps

1. **Determine intent.** Ask the user if not clear:
   - **Create** a new note, or **update** an existing one?
   - Note name (and folder, if creating)
   - Content to write
   - Tags (optional)

2. **Create or update the note** — follow the appropriate path below.

3. **Read the note back** to confirm the write succeeded, and report the result to the user. If the read-back shows unexpected content, flag it.

   ```bash
   obsidian vault=Hadrian read file="Note Name"
   ```

## Creating a New Note

Build the content with YAML frontmatter. Always include a `tags` array:

```bash
obsidian vault=Hadrian create name="Note Name" content="---\ntags:\n  - topic\n  - domain\n---\n\n# Note Name\n\nContent here."
```

To create inside a folder, include the path in the name:

```bash
obsidian vault=Hadrian create name="folder/Note Name" content="---\ntags:\n  - topic\n---\n\n# Note Name\n\nContent here."
```

Use wikilinks (`[[Other Note]]`) for any internal vault references. Use callout syntax for important notes. Follow all OFM conventions from the obsidian-flavored-markdown rule.

## Updating an Existing Note

**Append** content to the end of a note:

```bash
obsidian vault=Hadrian append file="Note Name" content="\n\n## New Section\n\nAdditional content."
```

**Prepend** content after frontmatter:

```bash
obsidian vault=Hadrian prepend file="Note Name" content="## New Section\n\nContent at top.\n\n"
```

**Set or update properties.** Use `type=list` for array properties like tags:

```bash
obsidian vault=Hadrian property:set name="tags" value="topic, domain" type=list file="Note Name"
```

Remove a property:

```bash
obsidian vault=Hadrian property:remove name="aliases" file="Note Name"
```
