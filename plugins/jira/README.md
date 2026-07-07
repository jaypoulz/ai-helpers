# Jira Plugin

Jira integration for Claude Code. Analyze issues, create solutions, and generate status rollups.

**Note:** This plugin is configured for Red Hat's Jira instance (issues.redhat.com) with templates and workflows designed for Red Hat product teams. Templates use Red Hat-specific custom fields and project conventions.

## Features

- 🔍 **Issue Analysis and Solutions** - Analyze JIRA issues and create pull requests to solve them
- 📊 **Status Rollups** - Generate status rollup comments for any Jira issue given a date range
- 📝 **Weekly Status Updates** - Automate weekly status summary updates with intelligent activity analysis and color-coded health indicators
- 📋 **Backlog Grooming** - Analyze new bugs and cards for grooming meetings
- 🏷️ **Activity Type Classification** - AI-powered classification of JIRA tickets into Sankey activity types, with single-issue and batch modes
- 🧪 **Test Generation** - Generate comprehensive test steps for JIRA issues by analyzing related PRs
- ✨ **Issue Creation** - Create well-formed stories, epics, features, tasks, bugs, and feature requests with guided workflows
- 📝 **Release Note Generation** - Automatically generate bug fix release notes from Jira and linked GitHub PRs
- 🤖 **Automated Workflows** - From issue analysis to PR creation, fully automated
- 💬 **Smart Comment Analysis** - Extracts blockers, risks, and key insights from comments

## Prerequisites

- Claude Code installed
- Atlassian Rovo MCP plugin configured (see below)
- Optional: `gh` CLI tools installed and configured, for GitHub access

### Atlassian MCP

The Atlassian Rovo MCP server is bundled with this plugin via `.mcp.json` — no separate installation is needed. On first use, you will be prompted to authenticate via your browser.

#### Direct API Commands

Some commands (e.g., `/jira:backlog`) use direct REST API calls for bulk operations that exceed MCP tool result size limits. These require environment variables:

