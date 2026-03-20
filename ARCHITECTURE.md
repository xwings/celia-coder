# Architecture

Celia Coder is a single skill that guides AI coding agents to write and review code with strong design sensibility. One set of design principles governs both modes — the standards applied when reviewing code are the same standards followed when writing it.

## Core Concepts

### Skill

The skill is defined in `SKILL.md` at the project root. It uses YAML frontmatter for metadata and markdown for behavior. The argument determines the mode:

- **Write mode** — argument describes work to do (feature, bug fix, refactor). The skill explores, designs, implements, and self-reviews against the principles.
- **Review mode** — argument points to existing code (file, commit, branch range, PR, or no argument for working changes). The skill analyzes the code against the principles and reports issues.

**Frontmatter fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `name` | yes | Short identifier used to invoke the skill |
| `description` | yes | One-line summary shown in skill listings |
| `argument-hint` | no | Usage hint displayed to the user |
| `allowed-tools` | yes | Tools the agent may use while executing the skill |

### Principles

Principle files are design heuristics the skill loads before writing or reviewing. Each file covers one dimension of software quality and contains:

- **Description** — what the principle is and why it matters
- **Core Principles** — the key ideas and guidelines
- **Red Flags** — concrete symptoms to look for in code
- **Issue Tags** — standardized labels for reporting problems

Both modes load the same principle files. This guarantees consistency — the write process self-reviews against the same standards the review process checks.

## Directory Structure

```
celia-coder/
├── ARCHITECTURE.md      # This file
├── SKILL.md             # The skill (write + review)
└── principles/          # Design principles (loaded by both modes)
    ├── abstraction-layers.md
    ├── cohesion-separation.md
    ├── documentation.md
    ├── error-handling.md
    ├── information-hiding.md
    ├── module-depth.md
    ├── naming-obviousness.md
    └── strategic-design.md
```

## Processes

### Write

1. **Understand** — Clarify what needs to be built or changed. Identify the goal, constraints, and scope.
2. **Explore** — Read relevant existing code to understand the current design, conventions, and integration points.
3. **Design** — Plan the approach informed by the principles. Decide on module boundaries, interfaces, error handling strategy, and naming.
4. **Implement** — Write the code, applying the principles to every design decision.
5. **Self-review** — Evaluate the output against the principles. Fix issues before presenting the result.

### Review

1. **Resolve input** — Determine what to review from the argument (staged changes, file, commit, branch range, or PR).
2. **Analyze** — Examine the code through each principle's lens, looking for red flags.
3. **Rank** — Order findings by severity based on how much complexity they introduce over time.
4. **Output** — Present the top issues (roughly 5) in the standardized format with tags, descriptions, and actionable suggestions.

## Extending

### Adding a principle

1. Create `principles/new-principle.md` following the existing format (Description, Core Principles, Red Flags, Issue Tags).
2. Add a reference line to the Design Principles section in `SKILL.md` so the skill knows to load it.
