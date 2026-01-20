# Skill Structure and Format Reference

## Directory Structure

Every skill requires a directory with a `SKILL.md` file:

```
skill-name/
├── SKILL.md (required)
├── references/ (optional - documentation and templates)
│   ├── REFERENCE.md (index when 5+ files)
│   ├── FORMS.md (form templates, structured data formats)
│   ├── methodology.md
│   └── examples.md
├── scripts/ (optional)
│   └── process-data.js (Node.js preferred)
└── assets/ (optional - static resources like images, data files)
```

## File Naming Conventions

**Use intention-revealing names for all supporting files:**

**Good Examples:**

- `./references/converting-sub-agents.md`
- `./references/aws-deployment-patterns.md`
- `./references/github-workflow-examples.md`
- `./scripts/analyze-complexity.js`
- `./templates/report-template.md`

**Bad Examples (avoid generic names):**

- `./reference.md`
- `./helpers.md`
- `./utils.md`
- `./misc.md`

**Reference files with relative paths:**

- Use `./filename.md` in SKILL.md
- Use lowercase filenames with hyphens
- Group related files in subdirectories (references/, scripts/, assets/)
- **One level deep only:** File references should stay one level deep from SKILL.md to avoid nested reference chains (e.g., `./references/guide.md` but NOT references that link to other references)

## SKILL.md Format

```yaml
---
name: skill-name
description: Clear description of what this skill does and when to use it (max 1024 chars)
---

# Main Instructions

Clear, detailed instructions for Claude to follow when this skill is invoked.

## Step-by-Step Guidance

1. First step
2. Second step
3. Third step

## Examples

Concrete examples showing how to use this skill.

## Reference Documentation

- See `./references/detailed-methodology.md` for [what it contains]
- See `./references/advanced-patterns.md` for [what it contains]
- See `./templates/` for [what templates are available]
```

## YAML Frontmatter Requirements

**Required fields:**

- `name`: Skill identifier (see metadata requirements)
- `description`: When to invoke this skill (see metadata requirements)

**Optional fields:**

- `license` - License specification (e.g., "MIT")
- `compatibility` - Environment requirements (max 500 chars)
- `metadata` - Arbitrary key-value pairs for custom data
- `allowed-tools` - Space-delimited pre-approved tool list (rarely needed; skills inherit CLI capabilities by default)

**Invalid fields (from sub-agent format):**

- `model` - Not applicable to skills
- `tools` - Legacy sub-agent field

**YAML syntax rules:**

- Use spaces, not tabs
- Name and description must be valid YAML strings
- Use quotes if description contains special characters
- Ensure proper indentation

## Skill Locations

**Personal Skills:**

- Location: `~/.claude/skills/`
- Scope: Available across all Claude Code projects
- Use for: General-purpose skills, cross-project utilities

**Project Skills:**

- Location: `.claude/skills/`
- Scope: Project-specific, shared with team
- Use for: Project-specific workflows, team conventions

## Progressive Disclosure Pattern

**Keep SKILL.md lean (<500 lines target):**

```markdown
# Main Skill Instructions

[Core workflow and essential guidance only]

## Step 1: Initial Analysis

[Brief overview - 3-5 bullets]

## Step 2: Processing

See `./references/processing-methodology.md` for detailed approach.

## Step 3: Validation

See `./references/validation-checklist.md` for complete checklist.
```

**Move details to reference files:**

- Detailed methodologies → `./references/methodology.md`
- Extensive examples → `./references/examples.md`
- Long checklists → `./references/checklist.md`
- Background information → `./references/background.md`

**Benefits:**

- Lower initial context cost
- Easier to maintain
- Clearer separation of concerns
- On-demand detail loading

## Multi-File Organization Example

```
analyzing-data/
├── SKILL.md                           # Core workflow (~150 lines)
├── references/
│   ├── REFERENCE.md                   # Index of reference files
│   ├── FORMS.md                       # Form templates, structured data formats
│   ├── data-processing-patterns.md    # Detailed examples
│   ├── sql-optimization-guide.md      # Query best practices
│   └── statistical-methods.md         # Background theory
├── scripts/
│   ├── aggregate-json.js              # Node.js utility
│   └── transform-csv.js               # Node.js utility
└── assets/
    └── sample-data.csv                # Sample data for testing
```

This structure keeps SKILL.md focused while making detailed information available when needed.

## Table of Contents for Long Reference Files

When reference files exceed 200 lines, include a table of contents:

```markdown
# Detailed Methodology Reference

## Table of Contents

- [Overview](#overview)
- [Step-by-Step Process](#step-by-step-process)
- [Advanced Patterns](#advanced-patterns)
- [Troubleshooting](#troubleshooting)

## Overview

[Content...]
```

## Directory Index Files

When a directory contains many files (5+), add an index file to improve navigation:

**references/REFERENCE.md** - Index for reference documentation

- List all reference files with brief descriptions
- Group by category if applicable
- Include usage guidance

**scripts/SCRIPTS.md** - Index for scripts (optional)

- List all scripts with their purpose
- Show usage examples and arguments
- Document dependencies

**assets/ASSETS.md** - Index for assets (optional)

- Catalog available assets
- Describe when to use each

**Benefits:**

- Claude can quickly scan the index to find relevant files
- Reduces need to read multiple files to find the right one
- Better progressive disclosure - load index first, then specific files

## File Organization Best Practices

1. **One topic per reference file** - Don't create catch-all files
2. **Use lowercase filenames** - e.g., `nodejs-patterns.md`, not `NodeJS-Patterns.md`
3. **Group by type** - references/, scripts/, templates/
4. **Intention-revealing names** - Name describes the content clearly
5. **Cross-reference sparingly** - Reference files should be self-contained when possible
6. **Add index files** - When directories have 5+ files, add REFERENCE.md/SCRIPTS.md/ASSETS.md
