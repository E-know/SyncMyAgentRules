# AGENTS.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

- Verify the requested behavior and relevant regressions with appropriate checks. Where applicable, check both required and forbidden outcomes; a successful build alone may not prove correctness.
- When a check fails, inspect the evidence and distinguish implementation defects from environment or test setup failures before changing code.
- Use each result to guide the next hypothesis or action. Do not repeat a failed approach without new evidence or a justified change.
- Do not weaken or remove valid checks merely to obtain a pass. Explain and resolve conflicts between requirements and checks.
- For long-running or handed-off work, preserve a concise record of verified facts, unresolved questions, failed attempts, and next steps; review it when resuming.
- Continue while meaningful progress is possible. If blocked or a user-defined limit is reached, report what is complete, what was actually verified, what remains unverified, and what is needed to proceed.

## 5. Subagent Delegation

- Subagents may create and delegate work to their own subagents when useful, within the available runtime limits.
- Each parent agent should select its subagents' models and reasoning effort based on the needs of the assigned tasks, using supported options and respecting runtime constraints.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

반드시 답변은 한국어로 진행할 것

## Preferred language for executable code

When Codex writes and runs new scripts or programs for a task, prefer Rust over Python whenever practical. Follow the applicable requirements when an existing project, a required tool or skill, or an explicit user request requires another language.
