# Code Quality Rules

Architectural and code quality rule sets for different development paradigms. Rules are loaded on-demand when you ask Claude Code to check code, not at startup.

## Available Rule Sets

| Name         | Description                                                                                       |
|--------------|---------------------------------------------------------------------------------------------------|
| oop          | OOP / JVM business applications (Kotlin, Java, TypeScript). Dependency injection, composition roots, package hierarchy, testability through interfaces. |
| rust-bevy    | Rust / Bevy game development with ECS. System parameter dependencies, typed events, plugin cohesion, module hierarchy, World-based testing. |

## How It Works

The system has three layers:

1. **Global `~/.claude/CLAUDE.md`** contains a ruleset registry mapping names to local directories on your machine, plus behavioral instructions for on-demand loading.

2. **Project `.claude/CLAUDE.md`** declares which ruleset the project uses with a single line like `ruleset: oop`. No rule content is loaded at startup.

3. **Rule files** live in their own repositories (e.g., `rules-oop/`, `rules-rust-bevy/`). They are only read when you say "check against rules".

When you say "check against rules" in a project:
- Claude reads the project's `ruleset:` declaration
- Looks up the path in the global registry
- Reads `quick-reference.md` first for the violation checklist
- Consults individual rule files as needed

## Adding a New Rule Set

1. Create a rule set repository with rule files and a `quick-reference.md`.
2. Add a row to the registry table in `~/.claude/CLAUDE.md` mapping the name to the local path.
3. Add a row to the table in this file documenting the name and description.

## Renaming a Rule Set

1. Update the name in the registry table in `~/.claude/CLAUDE.md`.
2. Update the `ruleset:` line in any project `.claude/CLAUDE.md` files that use it.
3. Update the name in this file.

## Per-Project Setup

In your project's `.claude/CLAUDE.md`, add:

```
ruleset: <name>
```

Where `<name>` matches an entry in the global registry. That's it. No `@rules/` references, no symlinks needed.

## Philosophy

Both rule sets share the same core principles:
- Group code that changes together
- Make dependencies explicit and testable
- Use events for cross-domain communication
- Organize by domain, not technical layer
- Zero violations is the standard

The rules differ in how these principles are expressed, because OOP and ECS are fundamentally different paradigms with different composition mechanisms, different performance constraints, and different idioms.
