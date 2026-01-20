# Task Prompt Templates

Copy-paste ready templates for common Task agent scenarios.

---

## Task with Skill Invocation

The most important template — ensures task agents actually invoke skills.

```
<context>
[Background information relevant to the task]
Project: [project name/path]
Files: [relevant file paths]
</context>

<task>
[Clear description of what needs to be accomplished]
</task>

<skill_invocation>
To complete this task, invoke the [skill-name] skill using the Skill tool:

  Skill(skill: '[skill-name]')

The skill provides: [what the skill offers - templates, workflows, criteria]
</skill_invocation>

<output_format>
[How to structure the results]
</output_format>
```

### Example: Citation Verification Task

```
<context>
Project: wellness-app
File to audit: articles/chronic-pain.md
Priority: Core claims (marked with 🔴) must be verified before launch
</context>

<task>
Verify the scientific citations in the chronic pain article. Check that each
claim is accurately attributed and the source says what we claim it says.
</task>

<skill_invocation>
To complete this task, invoke the auditing-citations skill using the Skill tool:

  Skill(skill: 'auditing-citations')

The skill provides: structured verification workflow, output templates,
red flag criteria for common citation errors.
</skill_invocation>

<output_format>
Return a verification report with:
- Summary table (claim, status, key issue)
- Detailed findings for each claim
- Required corrections with exact replacement text
</output_format>
```

---

## Explore Agent Template

For codebase exploration, file searches, and pattern discovery.

```
<task>
[What to search for or explore]
</task>

<search_scope>
[Where to look - directories, file patterns, keywords]
</search_scope>

<output_format>
Return:
- List of relevant files found
- Key patterns or implementations discovered
- Summary of findings
</output_format>
```

### Example: Find Authentication Patterns

```
<task>
Find how authentication is implemented in this codebase.
</task>

<search_scope>
- Search for files containing "auth", "login", "session", "token"
- Look in src/ directory
- Check for middleware patterns
</search_scope>

<output_format>
Return:
- Authentication-related files and their purposes
- How sessions/tokens are managed
- Any security patterns or concerns noted
</output_format>
```

---

## Plan Agent Template

For implementation design and architectural decisions.

```
<context>
[Current state, constraints, requirements]
</context>

<task>
Design an implementation plan for: [feature/change description]
</task>

<considerations>
- [Constraint 1]
- [Constraint 2]
- [Pattern to follow or avoid]
</considerations>

<output_format>
Return:
- Recommended approach with rationale
- Files to modify
- Step-by-step implementation plan
- Potential risks or edge cases
</output_format>
```

### Example: Plan Feature Implementation

```
<context>
We need to add rate limiting to the API. Current setup uses Express.js
with no existing rate limiting. Must not break existing endpoints.
</context>

<task>
Design an implementation plan for adding rate limiting to all API endpoints.
</task>

<considerations>
- Must support different limits for authenticated vs anonymous users
- Should use Redis if available, fallback to in-memory
- Needs to return proper 429 responses with retry-after headers
</considerations>

<output_format>
Return:
- Recommended approach (middleware pattern, library choice)
- Configuration structure
- Files to create/modify
- Implementation steps
- Testing strategy
</output_format>
```

---

## Bash Agent Template

For command execution and terminal operations.

```
<task>
[Command or operation to perform]
</task>

<working_directory>
[Path where commands should run]
</working_directory>

<constraints>
- [Any limitations or safety requirements]
</constraints>
```

### Example: Git Operations

```
<task>
Check the git status and recent commit history for the project.
</task>

<working_directory>
/Users/adambutterfield/code/wellness-app
</working_directory>

<constraints>
- Read-only operations only
- Do not modify any files or make commits
</constraints>
```

---

## Parallel Tasks Pattern

Launch multiple independent agents simultaneously.

```
Launch these tasks in parallel (single message, multiple Task tool calls):

**Task 1: [Agent Type]**
<task>[Task 1 description]</task>
<output_format>[Format 1]</output_format>

**Task 2: [Agent Type]**
<task>[Task 2 description]</task>
<output_format>[Format 2]</output_format>

**Task 3: [Agent Type]**
<task>[Task 3 description]</task>
<output_format>[Format 3]</output_format>
```

### Example: Multi-Area Exploration

