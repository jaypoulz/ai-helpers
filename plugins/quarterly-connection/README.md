# Quarterly Connection Plugin

Build quarterly accomplishment summaries with dense formatting and Red Hat competency mapping for performance reviews and promotion documents.

## Commands

### `/quarterly-connection:build`

Build a comprehensive quarterly accomplishment summary by gathering data from Jira, GitHub, and optional sources, then formatting into dense bullets with full URLs and Red Hat competency mappings.

**Usage:**
```bash
/quarterly-connection:build 2025-10-01 2025-12-31
```

**Features:**
- Automatically queries Jira for closed issues in date range
- Searches GitHub for merged PRs in date range
- Combines and deduplicates Jira+PR for same work
- Formats accomplishments in dense style (no fluff, facts only)
- Includes full URLs for all references
- Maps to Red Hat Engineering Competency framework
- Outputs copy-ready bullets for performance reviews

**Format Requirements:**
- Dense bullets: action + deliverable + readable links + **explicit Red Hat competency keywords** + impact + details
- No adjectives unless quantitative (5 bugs, 30% faster, 15+ work streams)
- No adverbs (successfully, carefully, thoroughly, effectively, clearly, etc.)
- Readable link text: `[PR#123](URL)`, `[JIRA-456](URL)`, `[Doc Name](URL)`
- One sentence per bullet
- **REQUIRED:** Use explicit Red Hat competency subcategory names (e.g., "cross-functional collaboration and functional area ownership" not just "collaboration")

**Output:**
- **Primary (HTML):** `q[N]-[year]-accomplishments-workday.html` (Workday-ready HTML, copy/paste directly)
- **Secondary (Markdown):** `q[N]-[year]-accomplishments-workday.md` (Markdown version)
- **Detailed Analysis:** `q[N]-[year]-complete-accomplishments.md` (with full competency analysis)
- Saved to `.work/quarterly-connection/` directory
- Includes: formatted bullets with explicit competencies, competency mappings, statistics, original data

**Examples:**

Build Q4 2025 accomplishments:
```bash
/quarterly-connection:build 2025-10-01 2025-12-31
```

Build Q3 2025 with additional input:
```bash
/quarterly-connection:build 2025-07-01 2025-09-30 "Created design doc for X. Led initiative Y."
```

Build from file:
```bash
/quarterly-connection:build 2025-10-01 2025-12-31 ~/additional-work.txt
```

## Red Hat Competency Framework

The plugin maps accomplishments to Red Hat's Software Engineering Competencies:

### Technical Contribution
- Business Impact
- Scope (task → component → technical area → multiple areas)
- Planning & Execution
- Opportunity Recognition
- Creativity & Innovation
- Technical Knowledge

### Leadership
- Work Impact
- Continuous Improvement
- Portfolio Impact
- Collaboration

### Mentorship
- Growth Impact
- Execution as a Mentee

### End-to-End Delivery
- Product Delivery Life Cycle
- Customer Involvement & Focus

## Output Format

**Workday-Ready Bullet Example (with explicit Red Hat competency keywords):**
```
Fixed job controller startup [[PR#1500](https://github.com/openshift/cluster-etcd-operator/pull/1500)] via technical innovation and business impact to prevent false degraded states, implementing exponential backoff to retry TNF setup for 10 minutes before degrading.
```

**Key Elements:**
- **Action:** Fixed
- **Deliverable:** job controller startup
- **Link:** [PR#1500](URL) - clickable in HTML
- **Explicit Competencies:** "technical innovation and business impact" (not just "innovation")
- **Impact:** prevent false degraded states
- **Details:** implementing exponential backoff, 10 minutes retry

**Detailed Document Includes Competency Mapping:**
- **Primary:** Technical Contribution - Business Impact
- **Secondary:** Technical Contribution - Creativity & Innovation
- **Level:** Senior Software Engineer

## Data Sources

**Automatic:**
- Jira: Assigned issues closed in date range
- GitHub: PRs authored and merged in date range

**Optional:**
- Additional text input
- File with accomplishments
- Google Drive documents (if MCP configured)

## Configuration

Requires:
- GitHub authentication (via `gh` CLI or `GH_TOKEN`)
- Jira authentication (via Atlassian MCP server)

**Jira MCP Setup:** See [plugins/jira/docs/MCP_SETUP.md](../jira/docs/MCP_SETUP.md)

**GitHub Token:** Place in `/home/claude/.claude/.gh-token` or configure `gh auth login`

## Tips

**For Performance Reviews:**
1. Run at end of quarter with date range
2. Review competency mappings - align with role expectations
3. Use bullets grouped by competency category
4. Highlight quantified impacts (3 developers, 4.16+, 10 minutes)

**For Promotion Packets:**
1. Show breadth across competency categories
2. Emphasize PSE-level work (multiple technical areas, cross-functional)
3. Include strategic/forward-looking accomplishments
4. Demonstrate scope progression (component → subsystem → product)

**For Self-Assessments:**
1. Use competency subcategory distribution
2. Map to your role's expectations
3. Identify gaps or areas for growth
4. Track quarter-over-quarter progression

## Reference

- **Red Hat Competencies:** `plugins/quarterly-connection/reference/red-hat-competencies.md`
- **Example Bullets:** `plugins/quarterly-connection/reference/example-bullets.md`
- **Command Documentation:** `plugins/quarterly-connection/commands/build.md`
