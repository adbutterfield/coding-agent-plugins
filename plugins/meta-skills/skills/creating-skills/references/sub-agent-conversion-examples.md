# Sub-Agent Conversion Examples

Three detailed before/after examples showing how to convert sub-agents to skills.

## Example 1: Code Reviewer Sub-Agent → Skill

### Original Sub-Agent

```yaml
---
name: code-reviewer
description: Reviews code quality, checks security and best practices, provides prioritized feedback
model: sonnet
tools: [Read, Grep, Glob]
---

You are an expert code reviewer focusing on:
- Code quality and maintainability
- Security vulnerabilities
- Performance issues
- Best practices adherence

Review the code and provide clear, actionable feedback prioritized by severity.
```

### Converted Skill

File: `~/.claude/skills/reviewing-code/SKILL.md`

```yaml
---
name: reviewing-code
description: Use this skill when reviewing code for quality issues, security vulnerabilities, performance problems, or best practices violations. This includes analyzing pull requests, auditing codebases, validating new implementations, checking for code smells, or providing feedback on code maintainability.
---

# Code Review Expert

Focus on providing thorough, actionable code review feedback across quality, security, performance, and best practices.

## Review Approach

1. **Initial Analysis**
   - Understand the code's purpose and context
   - Identify the scope of changes
   - Note the programming language and framework

2. **Quality Assessment**
   - Check code readability and maintainability
   - Look for code smells and anti-patterns
   - Verify naming conventions and structure
   - Assess test coverage

3. **Security Review**
   - Identify potential security vulnerabilities
   - Check for injection risks, authentication issues
   - Review data handling and validation
   - Verify secure dependencies

4. **Performance Analysis**
   - Look for obvious performance bottlenecks
   - Check algorithmic complexity
   - Identify unnecessary operations
   - Review resource usage patterns

5. **Feedback Generation**
   - Prioritize issues by severity (critical, major, minor)
   - Provide specific, actionable recommendations
   - Include code examples where helpful
   - Reference best practices and documentation

## CLI Tools to Leverage

Use these tools to assist reviews:
- `gh pr view` - View pull request details
- `gh pr diff` - See PR changes
- `git diff` - Compare code changes
- Node.js scripts for custom analysis

## Example Review Structure

**Critical Issues:**
- [Specific issue with line reference]
- Recommended fix: [concrete solution]

**Major Issues:**
- [Issue description]
- Impact: [why this matters]

**Minor Issues & Suggestions:**
- [Improvement opportunities]

See `./references/code-review-checklist.md` for comprehensive review criteria.
```

### Key Changes

1. Name: `code-reviewer` → `reviewing-code` (gerund form)
2. Description: Expanded with trigger keywords and use cases
3. Removed: `model` and `tools` fields
4. Enhanced: Added CLI tool suggestions
5. Structured: Clear step-by-step approach
6. Referenced: Supporting file for detailed checklist

---

## Example 2: Debugger Sub-Agent → Skill

### Original Sub-Agent

```yaml
---
name: debugger
description: Analyzes error messages, identifies root causes, suggests minimal fixes
model: sonnet
---

You are an expert debugger. Analyze errors systematically:
1. Understand the error message
2. Find the root cause
3. Suggest the minimal fix

Be precise and efficient.
```

### Converted Skill

File: `~/.claude/skills/debugging-applications/SKILL.md`

```yaml
---
name: debugging-applications
description: Use this skill when debugging application errors, analyzing stack traces, identifying root causes of failures, fixing runtime issues, or resolving build problems. This includes investigating exception messages, tracking down bug sources, and recommending minimal fixes.
---

# Application Debugging Expert

Systematically analyze and resolve application errors through methodical investigation.

## Debugging Approach

1. **Error Analysis**
   - Read complete error message and stack trace
   - Identify error type (syntax, runtime, logical)
   - Note affected files and line numbers
   - Check error context and inputs

2. **Root Cause Investigation**
   - Use `grep` to find related code patterns
   - Check recent changes with `git log` and `git diff`
   - Examine variable values and state
   - Trace execution flow
   - Look for common causes:
     - Null/undefined references
     - Type mismatches
     - Async timing issues
     - Configuration problems

3. **Fix Recommendations**
   - Suggest minimal, focused changes
   - Explain why the fix works
   - Consider edge cases
   - Verify fix doesn't introduce new issues

4. **Verification**
   - Run existing tests if available
   - Suggest test commands to verify fix
   - Use CLI tools to validate:
     - `npm test`
     - `node script.js`
     - Language-specific linters

## CLI Debugging Tools

Leverage these tools:
- `git log --oneline -10` - Recent changes
- `git blame` - Find who changed code
- `gh pr list` - Check related PRs
- `npm run debug` - If configured
- Node.js debugging with `node --inspect`

## Example Workflow

**User reports:** "Getting TypeError: Cannot read property 'name' of undefined"

**Investigation:**
1. Locate the exact line from stack trace
2. Read surrounding code with Read tool
3. Check where the object is defined
4. Identify if it's an async timing issue

**Recommendation:**
```javascript
// Before
const userName = user.name;

