# Code Quality Rules

Architectural and code quality rules for different development paradigms. Each rule set preserves the same spirit of disciplined, maintainable code while respecting the different design constraints of its target paradigm.

## Rule Sets

### [OOP / JVM Business Applications](https://github.com/SeanShubin/rules-oop)

For projects using object-oriented languages (Kotlin, Java, TypeScript) with dependency injection, composition roots, and package hierarchy. Targets business application architecture: cloud services, backend APIs, frontend applications.

Key concepts: constructor injection, staged dependency injection, event interfaces, package-by-domain, testability through interfaces.

### [Rust / Bevy Game Development](https://github.com/SeanShubin/rules-rust-bevy)

For projects using Rust with the Bevy game engine and ECS (Entity Component System) paradigm. Targets game development architecture: systems, components, resources, plugins.

Key concepts: system parameter dependencies, typed events, plugin cohesion, module hierarchy, World-based testing.

## Philosophy

Both rule sets share the same core principles:
- Group code that changes together
- Make dependencies explicit and testable
- Use events for cross-domain communication
- Organize by domain, not technical layer
- Zero violations is the standard

The rules differ in **how** these principles are expressed, because OOP and ECS are fundamentally different paradigms with different composition mechanisms, different performance constraints, and different idioms.

## Using with Claude Code

Each rule set contains its own setup instructions:
- [OOP setup](https://github.com/SeanShubin/rules-oop/blob/master/claude-code-setup.md)
- [Rust/Bevy setup](https://github.com/SeanShubin/rules-rust-bevy) (see README for module references)