| Variable | Description |
|----------|-------------|
| `JIRA_URL` | Jira instance URL (e.g., `https://redhat.atlassian.net`) |
| `JIRA_USERNAME` | Jira username (email) |
| `JIRA_API_TOKEN` | Jira API token (from [Atlassian API tokens](https://id.atlassian.com/manage-profile/security/api-tokens)) |

### Notes and tips

- Do not commit real tokens. If you must keep a project-local file, prefer committing a `mcp.json.sample` with placeholders, and keep your real `mcp.json` untracked.
- Consider using the [rh-pre-commit](https://source.redhat.com/departments/it/it_information_security/leaktk/leaktk_components/rh_pre_commit) hook to scan for secrets accidentally left in commits.

## Installation

Ensure you have the ai-helpers marketplace enabled, via [the instructions here](/README.md).

```bash
# Install the plugin
/plugin install jira@ai-helpers
```

## Reference Files

| File | Purpose |
|------|---------|
| [reference/markdown-for-jira.md](reference/markdown-for-jira.md) | Markdown formatting guide for Jira descriptions |

## Common Conventions

### Jira Formatting

**CRITICAL: When using MCP tools, always use markdown format in your input.**

The MCP Atlassian server automatically converts markdown to wiki markup for Jira. Even though Red Hat uses Jira Cloud (https://redhat.atlassian.net), the MCP tool handles the conversion, and the resulting wiki markup renders correctly.

**Required format for ALL Jira descriptions (via MCP tools):**

**Main sections (h4 level):**
```markdown
#### Section Name
```

**Subsections within a section (use italic emphasis, NOT headers):**
```markdown
*Subsection Name*
```
This converts to italic text, not a header. DO NOT use `##`, `###`, or `#####` for subsections as they create oversized headers.

**Numbered lists:**
```markdown
1. First item
2. Second item
3. Third item
```
DO NOT use `#` at the start of lines - this creates headers, not list items.

**Bullet lists:**
```markdown
- First item
- Second item
- Third item
```

**Inline code:**
```markdown
`code snippet`
```

**Code blocks:**
````markdown
```go
func main() {
    fmt.Println("Hello")
}
```
````

**Bold text:**
```markdown
**bold text**
```

**Italic text (for emphasis):**
```markdown
*italic text*
```

**Common formatting mistakes:**
- ❌ Using `#` for numbered lists → Creates headers
- ❌ Using `##` or `###` for subsections → Creates oversized headers
- ❌ Using `*` for bullet lists → Can conflict with emphasis
- ✅ Use `1.`, `2.`, `3.` for numbered lists
- ✅ Use `*Subsection*` for subsection labels (emphasis, not headers)
- ✅ Use `-` for bullet lists

## Available Commands

### `/jira:solve` - Analyze and Solve JIRA Issues

Analyze a JIRA issue and create a pull request to solve it. The command fetches issue details, analyzes the codebase, creates an implementation plan, makes the necessary changes, and creates a PR with conventional commits.

**Usage:**
```bash
/jira:solve OCPBUGS-12345 enxebre
```

See [commands/solve.md](commands/solve.md) for full documentation.

---

### `/jira:status-rollup` - Generate Weekly Status Rollups

Generate status rollup comments for any Jira issue by recursively analyzing all child issues and their activity within a date range. The command extracts insights from changelogs and comments to create well-formatted status summaries.

**Usage:**
```bash
/jira:status-rollup FEATURE-123 --start-date 2025-10-08 --end-date 2025-10-14
```

See [commands/status-rollup.md](commands/status-rollup.md) for full documentation.

---

### `/jira:grooming` - Backlog Grooming Assistant

Analyze and organize new bugs and cards added over a specified time period to prepare for grooming meetings. The command provides automated data collection, intelligent analysis, and generates structured, actionable meeting agendas.

**Usage:**
```bash
# Single project
/jira:grooming OCPSTRAT last-week

# Multiple OpenShift projects
/jira:grooming "OCPSTRAT,OCPBUGS,HOSTEDCP" last-week

# Filter by component
/jira:grooming OCPSTRAT last-week --component "Control Plane"

# Filter by label
/jira:grooming OCPSTRAT last-week --label "technical-debt"

# Combine filters
/jira:grooming OCPSTRAT last-week --component "Control Plane" --label "security"
```
See [commands/grooming.md](commands/grooming.md) for full documentation.

---

### `/jira:categorize-activity-type` - AI-Powered Activity Type Categorization

Analyze JIRA tickets and automatically assign Activity Type categories based on ticket content, issue type, labels, and parent Epic context. Uses AI-powered analysis with confidence scoring to ensure accurate categorizations.

**Usage:**
```bash
# Basic usage (prompts for confirmation)
/jira:categorize-activity-type ROX-12345

# Auto-apply for high confidence categorizations
/jira:categorize-activity-type ROX-12345 --auto-apply
```

See [commands/categorize-activity-type.md](commands/categorize-activity-type.md) for full documentation.

---

### `/jira:batch-categorize-activity-types` - Batch Activity Type Classification

Batch-categorize Jira issues that are missing Activity Types. Fetches issues via JQL, classifies each into one of six Sankey capacity allocation categories, validates results with deterministic scripts, generates a report, and applies updates with user approval. Shares the same classification skill as `/jira:categorize-activity-type`.

**Usage:**
```bash
# Classify all Epics in OCM with no Activity Type
/jira:batch-categorize-activity-types OCM

# Classify Stories instead of Epics
/jira:batch-categorize-activity-types ARO --type Story

# Add JQL filter
/jira:batch-categorize-activity-types OCM --jql 'AND resolved >= "2025-01-01"'

# Dry run (classify and report, don't apply)
/jira:batch-categorize-activity-types ROX --dry-run
```

See [commands/batch-categorize-activity-types.md](commands/batch-categorize-activity-types.md) for full documentation.

---

### `/jira:generate-test-plan` - Generate Test Steps

Generate test steps for a JIRA issue by analyzing related pull requests. The command supports auto-discovery of PRs from the JIRA issue or manual specification of specific PRs to analyze.

**Usage:**
```bash
# Auto-discover all PRs from JIRA
/jira:generate-test-plan CNTRLPLANE-205

# Test only specific PRs
/jira:generate-test-plan CNTRLPLANE-205 https://github.com/openshift/hypershift/pull/6888
```

See [commands/generate-test-plan.md](commands/generate-test-plan.md) for full documentation.

---

### `/jira:create` - Create Jira Issues

Create well-formed Jira issues with **dual-mode workflow**: structured templates with validation (Template Mode) or flexible prose-guided creation (Reference Mode). The command automatically applies project and team conventions.

**Basic Usage:**
```bash
# Auto-selects template if available, prompts you to accept
/jira:create story MYPROJECT "Add user dashboard"

# Explicit template mode
/jira:create bug OCPBUGS "API crash" --template ocpbugs-bug

# With options
/jira:create story CNTRLPLANE "Add metrics" --component "HyperShift" --version "4.21"
```

**Dual-Mode Workflow:**

The command supports two complementary modes:

**🎯 Template Mode** (Structured):
- Uses YAML templates with field validation
- Provides structured prompts with examples
- Enforces required fields and formats
- Combines template structure + reference examples + project conventions
- **When:** Template exists and you accept offer, or use `--template` flag
- **Best for:** Repeated tasks, learning proper format, team standards

**📖 Reference Mode** (Flexible):
- Uses markdown reference files with prose guidance
- Provides examples and best practices
- Flexible workflow without strict validation
- Combines reference examples + project conventions
- **When:** No template available, or you decline template offer
- **Best for:** One-off issues, custom scenarios, quick creation

**Mode Selection:**
```bash
# Command detects available template and prompts:
/jira:create story CNTRLPLANE "Add dashboard"
> Template available: common-story
> Use template workflow? (Y/n)

# Accept (Y) → Template Mode (structured)
# Decline (n) → Reference Mode (flexible)

# Force template mode:
/jira:create story CNTRLPLANE "Add dashboard" --template common-story

# No template available → automatically uses Reference Mode
```

**Example Workflows:**

```bash
# Template mode with full options
/jira:create bug OCPBUGS "kube-apiserver crashes" \
  --template ocpbugs-bug \
  --component "kube-apiserver" \
  --version "4.21" \
  --priority "High"

# Auto-select template (prompted to accept/decline)
/jira:create story CNTRLPLANE "Add authentication"

# Reference mode (decline template)
/jira:create task MYPROJECT "Update docs"
> Template available: common-task. Use template? (Y/n)
> n
> Using reference-guided workflow...

# Create with parent linking
/jira:create epic CNTRLPLANE "Mobile redesign" --parent CNTRLPLANE-100

# Feature request (specific project)
/jira:create feature-request RFE "Support custom SSL certs for ROSA HCP"
```

**Key Features:**
- **Dual-mode workflow** - Template (structured) or Reference (flexible)
- **Auto-detection** - Finds and offers templates automatically
- **Hybrid guidance** - Templates + Reference examples + Conventions combined
- **Smart defaults** - Project and team conventions applied automatically
- **Field validation** - Enforces required fields, lengths, patterns (Template Mode)
- **Security scanning** - Detects credentials, tokens, secrets before submission
- **Interactive prompts** - Context-aware questions with examples
- **Parent linking** - Epic → Feature, Story/Task → Epic relationships

**Supported Issue Types:**
- `story` - User stories with acceptance criteria
- `epic` - Epics with parent feature linking
- `feature` - Strategic features with market problem analysis
- `task` - Technical tasks and operational work
- `bug` - Bug reports with structured templates
- `feature-request` - Customer-driven feature requests for RFE project with business justification

**Project-Specific Conventions:**

Different projects may have different conventions (security levels, labels, versions, components, etc.). The command automatically detects your project and applies the appropriate conventions via project-specific skills.

**Team-Specific Conventions:**

Teams may have additional conventions layered on top of project conventions (component selection, custom fields, workflows, etc.). The command automatically detects team context and applies team-specific skills.

---

### `/jira:create-release-note` - Generate Bug Fix Release Notes

Automatically generate bug fix release notes by analyzing Jira bug tickets and their linked GitHub pull requests. The command extracts Cause and Consequence from the bug description, analyzes PR content (description, commits, code changes, comments), synthesizes the information into a cohesive release note, and updates the Jira ticket.

**Usage:**
```bash
/jira:create-release-note OCPBUGS-38358
```

**What it does:**
1. Fetches the bug ticket from Jira
2. Extracts Cause and Consequence sections from bug description
3. Finds all linked GitHub PRs
4. Analyzes each PR (description, commits, diff, comments)
5. Synthesizes Fix, Result, and Workaround information
6. Validates content for security (no credentials)
7. Prompts for Release Note Type selection
8. Updates Jira ticket fields

**Release Note Format:**
```
Cause: <extracted from bug description>
Consequence: <extracted from bug description>
Fix: <analyzed from PRs>
Result: <analyzed from PRs>
Workaround: <analyzed from PRs if applicable>
```

**Prerequisites:**
- MCP Jira server configured
- GitHub CLI (`gh`) installed and authenticated
- Access to linked GitHub repositories
- Jira permissions to update Release Note fields

**Example Output:**
```
✓ Release Note Created for OCPBUGS-38358

Type: Bug Fix

Text:
---
Cause: hostedcontrolplane controller crashes when hcp.Spec.Platform.AWS.CloudProviderConfig.Subnet.ID is undefined
Consequence: control-plane-operator enters a crash loop
Fix: Added nil check for CloudProviderConfig.Subnet before accessing Subnet.ID field
Result: The control-plane-operator no longer crashes when CloudProviderConfig.Subnet is not specified
---

Updated: https://redhat.atlassian.net/browse/OCPBUGS-38358
```

See [commands/create-release-note.md](commands/create-release-note.md) for full documentation.

---

### `/jira:update-weekly-status` - Update Weekly Status Summaries

Automate the process of updating weekly status summaries for Jira issues with intelligent activity analysis and color-coded health indicators. The command analyzes recent activity across tickets, GitHub PRs, and GitLab MRs to draft status updates (Red/Yellow/Green), then allows you to review and modify them before updating Jira.

**Usage:**
```bash
# Interactive mode (prompts for project and component)
/jira:update-weekly-status

# Specify project
/jira:update-weekly-status OCPSTRAT

# Specify project and component
/jira:update-weekly-status OCPSTRAT --component "Control Plane"

# With label filter
/jira:update-weekly-status OCPSTRAT --label strategic-work

# With specific users (by email)
/jira:update-weekly-status OCPBUGS antoni@redhat.com jdoe@redhat.com

# With excluded users
/jira:update-weekly-status OCPSTRAT !manager@redhat.com

# Full example with all options
/jira:update-weekly-status OCPSTRAT --component "Control Plane" --label strategic-work user@redhat.com
```

**Key Features:**
- Interactive component selection from available project components
- User filtering by email or display name (with auto-resolution)
- Intelligent activity analysis (comments, child issues, linked PRs/MRs)
- Recent update warnings to prevent duplicate updates (24-hour check)
- Batch processing with selective skip options
- Formatted status summaries with color-coded health indicators (Red/Yellow/Green)
- Auto-detects Status Summary custom field or prompts for field ID

**What it does:**
1. Filters issues by project, component, label, and assignee
2. Checks recent activity (comments, PR updates, child issues)
3. Drafts color-coded status summaries with specific accomplishments
4. Warns about recently-updated issues to avoid duplicates
5. Allows review and modification before updating
6. Provides comprehensive summary report with statistics

**Prerequisites:**
- Jira MCP server configured
- GitHub CLI (`gh`) installed and authenticated (optional but recommended)
- Jira permissions to update Status Summary field

See [commands/update-weekly-status.md](commands/update-weekly-status.md) for full documentation.

## Hybrid System: Templates + References + Conventions

The Jira plugin uses a **hybrid approach** combining three complementary systems:

### 1. Templates (Structured YAML)
Provide field structure, validation rules, and defaults.

**Common templates** (work with any project):
- `common-story` - User stories with acceptance criteria
- `common-epic` - Epics with scope and timeline
- `common-bug` - Bug reports with reproduction steps
- `common-spike` - Research and investigation
- `common-task` - Technical work and operational tasks
- `common-feature` - Strategic features with market analysis

**Product-specific templates:**
- `ocpbugs-bug` - OCPBUGS bug format with product fields
- `rhel-bug` - RHEL bug template
- `osdocs-bug` - OpenShift documentation bugs
- `rfe` - Feature requests (RFE project)

**Team-specific templates:**
- `ocpedge-spike` - OCPEDGE team spike format

### 2. Reference Files (Markdown Guidance)
Provide prose guidance, examples, and best practices.

Located in `plugins/jira/reference/`:
- `create-story.md` - Story format, user story structure, acceptance criteria
- `create-bug.md` - Bug template, reproduction steps, version fields
- `create-epic.md` - Epic Name field, scope/timeline, parent linking
- `create-task.md` - Task vs story distinction, action-verb summaries
- `create-feature.md` - Market problem, strategic value, success criteria
- `create-feature-request.md` - RFE workflow, business requirements

### 3. Conventions (Project/Team Rules)
Apply project-specific custom fields, labels, and version formats.

**Supported projects:**
- **CNTRLPLANE** - Control plane features, epics, stories
- **OCPBUGS** - OpenShift bugs
- **GCP** - GCP Hosted Control Planes (HyperShift on GKE)

**Supported teams:**
- **HyperShift** - Component selection, team labels
- **GCP HCP** - GCP project conventions, sizing guides

### How They Work Together

```
Template Mode:
  Templates (structure + validation)
     + Reference files (examples + guidance)  
     + Conventions (project rules)
     = Enriched interactive prompts

Reference Mode:
  Reference files (examples + guidance)
     + Conventions (project rules)
     = Flexible prose-guided workflow
```

**Example:** Creating a bug for OCPBUGS with Template Mode:

1. **Template** (`ocpbugs-bug.yaml`) provides:
   - Field structure (problem_description, steps, version, etc.)
   - Validation (min/max length, required fields)
   - Defaults (labels, format)

2. **Reference** (`create-bug.md`) provides:
   - Summary guidelines with good/bad examples
   - Interactive workflow descriptions
   - Best practices and anti-patterns

3. **Conventions** (OCPBUGS project) provides:
   - Custom field IDs (Target Version, Epic Link)
   - Version normalization ("4.21" → "openshift-4.21")
   - Default labels and security level

4. **Result**: Enriched prompts with:
   ```
   Problem Description:
   [?] (template) Clear, detailed description
       (reference) Include: what, component, impact, when
       (reference) Examples:
         - "kube-apiserver crashes after upgrade..."
       See reference/create-bug.md for more.
   ```

### Template Management

```bash
# List all available templates
/jira:template list

# Create issue with specific template
/jira:create bug OCPBUGS "API crash" --template ocpbugs-bug

# Create your own template
/jira:template create my-custom-template

# Create from existing template
/jira:template create-from ocpedge-spike

# Show template details
/jira:template show common-bug

# Validate template
/jira:template validate my-custom-template
```

**See [Template Documentation](templates/README.md) for:**
- Template inheritance and overrides
- Creating custom templates
- Template schema reference
- Hybrid approach details

---

## Troubleshooting

### "Could not find issue {issue-id}"
- Verify the issue ID is correct
- Ensure you have access to the issue in Jira
- Check that your Jira MCP server is properly configured

For command-specific troubleshooting, see the individual command documentation.

## Contributing

Contributions welcome! Please submit pull requests to the [ai-helpers repository](https://github.com/openshift-eng/ai-helpers).

## License

Apache-2.0