```
Launch these tasks in parallel:

**Task 1: Explore**
<task>Find all API endpoint definitions in src/api/</task>
<output_format>List of endpoints with HTTP methods and paths</output_format>

**Task 2: Explore**
<task>Find all database models in src/models/</task>
<output_format>List of models with their key fields</output_format>

**Task 3: Explore**
<task>Find test patterns in tests/</task>
<output_format>Testing framework used and example patterns</output_format>
```

---

## Sequential/Dependent Tasks Pattern

When later tasks depend on earlier results.

```
**Phase 1: Research**
Agent type: Explore
<task>[Research task]</task>
<output_format>[What to return]</output_format>

[WAIT FOR RESULTS]

**Phase 2: Design** (uses Phase 1 results)
Agent type: Plan
<context>
Based on the research findings:
[Include relevant results from Phase 1]
</context>
<task>[Design task using research]</task>

[WAIT FOR RESULTS]

**Phase 3: Implement** (uses Phase 2 plan)
Agent type: general-purpose
<context>
Following the approved plan:
[Include plan from Phase 2]
</context>
<task>[Implementation task]</task>
```

---

## General-Purpose Agent Template

For complex tasks requiring all tools (read, write, edit, bash).

```
<context>
[Full background needed for the task]
Project: [path]
Files involved: [list]
Constraints: [any limitations]
</context>

<task>
[Complete description of what to accomplish]
</task>

<approach>
[Suggested approach or steps to follow]
</approach>

<output_format>
[What to return when done]
</output_format>

<verification>
[How to verify the task was completed correctly]
</verification>
```

### Example: Implement Feature

```
<context>
Project: /Users/adambutterfield/code/wellness-app
Files to modify: src/api/users.ts, src/models/user.ts
Existing pattern: Follow the pattern in src/api/posts.ts
</context>

<task>
Add a "soft delete" feature to the user model. Users should have a
deletedAt timestamp field. The GET /users endpoint should exclude
soft-deleted users by default.
</task>

<approach>
1. Add deletedAt field to user model
2. Update GET /users query to filter deleted users
3. Add DELETE /users/:id endpoint that sets deletedAt
4. Add GET /users?includeDeleted=true option for admins
</approach>

<output_format>
Return:
- Summary of changes made
- Files modified with key changes
- Any issues encountered
</output_format>

<verification>
After implementation, verify by:
- Running existing user tests
- Manually testing the delete endpoint
</verification>
```

---

## Content Generation Template (Agent Returns, Main Writes)

When a custom agent lacks Write access but needs to produce file content. The agent generates content and returns it; the main agent does the actual writes.

```
<context>
[Background and constraints]
Output files: [where the content should eventually be written]
</context>

<task>
Generate the content for [description].
Do NOT write files directly - return the content in your response.
</task>

<output_format>
Return as structured data:

<file path="path/to/file1.ts">
[file content here]
</file>

<file path="path/to/file2.ts">
[file content here]
</file>

Include a summary of what was generated.
</output_format>
```

### Example: Generate Test Files

```
<context>
Generating unit tests for the size calculation module.
Test framework: Vitest
Output files: src/utils/__tests__/size-calc.test.ts
</context>

<task>
Analyze src/utils/size-calc.ts and generate comprehensive unit tests.
Do NOT write files directly - return the test content.
</task>

<output_format>
Return as:

<file path="src/utils/__tests__/size-calc.test.ts">
[test file content]
</file>

Summary: [what's covered, edge cases tested]
</output_format>
```

**Main agent follow-up:** After receiving the response, use Write tool to create the files.

---

## Batch File Transformation Template

When applying the same transformation to multiple files.

```
<context>
Files: [list of files or directory pattern]
Template/Format: [the structure each file should follow]
Example: [reference to an already-transformed file if available]
</context>

<task>
Transform each file to match the template format:
- [Specific change 1]
- [Specific change 2]
- [Specific change 3]
</task>

<output_format>
Return:
- Summary of files transformed
- Any files that couldn't be transformed and why
- Issues or edge cases encountered
</output_format>
```

### Example: Transform Rule Files

```
<context>
Files: /path/to/rules/*.md (44 files)
Template: See async-parallel.md for the target format
Key changes:
- Remove `tags` and `impactDescription` from frontmatter
- Add Review/Writing/Technique blockquote
- Rename Incorrect→Fail, Correct→Pass with inline comments
</context>

<task>
Transform each rule file to the new format, preserving the code
examples but restructuring sections.
</task>

<output_format>
Return:
- Count of files transformed
- Any files with unusual structure that needed special handling
</output_format>
```
