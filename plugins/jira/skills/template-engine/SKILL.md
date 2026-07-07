---
name: template-engine
description: Internal skill for template-driven issue creation - invoked by create router skill
---

# Template Engine (Internal Skill)

This skill provides the template-driven workflow engine for creating Jira issues. It is **not invoked directly** by users - instead, it is called by the `create` router skill when template mode is selected.

## When to Use This Skill

This skill is **automatically invoked** by the `create` skill when:
- User explicitly specifies `--template` flag
- Template auto-selection finds a matching template and user accepts
- The create router determines template mode should be used

## How It Works

1. **Template Selection**: Automatically selects the appropriate template based on project + issue type
2. **Dynamic Prompting**: Generates interactive prompts from template placeholder metadata
3. **Validation**: Applies validation rules defined in the template
4. **Issue Creation**: Creates the issue via MCP with all collected information

## Template-Driven Approach

Instead of hardcoding workflows for each issue type, this skill:
- Reads the template YAML file
- Extracts placeholder definitions
- Generates prompts dynamically from metadata
- Validates input based on template rules
- Formats the description using the template

**Benefits:**
- Single source of truth (templates)
- Consistent behavior across all issue types
- New issue types work automatically
- Template changes immediately reflected

## Workflow Overview

```
0. MCP Server Verification (first-time or reconnection)
   - Check if atlassian MCP tools are available
   - If not available and first time: offer guided setup
   - If not available but flag set: offer reconnection troubleshooting
   - Set mcp_setup_complete flag before requesting restart

1. Parse command arguments (project, type, summary, flags)
2. Load Resources (HYBRID APPROACH - Templates + Reference Files)
   2a. Load template YAML (project-specific → common fallback)
   2b. Load type-specific reference file based on issue_type from template
   2c. Invoke jira-conventions skill for project/team rules
   2d. Load educational docs (if template.documentation field exists)
3. Show resource references to user
4. Scan and describe template to user
5. Apply template defaults + convention overrides
6. Prompt for priority (all issue types)
7. Interactive placeholder collection (template + reference context)
8. Validate summary format
9. Security validation
10. Return structured issue data (NOT create - let create router handle MCP)
```

**Key Change:** Template-engine now loads BOTH templates and reference files for a hybrid approach.

## Detailed Implementation Steps

### Step 1: Parse Command Arguments

Extract from the create router invocation:
- `type` - issue type (story, bug, epic, etc.)
- `project-key` - Jira project
- `summary` - issue title
- `--template` - template name (if specified)
- `--component`, `--version`, `--parent`, etc. - optional flags

### Step 2a: Load Template YAML

**Template Selection Logic:**

1. **If --template flag provided:**
   ```
   template_name = args.template
   template_path = find_template(template_name)  # search all template dirs
   ```

2. **Else, auto-select template:**
   ```
   # Priority order:
   # 1. Project-specific template
   template_path = f"plugins/jira/templates/{project_key}/{type}.yaml"
   if not exists(template_path):
       # 2. Common template
       template_path = f"plugins/jira/templates/common/{type}.yaml"
   ```

3. **Load template YAML:**
   ```python
   template = yaml.load(template_path)
   
   # Resolve inheritance
   if 'inherits' in template:
       base = yaml.load(template['inherits'])
       template = merge(base, template)
   ```

4. **Validate template:**
   - Check required fields: `name`, `issue_type`, `description_template`
   - Verify `placeholders` is defined
   - Validate against SCHEMA.md

**Handle template not found:**
```
Error: Template not found: {template_name}
Available templates:
  - common-bug
  - common-epic
  - common-story
  ...
```

### Step 2b: Load Type-Specific Reference File

**Extract issue type from template:**
```python
issue_type = template.get('issue_type')  # e.g., "Bug", "Epic", "Story"
reference_file = f"../../reference/create-{issue_type.lower()}.md"
```

**Load reference file:**
```python
if os.path.exists(reference_file):
    reference_content = read_markdown(reference_file)
    reference_sections = parse_markdown_sections(reference_content)
else:
    reference_sections = {}
```

**Parse reference into sections:**

Extract these key sections from the markdown:
- `## Summary Guidelines` → summary_guidelines
- `## Bug Description Template` (or equivalent) → description_template_example
- `## Interactive Workflow` → workflow_steps
- `## Version Fields` → version_handling

**Store for later use:**
```python
context = {
    'template': template,
    'reference': reference_sections,
    'issue_type': issue_type
}
```

### Step 2c: Invoke jira-conventions Skill

