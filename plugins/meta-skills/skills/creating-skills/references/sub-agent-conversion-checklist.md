# Sub-Agent Conversion Checklist

Checklist, testing procedures, and troubleshooting for converting sub-agents to skills.

## Conversion Checklist

Use this checklist when converting any sub-agent to a skill:

### Pre-Conversion
- [ ] Read the sub-agent configuration completely
- [ ] Review official documentation (see `./sub-agent-conversion-guide.md`)
- [ ] Identify core expertise and capabilities
- [ ] Extract trigger keywords and use cases from agent description

### YAML Frontmatter
- [ ] Choose gerund-form skill name (e.g., `processing-data`, not `data-processor`)
- [ ] Verify name is lowercase, hyphens only, max 64 chars
- [ ] Verify no leading/trailing/consecutive hyphens
- [ ] Write new description with invocation triggers in third person
- [ ] Remove `model` field
- [ ] Remove `tools` field

### Content
- [ ] Copy core instructions and domain expertise
- [ ] Preserve examples (transform self-references to direct instructions)
- [ ] Add CLI and Node.js tooling emphasis
- [ ] Add validation/testing steps
- [ ] Keep SKILL.md under 500 lines

### Structure
- [ ] Create skill directory matching the name
- [ ] Consider if supporting files would help (use `references/`)
- [ ] Use intention-revealing names for all files
- [ ] Write complete SKILL.md with proper YAML frontmatter

### Validation
- [ ] Test with sample queries that should invoke the skill

## Testing Conversions

After conversion, verify:

### 1. Structure Validation

```bash
ls -la ~/.claude/skills/skill-name/
# Should show SKILL.md and any supporting files
```

### 2. YAML Syntax

- No `model` or `tools` fields
- Description under 1024 characters
- Name in gerund form, max 64 characters
- Name matches directory name
- No tabs (use spaces)

### 3. Invocation Testing

- Ask Claude queries that should trigger the skill
- Verify skill is invoked appropriately
- Check that instructions are followed
- Confirm CLI/Node.js approaches are present

### 4. Content Comparison

- Did we preserve core sub-agent expertise?
- Are examples still present and useful?
- Is domain knowledge intact?
- Are CLI tools emphasized?

## Common Issues and Solutions

### Issue: Skill Not Being Invoked

**Symptoms:** User query should trigger skill, but doesn't

**Causes:**
- Description doesn't contain trigger keywords matching query
- Description explains WHAT not WHEN
- Name not descriptive enough

**Solutions:**
- Add more trigger keywords to description
- Include concrete use cases in description
- Ensure third person voice
- Test with various query phrasings

### Issue: Converted Skill Too Python-Heavy

**Symptoms:** Examples and scripts use Python

**Solutions:**
- Replace all Python examples with Node.js
- Update script files to use `.js` extension with ESM
- Show CLI tool alternatives
- Emphasize Node.js v24+ patterns

### Issue: SKILL.md Too Long

**Symptoms:** Over 500 lines

**Solutions:**
- Move detailed background to `./references/methodology.md`
- Extract checklists to `./references/checklist.md`
- Move examples to `./references/examples.md`
- Keep only core instructions in SKILL.md
- Reference files with relative paths

### Issue: Missing CLI Tooling

**Symptoms:** No CLI tools mentioned or encouraged

**Solutions:**
- Add CLI tools section
- Show specific commands (gh, aws, npm, etc.)
- Provide complete, runnable examples
- Suggest global npm package installations
- Demonstrate command chaining

## Best Practices Summary

1. **Start with documentation** - Review official docs before converting
2. **Description is critical** - Spend time on invocation triggers
3. **Preserve expertise** - Don't lose the sub-agent's domain knowledge
4. **Keep examples** - They're invaluable for understanding
5. **Use gerund names** - `processing-data`, not `data-processor`
6. **Remove agent fields** - No `model` or `tools` in skill YAML
7. **Emphasize CLI/Node** - Show modern tooling approaches
8. **Intention-revealing names** - For all supporting files
9. **Progressive disclosure** - SKILL.md < 500 lines, details in `references/`
10. **Test thoroughly** - Verify invocation and functionality
