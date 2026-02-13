# Using These Rules with Claude Code

To ensure Claude Code consistently applies these rules when generating code in your projects, set up project memory.

## Quick Setup

Create `.claude/CLAUDE.md` in your project root that references these rules:

```markdown
# Code Quality Standards

See @rules/README.md for complete guidelines.

When generating or reviewing code, follow these rules in priority order:
1. @rules/coupling-and-cohesion.md
2. @rules/dependency-injection.md
3. @rules/abstraction-levels.md
4. @rules/package-hierarchy.md
5. @rules/anonymous-code.md
6. @rules/language-separation.md
7. @rules/free-floating-functions.md

Consult @rules/severity-guidance.md for severity calibration.
```

## Multi-Project Setup (Per-Project Linking)

To share these rules across multiple projects while keeping each project independent, symlink the shared rules into each project's `.claude` directory.

**In each project that wants to use these rules:**

1. Create a symlink to the shared rules:

```bash
cd /path/to/your/project
mkdir -p .claude/rules
ln -s /path/to/shared/ai/rules .claude/rules/shared-standards
```

Replace `/path/to/shared/ai/rules` with the actual path to where you cloned this rules repository.

2. Create `.claude/CLAUDE.md` in your project root:

```markdown
# Code Quality Standards

See @rules/shared-standards/README.md for complete guidelines.

When generating or reviewing code, follow these rules in priority order:
1. @rules/shared-standards/coupling-and-cohesion.md
2. @rules/shared-standards/dependency-injection.md
3. @rules/shared-standards/abstraction-levels.md
4. @rules/shared-standards/package-hierarchy.md
5. @rules/shared-standards/anonymous-code.md
6. @rules/shared-standards/language-separation.md
7. @rules/shared-standards/free-floating-functions.md

Consult @rules/shared-standards/severity-guidance.md for severity calibration.
```

This approach allows:
- Each project to reference the same shared rules
- Projects to be in different directories
- Version control to ignore the symlink (add `.claude/rules/` to `.gitignore`)
- Different projects to use different versions if needed (by pointing to different rule directories)

**Recommended `.gitignore` entry for each project:**

```gitignore
# Ignore symlinked shared rules (each developer points to their own location)
.claude/rules/
```

**Check in to version control:**
- `.claude/CLAUDE.md` - The reference to shared rules (everyone needs this)

**Don't check in:**
- `.claude/rules/shared-standards/` - The symlink (path is developer-specific)

## Global Setup (All Projects)

To apply these rules to ALL projects automatically, create `~/.claude/CLAUDE.md` in your home directory. Claude will load these rules for every project you work on.

Example `~/.claude/CLAUDE.md`:

```markdown
# Personal Coding Standards

These rules apply to all my projects.

## Architectural Rules

See @rules/shared-standards/README.md for complete guidelines.

I follow these code quality principles:
- @rules/shared-standards/coupling-and-cohesion.md
- @rules/shared-standards/dependency-injection.md
- @rules/shared-standards/abstraction-levels.md
- @rules/shared-standards/package-hierarchy.md
- @rules/shared-standards/anonymous-code.md
- @rules/shared-standards/language-separation.md
- @rules/shared-standards/free-floating-functions.md
```

Then symlink your rules:

```bash
mkdir -p ~/.claude/rules
ln -s /path/to/shared/ai/rules ~/.claude/rules/shared-standards
```

Replace `/path/to/shared/ai/rules` with the actual path to where you cloned this rules repository.

**Note:** Global setup applies to ALL projects. Use multi-project setup if you want selective adoption.

## Verification

After setup, verify Claude has loaded your rules:

```bash
claude /memory
```

This shows which memory files are loaded and in what order.

## How It Works

- `.claude/CLAUDE.md` is automatically loaded at the start of every session
- The `@rules/` references load those rule files into context
- Rules persist across sessions - no need to remind Claude
- Rules are checked into version control, so your team benefits too

## Documentation

For more details, see [Claude Code Memory Documentation](https://code.claude.com/docs/en/memory.md).