**When to invoke:**
- Always invoke if `project-key` is provided
- Parse summary for keywords (HyperShift, GCP HCP, etc.)

**How to invoke:**
```
Use the jira-conventions skill to load project and team conventions.

Project key: {project-key}
Issue type: {type}
Summary: {summary}
Component: {component if provided}

Return:
- Project conventions (custom fields, version format, labels)
- Team conventions (if keywords match)
- Component suggestions
- Required fields and their IDs
```

**Receive convention data:**
```python
conventions = {
    'project': 'CNTRLPLANE',
    'custom_fields': {
        'Epic Link': 'customfield_10014',
        'Target Version': 'customfield_10855'
    },
    'version_format': 'openshift-{major}.{minor}',
    'default_labels': ['cntrlplane'],
    'component_suggestions': ['HyperShift', 'Control Plane'],
    'team': 'hypershift',
    'team_labels': ['hypershift']
}
```

### Step 2d: Load Educational Docs

**Check template for documentation field:**
```yaml
documentation:
  guide: "../../docs/issue-types/epic.md"
  description: "Comprehensive guide on epics: best practices and anti-patterns"
```

**If documentation field exists:**
```python
if 'documentation' in template:
    doc_path = template['documentation']['guide']
    doc_description = template['documentation']['description']
    educational_doc = read_markdown(doc_path) if exists(doc_path) else None
```

### Step 3: Show Resource References to User

**Display loaded resources:**
```
Creating {issue_type} issue for {project-key}

Resources loaded:
📋 Template: {template_name} ({template_path})
   Inherits: {base_template if applicable}
   
📖 Reference: {reference_file}
   Provides: Summary guidelines, interactive workflow, examples
   
🏢 Project: {project conventions}
   Custom fields: Epic Link, Target Version
   Default labels: {labels}
   
👥 Team: {team if applicable}
   Additional labels: {team_labels}
   Component suggestions: {components}

📚 Guide: {doc_path if applicable}
   {doc_description}

Let's begin...
```

This transparency helps users understand:
- What template is being used
- What reference guidance is available
- What project/team rules apply
- Where to find educational content

### Step 4: Scan and Describe Template

(Existing content continues here...)

## Educational Content Integration

Templates can reference educational documentation:

```yaml
documentation:
  guide: "../../docs/issue-types/epic.md"
  description: "Comprehensive guide on epics: what they are, best practices, and anti-patterns"
```

**At workflow start**, display the guide reference:
```
Creating Epic issue for <PROJECT>

📖 For guidance, see: docs/issue-types/epic.md
   "Comprehensive guide on epics: what they are, best practices, and anti-patterns"

Using template: common-epic (common/epic.yaml)

Let's start...
```

**During prompting**, reference specific sections in help text:
```
What are the key outcomes that define this epic as complete?

[?] For help: Epic acceptance criteria are high-level outcomes...
    See docs/issue-types/epic.md#acceptance-criteria for examples.
```

**On validation errors**, point to relevant guidance:
```
⚠️  Epic seems too large (estimated >5 sprints).
    Consider splitting into multiple epics or creating as a Feature.
    See docs/issue-types/epic.md#sizing for guidance.
```

## Reference File Integration (Step 2b - NEW)

**Load type-specific reference file** to provide prose guidance alongside structured templates.

### Reference File Mapping

Map template `issue_type` field to reference file:

```python
issue_type = template.get('issue_type')  # e.g., "Bug", "Epic", "Story"
reference_file = f"../../reference/create-{issue_type.lower()}.md"
```

| Template issue_type | Reference File |
|---|---|
| Bug | `reference/create-bug.md` |
| Epic | `reference/create-epic.md` |
| Story | `reference/create-story.md` |
| Task | `reference/create-task.md` |
| Feature | `reference/create-feature.md` |
| Feature Request | `reference/create-feature-request.md` |

### What to Extract from Reference Files

Reference files provide:
1. **Summary guidelines** - Examples of good/bad summaries
2. **Description templates** - Prose examples (complementary to YAML templates)
3. **Interactive workflow guidance** - What questions to ask, in what order
4. **Best practices and anti-patterns**
5. **Version field handling** - Project-specific version conventions
6. **Formatting guidelines** - Markdown for Jira

### Using Reference Context During Prompting

When prompting for template placeholders, **enrich help text** with reference file content:

```
Epic Name (3-5 words):
[?] (from template) Epic Name is displayed in epic picker and boards.
    Often matches or abbreviates the summary.
    
    (from reference) Examples:
    - Good: "Mobile App Redesign"
    - Good: "API v2 Migration"
    - Bad: "As a user I want to redesign the mobile app" (too long, story format)
    
    See reference/create-epic.md for more examples.
```

**Combination strategy:**
- Template provides: structure, validation rules, required/optional status
- Reference provides: context, examples, best practices, anti-patterns

## Project and Team Conventions (Step 2c - NEW)

**Invoke the jira-conventions skill** to layer project and team rules.

### When to Invoke

Always invoke `jira-conventions` when:
- `project-key` is provided in arguments
- Template specifies a target project
- Summary contains project keywords (HyperShift, GCP HCP, etc.)

### How to Invoke

```
Use the jira-conventions skill to load project and team conventions.

Project key: {project-key}
Issue type: {type}
Summary: {summary}

The jira-conventions skill will determine:
- Which project conventions apply (CNTRLPLANE, OCPBUGS, GCP)
- Which team conventions apply (HyperShift, GCP HCP)
- Required custom fields and their IDs
- Version normalization rules
- Component requirements
- Default labels
```

### Merging Convention Rules with Template Defaults

**Merge strategy:**

1. **Start with template defaults** (from `defaults:` section in YAML)
2. **Apply project conventions** (from jira-conventions skill)
3. **Apply team conventions** (layered on top of project)
4. **Apply command-line flags** (highest priority, overrides all)

**Example:**

```yaml
# Template defaults
labels: ["ai-generated-jira", "template:common-bug"]
```

```
# jira-conventions adds
labels: ["ai-generated-jira", "template:common-bug", "hypershift"]
customfield_10855: "openshift-4.21"  # Target Version
```

```
# Final (after --label flag)
labels: ["ai-generated-jira", "template:common-bug", "hypershift", "customer-reported"]
```

## Template Description (Step 4)

Before collecting placeholder values, describe the template to the user. This demonstrates that template interpolation is working and sets expectations for what will be collected.

**Display:**
1. Template name and source file
2. Template inheritance chain (if inherited)
3. Complete list of placeholders that will be collected
4. Which placeholders are required vs optional
5. Brief description of each placeholder
6. **Source template file for each placeholder** (shows inheritance)

**Example output for simple template:**
```
Using template: common-epic (common/epic.yaml)
  Inherits from: _base.yaml

This template will collect the following information:

Required fields:
  • epic_name (common/epic.yaml) - Short identifier for this epic
  • objective (common/epic.yaml) - Epic objective describing the capability
  • acceptance_criteria (common/epic.yaml) - High-level outcomes (2-8 items)
  • in_scope (common/epic.yaml) - What is included in this epic (1+ items)
  • target_release (common/epic.yaml) - Target release (e.g., Q1 2025, OpenShift 4.18)

Optional fields:
  • out_of_scope (common/epic.yaml) - What is NOT included (default: none)
  • size (common/epic.yaml) - Epic size estimate (default: M)
  • additional_context (common/epic.yaml) - Design docs, dependencies (default: none)

Let's begin collecting this information...
```

**For inherited templates with overrides**, show which template each field comes from:
```
Using template: ocpedge-spike (ocpedge/spike.yaml)
  Inherits from: common-spike (common/spike.yaml)
  Base template: _base.yaml

This template will collect the following information:

Required fields:
  • overview (ocpedge/spike.yaml) - Brief overview of the spike
  • research_questions (ocpedge/spike.yaml) - Research questions to investigate (one per line)
  • involved_teams (ocpedge/spike.yaml) - Teams involved (one per line)

Optional fields:
  • additional_spikes_link (ocpedge/spike.yaml) - Link to additional spikes document
  • design_proposal_link (ocpedge/spike.yaml) - Link to design proposal document
  • additional_context (ocpedge/spike.yaml) - Background, related issues (default: none)

Let's begin collecting this information...
```

**If fields came from different templates in the chain**, show mixed sources:
```
Required fields:
  • summary (common/story.yaml) - Short title for the story
  • user_role (common/story.yaml) - Who benefits from this story
  • action (common/story.yaml) - What the user wants to do
  • value (common/story.yaml) - Why the user wants it
  • acceptance_criteria (common/story.yaml) - Conditions for done (2-8 items)

Optional fields:
  • sprint (ocpedge/story.yaml) - Target sprint number
  • additional_context (common/story.yaml) - Design docs, background (default: none)
```

This summary:
- Confirms template loading worked correctly
- Shows placeholder interpolation is functioning
- **Demonstrates template inheritance** (which fields come from where)
- Warns user what to expect
- Helps user prepare information before being prompted
- Makes template system transparent and debuggable