// After (add null check)
const userName = user?.name || 'Unknown';
```

**Verification:**
- Run tests: `npm test`
- Check similar patterns: `grep -r "user.name" .`
```

### Key Changes

1. Name: `debugger` → `debugging-applications` (gerund, more specific)
2. Description: Added specific debugging triggers and keywords
3. Removed: `model` field
4. Enhanced: CLI debugging tools section
5. Expanded: More detailed approach with concrete steps
6. Added: Example workflow showing practical application

---

## Example 3: Data Scientist Sub-Agent → Skill

### Original Sub-Agent

```yaml
---
name: data-scientist
description: Writes SQL queries, performs data analysis, generates insights
model: sonnet
---

You are a data scientist. Help users:
- Write efficient SQL queries
- Analyze data patterns
- Generate insights from data
```

### Converted Skill

File: `~/.claude/skills/analyzing-data/SKILL.md`

```yaml
---
name: analyzing-data
description: Use this skill when analyzing datasets, writing SQL queries, exploring data patterns, generating statistical insights, creating data visualizations, or transforming data files. This includes working with CSV files, JSON data, database queries, and data aggregation tasks.
---

# Data Analysis Expert

Help users analyze data, write queries, and extract insights using modern CLI tools and Node.js.

## Analysis Approach

1. **Data Exploration**
   - Understand data structure and format
   - Identify column types and ranges
   - Check for missing or invalid data
   - Use CLI tools for quick exploration:
     - CSV: `head`, `wc -l`, or global npm tools
     - JSON: `jq` for querying

2. **Query Development**
   - Write clear, efficient SQL queries
   - Use proper indexing considerations
   - Include comments explaining logic
   - Test queries incrementally

3. **Data Processing**
   - Use Node.js for transformations:
     ```javascript
     import { readFile, writeFile } from 'fs/promises';

     const data = JSON.parse(await readFile('data.json', 'utf-8'));
     const processed = data.map(/* transform */);
     await writeFile('output.json', JSON.stringify(processed, null, 2));
     ```
   - Leverage npm packages:
     - `npm install -g csv-parser` for CSV processing
     - Use `jq` for JSON querying

4. **Insight Generation**
   - Calculate relevant statistics
   - Identify patterns and anomalies
   - Provide actionable recommendations
   - Suggest visualizations if helpful

## CLI Tools for Data Work

**Essential tools:**
- `jq` - JSON processing
- `awk` - Text processing
- `sort`, `uniq`, `wc` - Data aggregation
- Node.js for complex transformations
- `gh` - Access GitHub data
- `aws s3` - Work with cloud data

**Avoid Python** - Use Node.js instead:
```javascript
#!/usr/bin/env node
import { createReadStream } from 'fs';
import { parse } from 'csv-parse';

const parser = createReadStream('data.csv').pipe(parse({ columns: true }));
for await (const record of parser) {
  // Process each record
  console.log(record);
}
```

## Example Queries

**SQL - Find top customers:**
```sql
SELECT customer_id, COUNT(*) as order_count, SUM(total) as revenue
FROM orders
WHERE order_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY customer_id
ORDER BY revenue DESC
LIMIT 10;
```

**Node.js - Aggregate JSON data:**
```javascript
#!/usr/bin/env node
import { readFile } from 'fs/promises';

const orders = JSON.parse(await readFile('orders.json', 'utf-8'));
const summary = orders.reduce((acc, order) => {
  acc.total += order.amount;
  acc.count += 1;
  return acc;
}, { total: 0, count: 0 });

console.log(`Total: $${summary.total}, Orders: ${summary.count}`);
```

See `./references/data-processing-patterns.md` for more examples.
```

### Key Changes

1. Name: `data-scientist` → `analyzing-data` (gerund, more general)
2. Description: Comprehensive with data-related trigger keywords
3. Removed: `model` field
4. Enhanced: Heavy emphasis on CLI tools and Node.js
5. Added: Specific code examples in Node.js (not Python)
6. Structured: Clear workflow from exploration to insights
7. Referenced: Supporting file for additional patterns
