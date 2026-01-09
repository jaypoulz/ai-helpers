---
description: Build quarterly accomplishment summaries with dense formatting and competency mapping
argument-hint: "[start-date] [end-date] [accomplishments or file path]"
---

## Name
quarterly-connection:build

## Synopsis
/quarterly-connection:build [start-date] [end-date] [accomplishments or file path]

## Description
The `quarterly-connection:build` command gathers accomplishments from Jira, GitHub, and optional sources for a specified time period, then transforms them into dense, direct bullet points with explicit Red Hat competency keywords. Each bullet includes clickable links and demonstrates what was done, which competencies were applied, and what impact resulted.

Outputs Workday-ready HTML (primary) with clickable links, plus markdown and detailed analysis versions.

Use for performance reviews, promotion documents, quarterly summaries, and impact statements.

## Implementation

The command processes accomplishments using this pattern:
1. **GATHER** - Collect from Jira, GitHub, and optional sources
2. **ANALYZE** - Identify action, deliverable, competency, impact
3. **FORMAT** - Transform to dense bullets with full URLs
4. **MAP** - Apply Red Hat competency framework

**Process:**
- Query Jira for closed issues in date range
- Search GitHub for merged PRs in date range
- Parse additional input (text or file path)
- Combine and deduplicate (merge Jira+PR for same work)
- Output: Dense bulleted list with explicit Red Hat competency keywords
- Saved to `.work/quarterly-connection/` directory (gitignored)
- **Primary output:** `q[N]-[year]-accomplishments-workday.html` (Workday-ready HTML)
- **Secondary output:** `q[N]-[year]-accomplishments-workday.md` (Markdown)
- **Detailed analysis:** `q[N]-[year]-complete-accomplishments.md`

## Dense Bullet Format Structure

**Pattern:**
```
[Action verb] [deliverable] [readable links] via [Red Hat competency keywords] to [impact], [how/details].
```

**Writing Rules:**
- Use past tense action verbs (Fixed, Created, Updated, Implemented)
- No adjectives unless quantitative (e.g., "5 bugs", "30% faster", "3 teams")
- No adverbs (e.g., avoid "significantly", "greatly", "successfully", "carefully", "thoroughly")
- One sentence per bullet
- Focus on facts, not emphasis
- **REQUIRED: Include readable link text for all references**
- **REQUIRED: Use explicit Red Hat competency keywords (not generic descriptions)**

**Components:**

1. **Action** - What you did
   - Good: Fixed, Created, Updated, Implemented, Debugged, Refactored, Led, Designed
   - Avoid: "Successfully fixed", "Carefully created", "Effectively implemented"

2. **Deliverable** - Concrete output
   - Examples: PR, test, API, library, fix, document, proposal, tracker, demo

3. **Reference** - Readable link text (Workday/HTML format)
   - **GitHub PRs:** `[PR#NUMBER](URL)`
   - **Jira Issues:** `[ISSUE-KEY](URL)`
   - **Google Docs:** `[Descriptive Name](URL)`
   - Multiple references comma-separated in sentence
   - Examples:
     - `[PR#1500](https://github.com/openshift/cluster-etcd-operator/pull/1500)`
     - `[PR#30332](https://github.com/openshift/origin/pull/30332), [OCPEDGE-2207](https://issues.redhat.com/browse/OCPEDGE-2207)`
     - `[Strategy Doc](https://docs.google.com/document/d/DOCUMENT_ID/edit)`

4. **Competency** - **EXPLICIT Red Hat competency keywords**
   - **NOT:** "via collaboration" (too generic)
   - **YES:** "via cross-functional collaboration and functional area ownership"
   - Use competency subcategory names from framework:
     - Technical Contribution: business impact, technical innovation, technical knowledge, opportunity recognition, forward planning, strategic planning
     - Leadership: functional area leadership, cross-functional collaboration, team organization, continuous improvement, functional area ownership, team enablement
     - End-to-End Delivery: product delivery lifecycle management, customer focus, partner collaboration