### Step 5: Apply Template Defaults + Convention Overrides

**Merge configuration from multiple sources** in priority order:

```python
# Initialize with template defaults
config = template.get('defaults', {}).copy()

# Apply project conventions
if conventions and 'project' in conventions:
    config = merge_deep(config, conventions['project'])

# Apply team conventions
if conventions and 'team' in conventions:
    config = merge_deep(config, conventions['team'])

# Apply command-line flags (highest priority)
if args.component:
    config['components'] = [{'name': args.component}]
if args.version:
    config['versions'] = normalize_version(args.version, conventions)
if args.priority:
    config['priority'] = {'name': args.priority}
if args.security_level:
    config['security'] = {'name': args.security_level}
if args.labels:
    config['labels'].extend(args.labels)
```

**Merge strategy for specific fields:**

**Labels (combine all sources):**
```python
labels = []
labels.extend(template.get('defaults', {}).get('labels', []))
labels.extend(conventions.get('project', {}).get('default_labels', []))
labels.extend(conventions.get('team', {}).get('team_labels', []))
labels.extend(args.labels or [])
labels = list(set(labels))  # deduplicate
```

**Custom fields (convention overrides template):**
```python
custom_fields = template.get('defaults', {}).get('custom_fields', {}).copy()
custom_fields.update(conventions.get('project', {}).get('custom_fields', {}))
```

**Version normalization (use convention format):**
```python
version_format = conventions.get('project', {}).get('version_format')
if args.version and version_format:
    normalized = format_version(args.version, version_format)
    # e.g., "4.21" → "openshift-4.21" with format "openshift-{major}.{minor}"
```

**Component suggestions (from conventions):**
```python
component_suggestions = conventions.get('project', {}).get('component_suggestions', [])
if component_suggestions and not args.component:
    # Offer component selection during interactive prompts
    pass
```

**Result:**
```python
merged_config = {
    'labels': ['ai-generated-jira', 'template:common-bug', 'cntrlplane', 'hypershift'],
    'security': {'name': 'Red Hat Employee'},
    'custom_fields': {
        'customfield_10014': None,  # Epic Link (will be set if --parent)
        'customfield_10855': 'openshift-4.21'  # Target Version
    },
    'components': [{'name': 'HyperShift'}],
    'priority': {'name': 'Medium'}
}
```

This merged config is used as the base for issue creation.

### Step 6: Prompt for Priority

(Existing priority prompting logic...)

## Interactive Collection Process (Step 7 - Enhanced with Reference Context)

For each placeholder in the template:

### 1. Determine Prompt Type

Based on placeholder metadata:
- `prompt_type: suggestion` → Generate suggestion, ask for confirmation
- `prompt_type: options` → Present options to choose from
- `prompt_type: guided` → Ask sub-questions, assemble answer
- `prompt_type: simple` (default) → Direct prompt

### 2. Get Prompt Text

Priority order:
1. `prompt_text` (if specified)
2. `description` (fallback)

### 3. Enrich with Reference Context (NEW - Hybrid Approach)

**Before displaying prompt**, check if reference file has relevant context:

```python
# Match placeholder name to reference sections
reference_context = find_reference_context(
    placeholder_name=placeholder['name'],
    reference_sections=reference_sections
)

# Example matches:
# placeholder "problem_description" → reference "## 1. Problem Description"
# placeholder "steps" → reference "## 4. Steps to Reproduce"
# placeholder "epic_name" → reference "Epic Name field"
```

**If reference context found, combine it with template help:**

```python
combined_help = []

# Template help text (structured)
if 'help_text' in placeholder:
    combined_help.append(f"(from template) {placeholder['help_text']}")

# Reference context (prose, examples)
if reference_context:
    combined_help.append(f"\n(from reference) {reference_context}")
    
# Examples from template
if 'examples' in placeholder:
    combined_help.append("\nExamples:")
    for example in placeholder['examples']:
        combined_help.append(f"  - {example}")

help_text = "\n".join(combined_help)
```

**Display enriched prompt:**
```
Problem Description:
[?] (from template) Clear, detailed description of the issue
    Include context, component affected, impact (who, severity)
    
    (from reference) Provide:
    - What you were trying to do
    - What component or feature is affected
    - Who is affected? How severe is it?
    - When did this start happening?
    
    Examples:
      - "The kube-apiserver pod crashes immediately after upgrading..."
      - "The login button on mobile app doesn't respond to taps..."
    
    See reference/create-bug.md for more examples.

>
```

