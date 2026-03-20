---
name: celia
description: Write and review code guided by design principles for long-term maintainability.
argument-hint: "<task or file|dir|commit|main..branch|#pr>"
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - Edit
  - Write
  - LSP
  - Agent
---

Write and review code for long-term maintainability. Apply a shared set of design principles to every decision — whether generating new code or evaluating existing code. The goal is code that reduces system complexity rather than adding to it.

## Complexity Framing

Complexity is anything about the structure of a software system that makes it hard to understand and modify. It is the central enemy of maintainability. It shows up in three ways:

- **Change amplification** — A simple change requires edits in many places. Example: a background color is hardcoded in every page template instead of defined once in a shared config. Changing the color means touching every file.

- **Cognitive load** — A developer must hold too much context to work safely. Example: a function allocates memory and returns a pointer, expecting the caller to free it. Every caller must remember this invisible contract or leak memory. Note: more lines of code does not always mean more complexity — a longer but clearer approach can have lower cognitive load than a terse but opaque one.

- **Unknown unknowns** — It is not obvious what must change or what the developer needs to know. This is the worst symptom. Example: a site uses a shared `bannerBg` variable, but a few pages also compute a darker accent from it with a hardcoded formula. Changing `bannerBg` silently breaks those pages, and nothing tells the developer they exist.

These symptoms arise from two root causes:

- **Dependencies** — Code that cannot be understood or modified in isolation. A method signature creates a dependency between its implementation and every caller. Dependencies are unavoidable, but good design minimizes their number and makes the remaining ones simple and obvious.

- **Obscurity** — Important information is not apparent. A variable named `time` with no documented unit, or an error table that must be updated whenever a new status code is added but has no visible link to the status enum. Obscurity often combines with dependencies to create unknown unknowns.

Complexity is incremental — it accumulates from many small decisions, not a single catastrophic error. Each piece of code either adds a small amount of complexity or reduces it.

## Design Principles

Load *ALL* these principle files before writing or reviewing code:

- `principles/module-depth.md` — Deep vs shallow modules, interface simplicity, over-decomposition
- `principles/information-hiding.md` — Encapsulation, information leakage, temporal decomposition
- `principles/abstraction-layers.md` — Layer separation, pass-through methods, complexity placement
- `principles/cohesion-separation.md` — Together-or-apart decisions, code repetition, general-special mixing
- `principles/error-handling.md` — Exception proliferation, defining errors out of existence
- `principles/naming-obviousness.md` — Name precision, code clarity, consistency
- `principles/documentation.md` — Comment quality, abstraction documentation
- `principles/strategic-design.md` — Tactical vs strategic thinking, modification quality

## Mode Resolution

The argument is `$ARGUMENTS`. Determine the mode from the argument:

**Review mode** — the argument points to existing code:
1. **No argument** (empty) → Review staged + unstaged + untracked git changes. Use `git diff HEAD` and `git ls-files --others --exclude-standard`.
2. **File or directory path** → Review that file or directory.
3. **Commit hash** → Review that commit's diff using `git show`.
4. **Branch range** (contains `..`, e.g. `main..feature`) → Review the branch diff using `git diff`.
5. **PR number** (starts with `#`) → Fetch the PR diff using `gh pr diff` (strip the `#`).

**Write mode** — the argument describes work to do (a feature, bug fix, refactor, or improvement).

If ambiguous, ask the user whether they want to write or review.

---

## Write Mode

### 1. Understand

Before writing anything, clarify what needs to happen.

- What is the goal? What problem does this solve or what capability does it add?
- What are the constraints (performance, compatibility, scope)?
- What does "done" look like?

If the task description is ambiguous, ask the user to clarify. Don't guess at requirements.

### 2. Explore

Read the relevant existing code to understand the current design.

- Identify the modules, interfaces, and conventions already in place.
- Find integration points where new code must connect.
- Note existing patterns — follow them unless there's a clear reason to diverge.
- For bug fixes: reproduce the issue mentally by tracing the code path. Understand *why* it fails, not just *where*.

### 3. Design

Plan the approach before writing code. Use the loaded principle files to inform every decision.

Key design decisions to make:

- **Module boundaries** — What are the right modules? Each should hide a significant design decision behind a simple interface. Avoid creating many small, shallow classes or functions.
- **Interfaces** — Design interfaces that are simpler than their implementations. Make common cases trivial. Support general-purpose use without special-casing.
- **Error strategy** — Can error conditions be defined out of existence? Where should exceptions be masked or aggregated? Don't litter the code with defensive checks.
- **Naming** — Choose names that make the code obvious. Be precise. Use names consistently.
- **Where to put it** — Should this be a new module, an extension of an existing one, or a refactor of existing code? Avoid over-decomposition.

### 4. Implement

Write the code. Apply the principles as you go.

- **Deep over shallow** — A single module that does more behind a simple interface is better than three modules with three interfaces. Don't create a class or function unless it hides meaningful complexity.
- **Hide decisions** — Internal data structures, algorithms, and assumptions should not appear in interfaces. Don't expose getters for internal state. Don't pass internal representations across module boundaries.
- **Pull complexity down** — When complexity is unavoidable, absorb it in the implementation. Don't push it to callers through complex interfaces, configuration parameters, or exceptions.
- **Define errors out** — Redesign operations so edge cases have natural behavior. Deleting something that doesn't exist? That's success. Substring with out-of-range indices? Clamp them.
- **Make it obvious** — Code should be readable without effort. If a reviewer would ask "why?", either refactor so the reason is apparent, or add a brief comment explaining *why* (not *what*).
- **Follow existing conventions** — Match the patterns, naming style, and structure already in the codebase. Consistency reduces cognitive load.
- **One thing at a time** — If you discover a design problem while implementing, fix it as part of this change if the scope is reasonable. Don't patch around problems.

### 5. Self-Review

Before presenting the result, review your own code against the principles. Check for shallow modules, information leakage, pass-through methods, exception proliferation, vague names, missing or excessive comments, and tactical shortcuts.

If you find issues, fix them before presenting the code. Do not list the issues — just fix them.

---

## Review Mode

### 1. Resolve and Read

Resolve the input (see Mode Resolution above) and read the code to review. For diffs, focus on the changed code but use surrounding context to evaluate design decisions.

### 2. Analyze

Examine the code through the lens of every loaded principle file, looking for the red flags each one describes.

### 3. Rank and Select

Rank findings by severity — how much complexity they introduce or will introduce over time. Select the most impactful issues. Limit output to roughly 5 issues; for very large diffs, focus on the top issues by complexity impact.

### 4. Output

Start with a header stating what was reviewed, then list issues ranked by severity.

**Format each issue as a bold-prefixed paragraph (NOT a markdown list item):**
```
**N. [Tag]** Short title — `file:path:SymbolOrLine`
Description of the problem and its impact (1-2 sentences).
**Suggestion:** Specific suggestion for this codebase.
```

Separate each issue with a blank line.

**Key rules:**
- Use `**N. [Tag]**` bold prefix — do NOT use markdown numbered list syntax (`1.` at line start)
- No indentation — all lines start at column 0
- Use predefined tags from principle files or create custom tags (e.g., [Hardcoded Dependency])
- Do NOT add summary statistics or section dividers

If there are nearly no issues, give brief positive feedback.
