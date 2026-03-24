# TODO: Synopsis Extraction Fix for Skillsaw

## Background

Commit `6f3a412` fixed synopsis extraction in the old `build-website.py` script to handle language specifiers in code blocks (e.g., ` ```bash`, ` ```text`).

The script was deleted in upstream commit `9dd2118` as part of migration to skillsaw documentation system.

## Problem Still Exists in Skillsaw

The synopsis extraction issue **still exists** in the current skillsaw-generated documentation.

**Evidence:** Check upstream `docs/index.html` for `ci:analyze-disruption`:
```json
"synopsis": "Analyze disruption events across one or more CI job runs, comparing backends to identify root causes:\n```text\n/ci:analyze-disruption <prowjob-url-1> [prowjob-url-2 ...]"
```

The synopsis is malformed - it captures descriptive text before the code block instead of just the command syntax.

## Original Fix

The original fix in `build-website.py` was:

```python
# OLD (broken):
match = re.search(r'## Synopsis\s*```\s*([^\n]+)', content, re.MULTILINE)

# NEW (fixed):
# Match synopsis after optional language specifier (bash, text, etc.)
match = re.search(r'## Synopsis\s*```(?:\w+)?\s*\n([^\n]+)', content, re.MULTILINE)
```

This handled:
1. Optional language specifier after opening backticks
2. Skip to next line and capture actual synopsis content
3. Backwards compatibility with code blocks without specifiers

## Action Items

- [ ] Check if skillsaw has synopsis extraction logic that needs similar fix
- [ ] Review skillsaw repository: https://github.com/stbenjam/skillsaw
- [ ] Test if the issue affects command rendering in the marketplace
- [ ] Consider contributing fix upstream to skillsaw if relevant
- [ ] Verify all commands with ` ```bash` or ` ```text` in Synopsis sections render correctly

## Related Commits

- Original fix: `6f3a412` - fix(scripts): handle language specifiers in synopsis extraction
- Skillsaw migration: `9dd2118` - Add OWNERS files, enable self-service plugin contributions, migrate to skillsaw

## Examples of Affected Commands

Commands using language specifiers in Synopsis sections:
- `ci-extras:check-release-health` (uses ` ```bash`)
- `ci:analyze-disruption` (uses ` ```text`)
- Many others - grep for ` ```bash` or ` ```text` after `## Synopsis`