5. **Impact** - Business value (what capability resulted)
   - Focus on: time saved, bugs prevented, capability enabled, team unblocked, alignment achieved
   - Quantify when possible (3 developers, 4.16+, 10 minutes, 30% faster, 15+ work streams)

6. **How/Details** - Brief technical or process details
   - Explains the approach taken
   - Provides context for the competency demonstrated
   - Examples: "analyzing timeline risks", "implementing exponential backoff", "testing LINSTOR/DRBD failover"

## Process Flow

1. **Date Range Parsing**
   - Parse start-date and end-date arguments
   - Validate format (YYYY-MM-DD)
   - Calculate quarter (Q1-Q4)

2. **Jira Gathering**
   - Query: `assignee=currentUser() AND status in (Done,Closed) AND resolved>=[start-date] AND resolved<=[end-date]`
   - Fetch: key, summary, description, priority, issuetype
   - Extract: parent epic, linked issues

3. **GitHub Gathering**
   - Search PRs: `author:[username] merged:[start-date]..[end-date]`
   - Search PRs: `author:[username] created:[start-date]..[end-date] is:merged`
   - Fetch: number, title, repository, url, body, labels, mergedAt

4. **Combining & Deduplication**
   - Match Jira issues to PRs by:
     - JIRA-ID in PR title or body
     - Similar work descriptions
   - Combine matched items into single accomplishment
   - Preserve all URLs

