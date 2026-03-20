# Celia Coder

A skill that teaches AI coding agents to write and review code for long-term maintainability. One set of design principles governs both — the standards applied when reviewing are the same standards followed when writing.

## How It Works

An AI agent loads `SKILL.md`, which instructs it to load 8 design principle files from `principles/`. The argument determines the mode:

- **Write** — `"add a caching layer to the database module"` — the agent explores the codebase, designs with the principles in mind, implements, and self-reviews before presenting the result.
- **Review** — `src/auth/` or `main..feature` or `#42` — the agent analyzes the code against every principle and reports the top issues ranked by complexity impact.

## Principles

| Principle | Governs |
|-----------|---------|
| Module Depth | Deep modules with simple interfaces over many shallow ones |
| Information Hiding | Encapsulating design decisions, avoiding leakage between modules |
| Abstraction Layers | Layer separation, pass-through elimination, complexity placement |
| Cohesion & Separation | Together-or-apart decisions, splitting for abstractions not length |
| Error Handling | Defining errors out of existence, minimizing exception surface |
| Naming & Obviousness | Precise names, code that reads without effort |
| Documentation | Documenting what and why, not how |
| Strategic Design | Investing in design, resisting tactical shortcuts |

## Structure

```
SKILL.md             # The skill definition (write + review)
principles/          # Design principles loaded by the skill
├── module-depth.md
├── information-hiding.md
├── abstraction-layers.md
├── cohesion-separation.md
├── error-handling.md
├── naming-obviousness.md
├── documentation.md
└── strategic-design.md
```

## Adding a Principle

1. Create `principles/your-principle.md` with four sections: Description, Core Principles, Red Flags, Issue Tags.
2. Add a reference line to the Design Principles section in `SKILL.md`.
