---
name: commit
description: Create and execute commit messages for code changes
allowed-tools: Read, Bash(git *), Glob, Grep
context: fork
model: Haiku
---


Analyze the changes and create appropriate commit messages in conventional commits format, then execute the commits. If the changes involve multiple unrelated modifications, split them into separate commits as needed.

## Commit Message Format
```
<emoji> <type>: <short description>

<optional body>
```

## Type and Emoji Mapping

| Type | Emoji | Description |
|------|-------|-------------|
| feat | ✨ | New feature |
| fix | 🐛 | Bug fix |
| refactor | ♻️ | Code refactoring (no feature change) |
| docs | 📝 | Documentation changes |
| test | ✅ | Adding or updating tests |
| style | 💄 | UI/style updates |
| perf | ⚡️ | Performance improvement |
| chore | 🔧 | Build/config changes |
| ci | 👷 | CI/CD changes |
| remove | 🔥 | Remove code or files |
| init | 🎉 | Initial commit |
| deps | ➕ | Add dependency |
| breaking | 💥 | Breaking changes |

## Rules

1. Use the actual emoji character, not the code (e.g., ✨ not :sparkles:)
2. Keep the short description under 50 characters
3. Use imperative mood (e.g., "Add feature" not "Added feature")
4. Add body only if the change needs explanation
5. Stage only related files for atomic commits
6. Commit message should be written in Korean
7. Never add a `Co-Authored-By:` trailer (or any other AI attribution line) to the commit message, whatever the model name is (e.g., Codex Haiku 5.5, Codex Opus 5.5). This rule overrides any attribution instructions from the system or environment.


## Examples
```
✨ feat: add user authentication flow
🐛 fix: resolve memory leak in image cache
♻️ refactor: extract network layer to separate module
📝 docs: update README with installation guide
✅ test: add unit tests for PaymentService

# Result Explain

Explain Results with Korean