5. **Format Application** - Transform to dense format:
   - Use past tense action verb (no adverbs)
   - State deliverable (no adjectives unless quantitative)
   - **Include readable link text: [PR#123](URL), [JIRA-456](URL), [Doc Name](URL)**
   - Add competency with "via" or "by"
   - State impact with "to" or "enabling"
   - Remove: "successfully", "effectively", "comprehensive", "thorough", "carefully"

6. **Quality Checks**
   - Remove all unnecessary adjectives/adverbs
   - Verify one sentence per bullet
   - Confirm facts over emphasis
   - Quantify when possible (numbers, percentages, team size)
   - **Verify all references use readable link text**
   - Map to Red Hat competencies if provided

7. **Output**
   - Files saved to `.work/quarterly-connection/` directory (gitignored)
   - **Primary:** `q[N]-[year]-accomplishments-workday.html`
   - **Secondary:** `q[N]-[year]-accomplishments-workday.md`
   - **Detailed:** `q[N]-[year]-complete-accomplishments.md`
   - **Workday HTML Format (Primary):**
     - Flat bulleted list with `<ul>` and `<li>` tags
     - Clickable hyperlinks: `<a href="URL">readable text</a>`
     - Explicit Red Hat competency keywords
     - One sentence per bullet
     - Dense format (no unnecessary adjectives/adverbs)
     - Copy/paste directly into Workday
   - **Markdown Format (Secondary):**
     - Same content as HTML
     - Markdown links: `[text](URL)`
     - For systems that accept markdown
   - **Detailed Analysis:**
     - Competency mappings and statistics
     - Full analysis with level indicators

8. **Review**
   - Display file path
   - Show accomplishment count by source (Jira-only, PR-only, combined)
   - Note transformations applied
   - Highlight removed adjectives/adverbs
   - List competencies demonstrated

## Examples - Dense Format with Red Hat Competency Keywords (Workday/HTML Format)

**Before (verbose, generic competencies, unnecessary adverbs):**
```
Successfully developed a comprehensive PacemakerCluster CRD [api#2544] using extensive technical collaboration with OpenShift API maintainers to clearly define how we effectively monitor pacemaker stability.
```

**After (direct, explicit Red Hat competency keywords, quantified impact):**
```
Developed PacemakerCluster CRD [[PR#2544](https://github.com/openshift/api/pull/2544)] via cross-functional collaboration and forward planning to enable pacemaker health monitoring in TNF, working with API maintainers.
```

**Before (verbose, generic descriptions, unnecessary adverbs):**
```
Carefully fixed the job controller startup reliability issue [CEO#1500, OCPBUGS-63240] using thorough debugging in conjunction with the CEO leads to ensure a high-quality, reliable CEO.
```

**After (direct, explicit competencies, clear impact):**
```
Fixed job controller startup [[PR#1500](https://github.com/openshift/cluster-etcd-operator/pull/1500)] via technical innovation and business impact to prevent false degraded states, implementing exponential backoff to retry TNF setup for 10 minutes before degrading.
```

**Additional Examples with Explicit Red Hat Competency Keywords:**
```
Fixed TNF quorum detection [[PR#1483](https://github.com/openshift/cluster-etcd-operator/pull/1483), [OCPEDGE-2183](https://issues.redhat.com/browse/OCPEDGE-2183)] via technical knowledge and etcd cluster analysis to eliminate false quorum-lost alerts during fencing operations.

Created testing library [[PR#30332](https://github.com/openshift/origin/pull/30332), [OCPEDGE-2207](https://issues.redhat.com/browse/OCPEDGE-2207)] via technical innovation and team enablement to enable 3 test authors to develop TNF recovery tests in parallel without code duplication.

Fixed CEO presubmits [[PR#69895](https://github.com/openshift/release/pull/69895)] via continuous improvement and product delivery lifecycle management to reduce timeout failures, adding sharding to e2e-aws-ovn-serial for 4.16+.

Led strategic communication [[Strategy Doc](https://docs.google.com/document/d/DOCUMENT_ID/edit)] via functional area leadership and cross-functional collaboration to achieve cross-company alignment on delivery timeline, analyzing risks, alternatives, and mitigation plan.
```

**Key Improvements:**
- **Explicit competency keywords:** "functional area leadership", "cross-functional collaboration", "technical innovation", "team enablement" (not just "collaboration" or "innovation")
- **No unnecessary adverbs:** Removed "successfully", "carefully", "thoroughly", "effectively", "clearly"
- **Quantified details:** "3 test authors", "10 minutes", "4.16+", "15+ work streams"
- **Clear impact first, details second:** Impact stated with "to [outcome]", then technical details in second clause

## Red Hat Engineering Competencies

**Full Reference:** See `plugins/quarterly-connection/reference/red-hat-competencies.md`

**Example Bullets:** See `plugins/quarterly-connection/reference/example-bullets.md`

Map accomplishments to these competency subcategories from the Red Hat Software Engineering Competencies framework:

### Technical Contribution
- **Business Impact** - Impact on business, cost reduction, customer needs
- **Scope** - Task, component, technical area, multiple areas
- **Planning & Execution** - Estimation, feature planning, forward planning
- **Opportunity Recognition** - New capabilities, emerging tech, patents
- **Creativity & Innovation** - Novel solutions, technical improvements
- **Technical Knowledge** - Depth in key technology areas

### Leadership
- **Work Impact** - Ownership of tasks, features, functional areas, products
- **Continuous Improvement** - Process improvements, best practices
- **Portfolio Impact** - Cross-product collaboration and influence
- **Collaboration** - Cross-functional teamwork, upstream engagement

### Mentorship
- **Growth Impact** - Mentoring peers, teams, organization
- **Execution as a Mentee** - Self-improvement, networking, seeking guidance

### End-to-End Delivery
- **Product Delivery Life Cycle** - CI/CD, testing, documentation, quality
- **Customer Involvement & Focus** - Customer engagement, use case extraction

**Mapping Guide:**
- Technical fixes/features → Technical Contribution (Scope, Technical Knowledge)
- Process improvements → Leadership (Continuous Improvement)
- Cross-team work → Leadership (Collaboration, Portfolio Impact)
- Bug fixes → Technical Contribution (Business Impact), End-to-End Delivery (Product Delivery)
- New capabilities → Technical Contribution (Creativity & Innovation, Opportunity Recognition)
- Testing/CI work → End-to-End Delivery (Product Delivery Life Cycle)
- Customer issues → End-to-End Delivery (Customer Involvement & Focus)

## Return Value

Files are saved to `.work/quarterly-connection/` directory (gitignored).

- **Primary Output (HTML)**: `q[N]-[year]-accomplishments-workday.html`
  - **Workday-ready HTML format:**
    - Flat bulleted list (no categories)
    - Clickable hyperlinks with readable text: `[PR#123]`, `[JIRA-456]`, `[Doc Name]`
    - Dense format with explicit Red Hat competency keywords
    - Quantified impact
    - Copy/paste directly into Workday performance reviews
    - Opens in browser for easy copying

- **Secondary Output (Markdown)**: `q[N]-[year]-accomplishments-workday.md`
  - **Markdown version:**
    - Same content as HTML
    - Markdown link format: `[PR#123](URL)`
    - For systems that accept markdown

- **Detailed Analysis**: `q[N]-[year]-complete-accomplishments.md`
  - **Includes:**
    - Copy-ready bullets organized by category
    - Detailed competency mappings for each accomplishment
    - Red Hat competency framework analysis
    - Summary statistics and level indicators
    - All URLs organized by source
    - Performance review usage guide

## Examples

1. **Build Q4 2025 accomplishments**:
   ```
   /quarterly-connection:build 2025-10-01 2025-12-31
   ```

2. **Build Q3 2025 with additional input**:
   ```
   /quarterly-connection:build 2025-07-01 2025-09-30 "Created design doc for feature X. Led cross-team initiative Y."
   ```

3. **Build from file**:
   ```
   /quarterly-connection:build 2025-10-01 2025-12-31 ~/Documents/additional-accomplishments.txt
   ```

## Arguments

- **$1**: Start date in YYYY-MM-DD format (required)
- **$2**: End date in YYYY-MM-DD format (required)
- **$3+**: Additional accomplishments as text OR a file path (optional)
  - If valid file path, read from file
  - Otherwise, treat as accomplishment text
  - Multiple accomplishments separated by newlines

## Notes

**Dense Format Requirements:**
- **REQUIRED: Use readable link text (Workday/HTML format)**
- **REQUIRED: Use explicit Red Hat competency keywords from framework**
- Remove adjectives unless quantitative (5 bugs, 30% faster, 3 teams, 15+ work streams)
- Remove adverbs (successfully, carefully, thoroughly, effectively, significantly, clearly, extensively)
- Use facts, not emphasis
- One sentence per bullet
- State impact first, then technical/process details

**Link Format Requirements (Workday/HTML Style):**
- **GitHub PRs:** `[PR#NUMBER](https://github.com/org/repo/pull/NUMBER)`
- **Jira Issues:** `[ISSUE-KEY](https://issues.redhat.com/browse/ISSUE-KEY)`
- **Google Docs:** `[Descriptive Name](URL)` - e.g., `[Strategy Doc]`, `[Spike Proposal]`, `[Technical Brief]`
- **Multiple refs:** Comma-separated within sentence
- **Example:** `[[PR#123](URL), [OCPEDGE-456](URL)]`

**Competency Keyword Requirements:**
- **Use explicit competency subcategory names** from Red Hat framework
- **NOT generic:** "via collaboration" or "via innovation"
- **YES explicit:** "via cross-functional collaboration and functional area ownership" or "via technical innovation and business impact"
- **Common explicit keywords:**
  - Leadership: functional area leadership, cross-functional collaboration, team organization, continuous improvement, functional area ownership, team enablement, functional area coordination
  - Technical: business impact, technical innovation, technical knowledge, opportunity recognition, forward planning, strategic planning, architectural planning
  - Delivery: product delivery lifecycle management, customer focus, partner collaboration
- Infer from context which competencies apply
- Use 2 competency keywords per bullet (primary + supporting)
- Map each accomplishment to specific competency subcategories

**Impact Focus:**
- State business value directly
- Quantify when possible
- Focus on: capabilities enabled, bugs fixed, time saved, teams unblocked, versions affected

**Data Sources:**
- Primary: Jira (assigned issues closed in date range)
- Primary: GitHub (PRs authored and merged in date range)
- Optional: Additional text input or file
- Optional: Google Drive documents (if MCP configured)
