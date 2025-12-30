# Directives

> Layer 1: What to do

This folder contains **Standard Operating Procedures (SOPs)** written in Markdown. Each directive defines:

- **Goal**: What the directive accomplishes
- **Inputs**: What data/parameters are needed
- **Tools/Scripts**: Which execution scripts to use
- **Outputs**: What gets produced (files, API calls, deliverables)
- **Edge Cases**: Known gotchas, error handling, special conditions

## Directive Template

When creating a new directive, use this structure:

```markdown
# Directive Name

## Goal
What this directive accomplishes.

## Inputs
- Input 1: description
- Input 2: description

## Execution Steps
1. Step one (use `execution/script_name.py`)
2. Step two
3. Step three

## Outputs
- What gets produced

## Edge Cases
- Known issues, rate limits, timing constraints
- Error handling guidance

## Learnings
(Updated as the system learns)
- Date: What was discovered
```

## Key Principles

1. **Write like you're training a mid-level employee** — clear, actionable instructions
2. **Reference execution scripts** — don't reinvent, check `execution/` first
3. **Keep them living documents** — update with learnings as the system improves
4. **Don't create without asking** — directives are the instruction set, treat them carefully
