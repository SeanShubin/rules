# Code Quality Rules

Architectural and code quality rule sets for different development paradigms. Rules are loaded on-demand when you ask Claude Code to check code, not at startup.

## Available Rule Sets

| Name         | Repository | Description                                                                                       |
|--------------|------------|---------------------------------------------------------------------------------------------------|
| oop          | [rules-oop](https://github.com/SeanShubin/rules-oop) | OOP / JVM business applications (Kotlin, Java, TypeScript). Dependency injection, composition roots, package hierarchy, testability through interfaces. |
| rust-bevy    | [rules-rust-bevy](https://github.com/SeanShubin/rules-rust-bevy) | Rust / Bevy game development with ECS. System parameter dependencies, typed events, plugin cohesion, module hierarchy, World-based testing. |

## Why On-Demand Loading?

**Problem:** Loading rules at startup (~200k+ characters) causes:
- Performance warnings for large files
- High context cost every session
- Rules still require explicit invocation to be applied (fundamental AI limitation)

**Solution:** Load rules only when explicitly requested:
- Minimal startup context (~1k characters)
- No performance warnings
- Same explicit invocation requirement, but honest about it

**Key insight:** AI assistants do not automatically apply rules in context. You must explicitly say "check against rules" whether rules are loaded at startup or on-demand. Since explicit invocation is required either way, on-demand loading is optimal.

**Decision rationale:**
1. Rules in memory → AI doesn't automatically apply them (observed behavior, documented in `tooling-and-ai.md`)
2. Explicit invocation required regardless (documented AI limitation)
3. If explicit invocation needed anyway → no benefit to paying context cost
4. Therefore → on-demand loading is optimal (same behavior, better performance)

## Architecture

The system has three layers:

### 1. Global Registry (`~/.claude/CLAUDE.md`)
Contains a registry mapping ruleset names to **local directories on your machine**, plus behavioral instructions for on-demand loading.

**Important:** This file is local to your machine and should NOT be committed to version control. Paths are machine-specific.

### 2. Project Declaration (`.claude/CLAUDE.md` in each project)
Declares which ruleset the project uses with a single line:
```markdown
ruleset: <name>
```

No rule content is loaded at startup. The declaration is a simple string reference.

### 3. Rule Files (separate repositories)
Rule sets live in their own Git repositories:
- `rules-oop/` - OOP rule set with `quick-reference.md` and detailed rule files
- `rules-rust-bevy/` - Rust/Bevy rule set with `quick-reference.md` and detailed rule files

Each repository must contain:
- `quick-reference.md` at the root (violation checklist)
- Individual rule files with detailed guidance

## Initial Setup

### Step 1: Clone Rule Repositories

Clone the rule sets you want to use to a location on your machine:

```bash
cd ~/github.com/YourName/  # Or wherever you keep repos
git clone https://github.com/SeanShubin/rules-oop.git
git clone https://github.com/SeanShubin/rules-rust-bevy.git  # Optional
```

### Step 2: Create Global Registry

Create `~/.claude/CLAUDE.md` with the following template (adjust paths to match where you cloned the repositories):

```markdown
# Personal Coding Standards

## Rule Set Registry

This registry maps rule set names to their local directories. Rules are loaded on-demand only when you ask me to check code.

| Name      | Path |
|-----------|------|
| oop       | /Users/yourname/github.com/YourName/rules-oop |
| rust-bevy | /Users/yourname/github.com/YourName/rules-rust-bevy |

## On-Demand Loading Behavior

When you say "check against rules" or similar:
1. I read the project's `ruleset:` declaration from `.claude/CLAUDE.md`
2. I look up the path in the registry above
3. I read `quick-reference.md` first for the violation checklist
4. I consult individual rule files as needed for detailed guidance

I do NOT load rules at startup. I only load them when you explicitly ask me to check code.

## Default Principles (Always Active)

These general principles apply regardless of ruleset:

### Tooling and Static Analysis
- Static analysis provides objective measurement; zero violations is optimal
- AI helps achieve zero, never justifies non-zero
- No ignore lists or subjective exceptions
- When violations occur: evaluate, restructure, or petition tool maintainers for refinement

### Explicit Rule Invocation Required
As documented in the rules themselves, I do not automatically apply loaded rules. You must explicitly invoke them:
- ❌ Won't work: "Review this code" (I'll use generic training patterns)
- ✅ Will work: "Check this code against rules" (I'll load and apply the ruleset)

This is a fundamental AI limitation - rules in context are reference material, not automatic governing standards. Explicit invocation is always required.
```

**Important:** Replace `/Users/yourname/github.com/YourName/` with your actual local paths.

### Step 3: Declare Ruleset in Projects

In each project that should use rules, create `.claude/CLAUDE.md`:

```bash
cd /path/to/your/project
echo "ruleset: oop" > .claude/CLAUDE.md
```

Or for Rust/Bevy projects:
```bash
echo "ruleset: rust-bevy" > .claude/CLAUDE.md
```

### Step 4: Verify Setup

Start Claude Code in a project and verify:

```bash
claude
```

Then try:
1. Say: "What ruleset does this project use?" - Should see "oop" or "rust-bevy"
2. Say: "Check against rules" - Should load rules on-demand
3. Verify no performance warnings about large files

## Usage Workflow

### When Working in a Project

**General coding (rules not loaded):**
```
> Help me implement user authentication
> Add error handling to this function
> Refactor this code for clarity
```

Claude responds using general software engineering knowledge without loading rules.

**Rule-based checking (rules loaded on-demand):**
```
> Check this code against rules
> Review this implementation according to architectural rules
> Does this violate any of our rules?
```

Claude loads the appropriate ruleset and checks for violations.

### Explicit Invocation Phrases

These phrases trigger rule loading:
- "Check against rules"
- "Check this code against rules"
- "Review according to architectural rules"
- "Does this violate our rules?"
- "Validate this against the ruleset"

**Important:** Generic phrases like "review this code" will NOT load rules. You must explicitly mention "rules" or "architectural rules" in your prompt.

## Adding a New Rule Set

To add a new rule set (e.g., `python-ml` for Python machine learning projects):

### 1. Create the Rule Repository

Create a new Git repository with:
- `quick-reference.md` - Violation checklist
- Individual rule files (e.g., `coupling-and-cohesion.md`, `dependency-injection.md`)
- `README.md` explaining the rules

### 2. Clone Locally

```bash
git clone https://github.com/YourOrg/rules-python-ml.git ~/path/to/rules-python-ml
```

### 3. Update Global Registry

Add a row to the registry table in your `~/.claude/CLAUDE.md`:

```markdown
| Name      | Path |
|-----------|------|
| oop       | /Users/yourname/github.com/YourName/rules-oop |
| rust-bevy | /Users/yourname/github.com/YourName/rules-rust-bevy |
| python-ml | /Users/yourname/github.com/YourName/rules-python-ml |
```

### 4. Document in This File

Submit a PR to update the "Available Rule Sets" table in this README with:
- Name of the ruleset
- Link to the GitHub repository
- Description of what paradigm/domain it covers

### 5. Use in Projects

In Python ML projects, create `.claude/CLAUDE.md`:
```markdown
ruleset: python-ml
```

## Renaming a Rule Set

If you need to rename a ruleset (e.g., `oop` → `oop-kotlin`):

1. **Update the global registry** in `~/.claude/CLAUDE.md`:
   ```markdown
   | Name        | Path |
   |-------------|------|
   | oop-kotlin  | /Users/yourname/github.com/YourName/rules-oop |
   ```

2. **Update all project declarations** - Find projects using the old name:
   ```bash
   find ~ -name ".claude" -type d -exec grep -l "ruleset: oop" {}/.CLAUDE.md \;
   ```
   Update each to use the new name.

3. **Update this README** - Submit PR updating the "Available Rule Sets" table.

## Troubleshooting

### Rules Not Loading

**Symptom:** Say "check against rules" but get generic advice.

**Diagnosis:**
1. Check project has `.claude/CLAUDE.md` with `ruleset:` declaration
2. Check ruleset name matches registry in `~/.claude/CLAUDE.md`
3. Check path in registry points to actual directory with rule files
4. Check `quick-reference.md` exists at the root of the rule directory

**Fix:**
```bash
# Verify project config exists
cat .claude/CLAUDE.md

# Verify path in global registry exists
ls -la ~/path/from/registry/

# Verify quick-reference.md exists
ls ~/path/from/registry/quick-reference.md
```

### Performance Warnings About Large Files

**Symptom:** Warning like "Large file will impact performance (41.5k chars > 40.0k)"

**Cause:** You may have old `@rules/` references in `~/.claude/CLAUDE.md` loading rules at startup.

**Fix:** Ensure your global config uses the **registry pattern** (see Step 2 above), not `@rules/` references.

### AI Says "I Don't See a Ruleset Declaration"

**Symptom:** Claude can't find the `ruleset:` line.

**Cause:** Project `.claude/CLAUDE.md` may not exist or has wrong content.

**Fix:**
```bash
cd /path/to/project
echo "ruleset: oop" > .claude/CLAUDE.md
cat .claude/CLAUDE.md  # Verify it contains: ruleset: oop
```

### Wrong Rules Loading

**Symptom:** Claude loads OOP rules when you expected Rust/Bevy rules.

**Cause:** Project declaration doesn't match intended ruleset.

**Fix:**
```bash
# Check what's declared
cat .claude/CLAUDE.md

# Update if wrong
echo "ruleset: rust-bevy" > .claude/CLAUDE.md
```

## Philosophy

All rule sets share core principles:
- **Group code that changes together** - Maintain high cohesion and low coupling
- **Make dependencies explicit and testable** - Dependency injection and composition
- **Use events for cross-domain communication** - Explicit event interfaces
- **Organize by domain, not technical layer** - Feature-based structure
- **Zero violations is the standard** - No ignore lists, fix problems or refine tools

The rules differ in **how** these principles are expressed:
- **OOP (Java/Kotlin/TypeScript):** Constructor injection, composition roots, package hierarchy
- **Rust/Bevy (ECS):** System parameters, resources, typed events, plugin cohesion

Different paradigms require different implementations of the same underlying principles. OOP and ECS have fundamentally different composition mechanisms, performance constraints, and idioms, so rules are tailored to each paradigm while maintaining philosophical consistency.

## For AI Assistants

When a user says "check against rules":
1. Read the project's `.claude/CLAUDE.md` for the `ruleset:` declaration
2. Look up the path in your global registry (in `~/.claude/CLAUDE.md`)
3. Read `quick-reference.md` from that path first
4. Consult individual rule files as needed based on the code being checked
5. Provide analysis citing specific rule violations with file references (e.g., "coupling-and-cohesion.md:42")

Remember: You do not automatically apply rules. They must be explicitly invoked by the user.
