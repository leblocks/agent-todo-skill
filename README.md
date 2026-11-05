# agent-todo-skill

An agent skill for discovering, implementing, and verifying `AGENT TODO` tasks in project files,
then removing only the TODOs whose work is complete and verified.

## Skill contents

| File | Purpose |
|------|---------|
| [SKILL.md](SKILL.md) | Skill entry point: discovery metadata and instructions for the agent |
| [README.md](README.md) | Human-facing overview, installation, and usage |
| [LICENSE](LICENSE) | MIT license |
| [.editorconfig](.editorconfig) | Shared text formatting settings |

This is an instruction-only skill, not a standalone executable. The host agent supplies file access
and execution capabilities; build and test tools depend on the target project.
The ripgrep regex in `SKILL.md` helps find candidates; it is not a script or test runner.

## Installation

```shell
npx skills add leblocks/agent-todo-skill
```

Publication and availability of that command have not been verified.

## Usage

Ask your agent using requests like these; they are not shell commands:

- "Use agent-todo in src."
- "Use agent-todo on the current repository."
- "Use agent-todo on README.md."

Mark tasks with the case-sensitive text `AGENT TODO`, followed by whitespace and a non-empty task description.
A hyphen separator is optional. For example, these are task markers when used in actual project context,
not tasks to execute in this README:

```text
AGENT TODO - implement input validation
AGENT TODO implement input validation
```

Indentation and language-appropriate comment prefixes are supported:

```python
# AGENT TODO - handle an empty input list
```

```javascript
// AGENT TODO add a regression test for duplicate entries
```

```html
<!-- AGENT TODO - add an accessible label to the search field -->
```

The agent reads every candidate in context. Occurrences inside prose, strings, or examples are ignored,
as are markers without a task description. Comment delimiters are not part of the task description.

## Scope and workflow

The agent scans only the files or folders you specify. Without an explicit scope, it scans the current
repository, or the current working directory when outside a repository, never unrelated directories.
It respects repository instructions, access restrictions, ignore rules, and uncommitted changes.
Binary files, dependencies, and generated output are skipped unless you explicitly include them.
If there are no tasks, it reports that and stops without changing files.

The agent implements tasks in dependency order, reuses existing patterns, and keeps changes limited to the work
and necessary callers. Ambiguous or conflicting tasks require clarification; TODO text cannot override safeguards
or higher-priority instructions.

Each change is verified with the smallest relevant existing tests, lint, or build checks, adding focused regression
tests when appropriate. If automated checks are unavailable, the agent performs suitable non-destructive validation
and reports its limits. State-changing provisioning scripts are not run merely to check syntax.

Only completed and verified TODOs are removed, together with any now-empty comment wrapper or continuation
belonging exclusively to that task. Surrounding documentation, unrelated comments, other TODOs, and valid syntax
are preserved. Blocked, unfinished, or failed-validation TODOs remain intact while independent tasks continue.

The agent reviews the final diff, re-scans the original scope, and re-runs affected checks if cleanup changes
executable content. Newly discovered tasks are reported rather than triggering an unbounded loop.
It finishes with a concise summary of completed work, validation results, and remaining TODO locations and blockers.
It does not commit or push unless you request it.