**Benefits of hybrid approach:**
- Template provides: structure, validation, field definition
- Reference provides: context, prose guidance, real-world examples
- User gets best of both: structured + contextual help

### 4. Show Help if Available

Help is displayed automatically (enriched from template + reference).

If user types "?" or "help" explicitly:
- Display full help_text (template + reference combined)
- Show all examples
- Show link to reference file section
- Re-prompt for value

### 4. Collect Value

Based on `type`:
- `text`: Single-line input
- `multiline`: Multi-line text block
- `list`: Multiple items, one per line

### 5. Validate Input

Apply validation rules from `validation`:
- `required`: Must have value
- `min_length` / `max_length`: Length constraints
- `min_items` / `max_items`: List size constraints
- `pattern`: Regex validation
- `allowed_values`: Enum validation

### 6. Use Default if Applicable

If user provides no value and `default` is specified:
- Use the default value
- Skip if not required

## Prompt Type Examples

### Simple Prompt

```yaml
placeholders:
  - name: task_description
    description: "What work needs to be done?"
    required: true
    type: text
```

**Execution:**
```
What work needs to be done?
> [user types answer]
```

### Suggestion Prompt

```yaml
placeholders:
  - name: epic_name
    prompt_type: suggestion
    prompt_with_suggestion: true
    suggestion_template: "Generate from summary: extract key nouns/action, max 3-5 words"
```

**Execution:**
```
Epic Name:
  Summary: "Enable graceful API server responses during planned maintenance"
  Suggested: "API Server Graceful Responses"

Use this epic name? (yes/no/custom)
> [user responds]
```

### Options Prompt

```yaml
placeholders:
  - name: size
    prompt_type: options
    prompt_text: "What is the estimated size?"
    prompt_with_options:
      - value: "S"
        description: "Small - About 2 sprints"
      - value: "M"
        description: "Medium - About 3 sprints"
```

**Execution:**
```
What is the estimated size?
1. S - Small - About 2 sprints
2. M - Medium - About 3 sprints
3. L - Large - About 4 sprints

Select option (1-3):
> [user selects]
```

### Guided Prompt

```yaml
placeholders:
  - name: user_story
    prompt_type: guided
    guided_questions:
      - "Who is the user?"
      - "What do they want to do?"
      - "Why do they want it?"
    assembly_template: "As a {{q1}}, I want to {{q2}}, so that {{q3}}."
```

**Execution:**
```
Let's build the user story together.

Who is the user?
> cluster admin

What do they want to do?
> configure automatic node pool scaling

Why do they want it?
> to handle traffic spikes without manual intervention

User story:
  As a cluster admin, I want to configure automatic node pool scaling,
  so that I can handle traffic spikes without manual intervention.

Does this look correct? (yes/no/modify)
> [user confirms]
```

## Priority Prompting

**Always prompt for priority** (applies to all issue types):

```
What is the priority for this issue?

Common values:
  - Blocker, Urgent, Critical, Must Have, High
  - Major, Should Have
  - Normal (default)
  - Medium, Minor, Low, Could Have
  - Trivial, Optional

Priority: [Normal]
> [user enters or accepts default]
```

## OpenShift Version Handling

For OpenShift projects (CNTRLPLANE, OCPBUGS, etc.), version handling follows standard conventions.

### Version Normalization

Users may specify versions in various formats. Normalize all inputs to the Jira format `openshift-X.Y`:

| User Input | Normalized Output |
|------------|-------------------|
| `4.21` | `openshift-4.21` |
| `4.22.0` | `openshift-4.22` |
| `openshift 4.23` | `openshift-4.23` |
| `openshift-4.21` | `openshift-4.21` |
| `OCP 4.22` | `openshift-4.22` |
| `ocp 4.21` | `openshift-4.21` |
| `OpenShift 4.23` | `openshift-4.23` |

**Normalization rules:**
1. Convert to lowercase
2. Remove "ocp" or "openshift" prefix (with or without space/hyphen)
3. Extract version number (X.Y or X.Y.Z → X.Y)
4. Prepend "openshift-"

### Target Version Field (customfield_12319940)

**Status:** OPTIONAL by default (templates can override to make it required)

**When specified**, follow this workflow:

1. **Normalize** the user input to `openshift-X.Y` format
2. **Fetch available versions:**
   ```python
   versions = mcp__atlassian__jira_get_project_versions(project_key=project_key)
   ```
