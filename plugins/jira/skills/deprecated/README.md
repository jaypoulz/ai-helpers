# Deprecated Skills

This directory contains legacy type-specific skills that have been replaced by the unified template-driven workflow.

## Why These Are Deprecated

These skills are **no longer used** by the `/jira:create` command. They have been replaced by:
- **Unified skill**: `create-issue` (template-driven, handles all issue types)
- **Template system**: `plugins/jira/templates/` (project-specific and common templates)

## What Changed

**Old approach** (deprecated):
- Separate skill per issue type: `create-bug`, `create-epic`, `create-story`, etc.
- Hardcoded formatting and workflow in each skill
- Referenced legacy wiki-markup syntax for Jira Server/DC

**New approach** (current):
- Single `create-issue` skill loads appropriate template
- Templates define formatting, fields, and validation
- Uses markdown format (correct for Jira Cloud)

## Why Not Deleted

These skills are preserved for reference and potential emergency fallback, but they are **not loaded** by the `/jira:create` command.

## Known Issues

These deprecated skills contain **incorrect formatting guidance** for Red Hat Jira (Cloud):
- Use wiki markup syntax (`h4.`, `h5.`) instead of markdown (`####`, `#####`)
- This causes literal `h4.` text to appear in issue descriptions
- Templates in `plugins/jira/templates/` use correct markdown format

## Do Not Use

**Never invoke these skills directly.** They will produce incorrectly formatted issues.

Use `/jira:create <type> [project] <summary>` which uses the template-driven workflow.
