---
name: ai-research-to-obsidian
description: Use Claude to search a topic and save the organized results as an Obsidian document. Trigger scenarios: (1) User asks to search a topic using AI (2) User asks to do a browser search and save to Obsidian (3) User says "look this up for me" and mentions saving to notes/docs/Obsidian
---

# AI Research Save to Obsidian

Automated workflow: AI search -> Content organization -> Save in Obsidian format

## Workflow

### 1. Open AI Tool

- **General questions/Research/CitationsProgramming/Technical** -> [Claude](https://claude.ai)

```bash
browser action=open target=host url="https://claude.ai"
```

### 2. Enter Search Content

Type the user's question into the input box and click send.

### 3. Wait for Response

Take a snapshot to capture the response. If the content is long, scroll down to get the full text.

### 4. Organize into Obsidian Format

Create a Markdown file containing:

- YAML frontmatter (date, tags)
- Title (with date annotation)
- Formatted content (tables, lists, hierarchical structure)
- Source attribution

### 5. Save to Obsidian Vault

Find the Obsidian path on macOS:

```bash
mdfind "kMDItemFSName == 'Obsidian'"  # Find local vault
ls ~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/  # iCloud vault
```

Move the file:

```bash
mv <source_file> <obsidian_path>/
```

## Obsidian Document Template

```markdown
---
date: { { DATE } }
tags: [{ { TAGS } }]
---

# {{TITLE}} | {{DATE}}

> Source: {{AI_SOURCE}}

## Key Takeaways

- Point 1
- Point 2

## Details

### Section 1

Content...

---

_Document generated: {{DATE}}_
```

## Notes

- For long content, capture in multiple passes
- Maintain Obsidian formatting standards (frontmatter, heading hierarchy)
- Ask the user for their Vault path if unknown

## Notes

- Prefer Doubao for Chinese-language understanding
- For long content, capture in multiple passes
- Maintain Obsidian formatting standards (frontmatter, heading hierarchy)
- Ask the user for their Vault path if unknown