3. **Find the version ID** for the normalized version name
4. **If version doesn't exist**, suggest closest match or ask user to confirm
5. **Use correct MCP format** (array of version objects with ID):
   ```python
   "customfield_12319940": [{"id": "VERSION_ID"}]  # e.g., openshift-4.22
   ```

**IMPORTANT:**
- Do NOT use string format like `"openshift-4.22"` - this will fail
- Must use array with version ID: `[{"id": "12448830"}]`

### Affects Version/s Field (versions)

**Purpose:** Version where a bug was found (bugs only)

**Format:**
```python
"versions": [{"name": "4.21"}]  # version name as string in array
```

**Note:** Unlike Target Version, Affects Version uses version NAME, not ID.

### Never Set Fix Version/s

The `fixVersions` field is managed by the release team and **must never be set** by issue creation workflows.

## Validation

### Template-Based Validation

Apply rules from `validation` block:

```yaml
validation:
  summary_max_length: 80
  summary_should_start_with_verb: true
  required_fields:
    - field_name
```

### Placeholder-Level Validation

Apply rules from placeholder's `validation`:

```yaml
placeholders:
  - name: acceptance_criteria
    type: list
    validation:
      min_items: 2
      max_items: 8
```

### Universal Validation

Always validate:
- Required placeholders have values
- Summary is not empty
- No sensitive data (credentials, keys, tokens)

## Description Formatting

**CRITICAL: Always use markdown format when rendering templates.**

