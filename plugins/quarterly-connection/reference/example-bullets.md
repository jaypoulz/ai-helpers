# Example Accomplishment Bullets

These examples demonstrate the correct format for quarterly accomplishment bullets.

## Format Pattern

```
[Action verb] [deliverable] [readable links] via [Red Hat competency keywords] to [impact], [how/details].
```

## Good Examples

### Strategic Planning
```
Led strategic communication [Strategy Doc] via functional area leadership and cross-functional collaboration to achieve cross-company alignment on delivery timeline, analyzing risks, alternatives, and mitigation plan.
```

**Why this works:**
- ✅ Direct action verb: "Led"
- ✅ Explicit competencies: "functional area leadership and cross-functional collaboration"
- ✅ Clear impact: "achieve cross-company alignment"
- ✅ Quantified details: No unnecessary adjectives, just facts
- ✅ No adverbs: No "successfully", "effectively", etc.

### Technical Work
```
Fixed job controller startup [PR#1500] via technical innovation and business impact to prevent false degraded states, implementing exponential backoff to retry TNF setup for 10 minutes before degrading.
```

**Why this works:**
- ✅ Direct action: "Fixed"
- ✅ Explicit competencies: "technical innovation and business impact"
- ✅ Clear impact: "prevent false degraded states"
- ✅ Quantified: "10 minutes"
- ✅ Technical details at end: "implementing exponential backoff..."

### Team Enablement
```
Created testing library [PR#30332, OCPEDGE-2207] via technical innovation and team enablement to enable 3 test authors to develop TNF recovery tests in parallel without code duplication.
```

**Why this works:**
- ✅ Explicit competencies: "technical innovation and team enablement"
- ✅ Quantified impact: "3 test authors", "in parallel"
- ✅ Clear value: "without code duplication"

### Cross-Functional Collaboration
```
Advanced PacemakerCluster API design [OCPEDGE-2215] via cross-functional collaboration and forward planning to define pacemaker health monitoring in TNF operator conditions, working with API maintainers.
```

**Why this works:**
- ✅ Explicit competency: "cross-functional collaboration and forward planning"
- ✅ Specific partnership: "working with API maintainers"
- ✅ Clear deliverable: "define pacemaker health monitoring"

### Product Delivery
```
Fixed CEO presubmits [PR#69895] via continuous improvement and product delivery lifecycle management to reduce timeout failures, adding sharding to e2e-aws-ovn-serial for 4.16+.
```

**Why this works:**
- ✅ Explicit competencies: "continuous improvement and product delivery lifecycle management"
- ✅ Clear impact: "reduce timeout failures"
- ✅ Quantified scope: "4.16+"
- ✅ Technical approach: "adding sharding"

---

## Bad Examples (and how to fix them)

### ❌ Generic Competencies
```
Fixed job controller via debugging to improve reliability.
```

**Problems:**
- "via debugging" - too generic, not a Red Hat competency keyword
- "improve reliability" - vague impact
- No link references
- No details/quantification

**✅ Fixed version:**
```
Fixed job controller startup [PR#1500] via technical innovation and business impact to prevent false degraded states, implementing exponential backoff to retry TNF setup for 10 minutes before degrading.
```

### ❌ Unnecessary Adverbs
```
Successfully created a comprehensive testing library [PR#30332] using extensive collaboration to effectively enable developers to efficiently write tests.
```

**Problems:**
- "Successfully" - unnecessary adverb
- "comprehensive" - unnecessary adjective
- "extensive collaboration" - not explicit Red Hat competency
- "effectively", "efficiently" - unnecessary adverbs

**✅ Fixed version:**
```
Created testing library [PR#30332, OCPEDGE-2207] via technical innovation and team enablement to enable 3 test authors to develop TNF recovery tests in parallel without code duplication.
```

### ❌ Missing Impact
```
Led strategic communication [Strategy Doc] via functional area leadership, analyzing timeline risks and alternatives.
```

**Problems:**
- No clear impact statement ("to achieve...")
- Details given but outcome missing

**✅ Fixed version:**
```
Led strategic communication [Strategy Doc] via functional area leadership and cross-functional collaboration to achieve cross-company alignment on delivery timeline, analyzing risks, alternatives, and mitigation plan.
```

---

## Competency Keyword Reference

### Most Common Explicit Keywords to Use:

**Leadership:**
- functional area leadership
- cross-functional collaboration
- team organization
- continuous improvement
- functional area ownership
- team enablement
- functional area coordination

**Technical Contribution:**
- business impact
- technical innovation
- technical knowledge
- opportunity recognition
- forward planning
- strategic planning
- architectural planning

**End-to-End Delivery:**
- product delivery lifecycle management
- customer focus
- partner collaboration

### DON'T Use Generic Terms:
- ❌ "collaboration" → ✅ "cross-functional collaboration"
- ❌ "innovation" → ✅ "technical innovation"
- ❌ "leadership" → ✅ "functional area leadership"
- ❌ "planning" → ✅ "forward planning" or "strategic planning"
- ❌ "debugging" → ✅ "technical knowledge"
- ❌ "working with team" → ✅ "cross-functional collaboration and team enablement"

---

## HTML Format Example

When outputting to HTML, use this format:

```html
<ul>
<li>Fixed job controller startup [<a href="https://github.com/openshift/cluster-etcd-operator/pull/1500">PR#1500</a>] via technical innovation and business impact to prevent false degraded states, implementing exponential backoff to retry TNF setup for 10 minutes before degrading.</li>
</ul>
```

This renders as clickable links when pasted into Workday.