For complete formatting guidelines, see the [Jira Formatting section in the plugin README](../../README.md#jira-formatting).

**Required format rules:**
- Main sections: `#### Section Name` (h4 level)
- Subsections: `*Subsection Name*` (emphasis, NOT headers like `##` or `###`)
- Numbered lists: `1.`, `2.`, `3.` (NOT `#` which creates headers)
- Bullet lists: `-` (NOT `*` which can conflict with emphasis)
- Code blocks: ` ```language\ncode\n``` `
- Inline code: `` `code` ``

**Common mistakes to avoid:**
- ❌ Using `#` for numbered lists → Creates `h1.` headers instead of list items
- ❌ Using `##` or `###` for subsections → Creates oversized headers
- ✅ Use `1.`, `2.`, `3.` for numbered lists
- ✅ Use `*Subsection*` for subsection labels (emphasis)

### Template Rendering

1. Load description_template from template
2. Populate with collected placeholder values
3. Use Mustache rendering:
   - `{{name}}` → Insert value
   - `{{#list}}...{{/list}}` → Iterate list
   - `{{#optional}}...{{/optional}}` → Show if truthy
4. **Verify formatting:** Ensure numbered lists use `1.` syntax, not `#`

## MCP Tool Parameters

### Basic Issue Creation

```python
mcp__atlassian__jira_create_issue(
    project_key="<PROJECT>",
    summary="<summary>",
    issue_type="<Epic|Story|Task|Bug|Spike|Feature>",
    description="<rendered template>",
    priority="<priority>",
    components="<component>",  # if specified
    additional_fields={
        "labels": ["ai-generated-jira", "template:<template-name>"],
        # Custom fields from template
        "customfield_12311141": "<epic_name>",  # if Epic
        "customfield_12320852": "<size>",  # if Epic with size
        # Parent linking if --parent flag
        "customfield_12313140": "<parent-key>",  # Epic → Feature
        "customfield_12311140": "<epic-key>",  # Story/Task → Epic
    }
)
```

### Field Mapping

Use `field` attribute from placeholder:

```yaml
placeholders:
  - name: epic_name
    field: customfield_12311141
```

Maps to:
```python
additional_fields["customfield_12311141"] = values["epic_name"]
```

## Error Handling

### Template Not Found

```
Could not find template for <project> <issue_type>.

Searched:
  1. ~/.jira-templates/<project>-<type>.yaml
  2. plugins/jira/templates/<project>/<type>.yaml
  3. plugins/jira/templates/common/<type>.yaml

Would you like to:
  1. Use a different template (specify path)
  2. Create issue without template
  3. Cancel
```

### Template Parse Error

Template file exists but contains invalid YAML syntax:

```
Failed to load template: ocpedge-spike.yaml
  YAML Parse Error: Line 45, Column 3: unexpected indent

The template file has a syntax error and cannot be parsed.

Would you like to:
  1. Edit the template to fix errors
  2. Use a different template
  3. Cancel
```

**Common causes:**
- Incorrect indentation (YAML requires consistent spaces, not tabs)
- Missing colons after keys
- Unmatched quotes or brackets
- Invalid characters in field names

**Debugging:**
- Use a YAML validator: `yamllint <template-file>`
- Check line number indicated in error
- Verify indentation is consistent (2 or 4 spaces, no tabs)

### Template Schema Violation

Template has valid YAML but violates the template schema:

```
Template validation failed: ocpedge-spike.yaml

Schema violations:
  - Missing required field: 'issue_type'
  - Invalid field type: 'defaults.priority' must be string, got list
  - Unknown placeholder referenced: 'unknown_field' used in description_template but not defined in placeholders

The template does not conform to the required schema.

Would you like to:
  1. Edit the template to fix violations
  2. Use a different template
  3. Cancel
```

**Common violations:**
- Missing required top-level fields: `name`, `description`, `version`, `issue_type`
- Wrong field types (e.g., string instead of list, object instead of string)
- Placeholder referenced in `description_template` but not defined in `placeholders`
- Invalid placeholder field types (must include `name` and `required`)
- Invalid validation rules (e.g., `min_items` on non-list field)

**See:** [Template Schema](../../templates/SCHEMA.md) for complete specification

### Circular Inheritance

Template inheritance creates a cycle:

```
Template inheritance cycle detected:

  team-epic.yaml → common-epic.yaml → team-epic.yaml

Templates cannot inherit from themselves, either directly or indirectly.

Please fix the inheritance chain in one of these templates:
  - team-epic.yaml
  - common-epic.yaml
```

**Common causes:**
- Template A inherits from B, B inherits from A (direct cycle)
- Template A inherits from B, B inherits from C, C inherits from A (indirect cycle)
- Typo in `inherits` path pointing to wrong template

**Fix:**
- Review inheritance chain
- Ensure linear inheritance (no cycles)
- Typical pattern: `team-template.yaml → common/template.yaml → _base.yaml`

### Missing Placeholder Reference

Template's `description_template` references undefined placeholder:

```
Template validation warning: ocpedge-epic.yaml

Undefined placeholder reference in description_template:
  - Template uses {{missing_field}} but no placeholder named 'missing_field' is defined

This will cause an error when rendering the description.

Would you like to:
  1. Edit the template to add the missing placeholder
  2. Edit the template to remove the reference
  3. Continue anyway (description will be incomplete)
```

**Common causes:**
- Typo in placeholder name in description_template
- Placeholder defined in parent but removed in child
- Copy-paste error from another template

**Fix:**
- Add missing placeholder to `placeholders` list
- Or remove `{{missing_field}}` from `description_template`
- Ensure placeholder names match exactly (case-sensitive)

### Invalid Placeholder Value

```
Invalid value for '<placeholder_name>':
  Error: Must be at least 2 items (got 1)

Please provide at least 2 <placeholder_description>:
> [retry]
```

### Required Field Missing

```
Required field '<placeholder_name>' has no value.

<description>

[Shows help_text if available]
[Shows examples if available]

> [prompt again]
```

### MCP Tool Error

```
Failed to create issue:
  Error: <mcp error message>

Suggested action: <based on error>
```

### Step 8: Validate Summary Format

**Check for common anti-patterns:**

1. **Summary looks like full user story:**
   ```python
   if summary.lower().startswith("as a") or "i want" in summary.lower() or "so that" in summary.lower():
       warn_user_story_in_summary()
   ```

2. **Summary exceeds recommended length:**
   ```python
   if len(summary) > 100:
       warn_summary_too_long()
   ```

**Prompt for correction if detected:**
```
⚠️  Summary looks like a full user story. Summaries should be concise titles.

Current: "As a cluster admin, I want to configure ImageTagMirrorSet..."

Suggested: "Enable ImageTagMirrorSet configuration in HostedCluster CRs"

Use suggested summary? (yes/no/edit)
```

### Step 9: Security Validation

**Scan all collected content for sensitive data:**

```python
sensitive_patterns = {
    'credentials': r'(password|passwd|pwd|secret|token|api[_-]?key)',
    'cloud_keys': r'(aws[_-]?(access|secret)|gcp[_-]?key|azure[_-]?key)',
    'kubeconfig': r'(kubeconfig|\.kube/config)',
    'ssh_keys': r'(ssh[_-]?rsa|-----BEGIN)',
    'certificates': r'(-----BEGIN CERTIFICATE|\.pem|\.crt)',
    'urls_with_creds': r'https?://[^:]+:[^@]+@'
}

for field_name, field_value in collected_data.items():
    for pattern_name, pattern in sensitive_patterns.items():
        if re.search(pattern, field_value, re.IGNORECASE):
            alert_sensitive_data(field_name, pattern_name)
            return STOP_CREATION
```

**If sensitive data detected:**
```
🚨 SECURITY ALERT: Potential {pattern_name} detected in {field_name}

DO NOT include sensitive data in Jira issues:
- Credentials, API tokens, passwords
- Cloud access keys (AWS, GCP, Azure)
- Kubeconfigs, SSH keys, certificates
- URLs with embedded credentials

Please remove the sensitive information and try again.
Use placeholders like: "<redacted>", "XXXXXX", or "<your-value-here>"
```

**Stop creation** - do not proceed to MCP if sensitive data found.

### Step 10: Return Structured Issue Data (NOT Create)

**Template-engine returns data, does NOT create the issue.**

The `create` router skill handles MCP creation (Phase 7).

**Return format:**
```python
return {
    'summary': final_summary,
    'description': rendered_description,
    'issue_type': issue_type,
    'project': {
        'key': project_key
    },
    'fields': {
        'components': components,
        'versions': versions,
        'customfield_10014': epic_link,  # if applicable
        'customfield_10855': target_version,  # if applicable
        'customfield_10018': parent_link,  # if applicable
        # ... other custom fields
    },
    'labels': labels,
    'priority': {'name': priority},
    'security': security,
    'template_used': template_name,
    'validation_passed': True
}
```

**Create router receives this data and:**
1. Runs final security validation (Phase 7)
2. Creates issue via MCP (Phase 8)
3. Returns result to user (Phase 9)

This separation allows:
- Template-engine focuses on template processing
- Create router handles MCP interaction
- Clear separation of concerns
- Easier testing and debugging

## Educational Content

For detailed guidance on each issue type, see:
- [Epic Best Practices](../../docs/issue-types/epic.md)
- [Story Best Practices](../../docs/issue-types/story.md)
- [Task Best Practices](../../docs/issue-types/task.md)
- [Bug Best Practices](../../docs/issue-types/bug.md)
- [Feature Best Practices](../../docs/issue-types/feature.md)
- [Spike Best Practices](../../docs/issue-types/spike.md)

## Implementation Notes

### Template Loading

Templates are loaded using a search path strategy:

1. **Search paths** (in priority order):
   - `~/.jira-templates/{project}-{type}.yaml` (user overrides)
   - `plugins/jira/templates/{project}/{type}.yaml` (project-specific)
   - `plugins/jira/templates/common/{type}.yaml` (common fallback)

2. **Override application**: If using a common template, check for `{project}/overrides.yaml` and apply if the `applies_to` field matches the template path

3. **Inheritance resolution**: If template has `inherits` field, recursively load parent template and merge

4. **Security protections**:
   - Path traversal protection (templates must be within approved directory)
   - File size limit: 1MB max
   - Inheritance depth limit: 10 levels max

### Template Merging

When merging child templates into parent templates:

- **Top-level fields**: Child values replace parent values
- **Placeholders**: Merged by name with field-level overrides
  - Same-named placeholders: child fields override parent fields
  - Unspecified fields in child inherit from parent
  - Child-only placeholders are added
- **Defaults**: Shallow merge (child defaults extend parent defaults)

See [Template Schema - Placeholder Field Merging](../../templates/SCHEMA.md#placeholder-field-merging) for details.

### Dynamic Prompting

Prompts are generated based on placeholder metadata:

- **`prompt_type: suggestion`**: Generate suggestion, ask for confirmation
- **`prompt_type: options`**: Present list of options to choose from
- **`prompt_type: guided`**: Ask sub-questions, assemble answer
- **`prompt_type: simple`**: Direct text prompt (default)

### Validation

Validation is applied at multiple levels:

1. **Placeholder validation**: Each placeholder value is validated against rules in its `validation` field:
   - `required`: Must have a value
   - `min_length`/`max_length`: Length constraints for text
   - `min_items`/`max_items`: Count constraints for lists
   - `pattern`: Regex pattern matching
   - `allowed_values`: Enum validation

2. **Template validation**: Template structure and references are validated by `validate_template.py`:
   - YAML syntax
   - Required fields presence
   - Placeholder references (all placeholders used in `description_template` must be defined)
   - Inheritance cycles
   - Redundant overrides

## See Also

- [Template Schema](../../templates/SCHEMA.md)
- [Create Router Skill](../create/SKILL.md) - Dual-mode router that invokes this skill
- [Template Management](../template-management/SKILL.md)
- [Templates README](../../templates/README.md) - Template system guide
