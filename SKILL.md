---
name: data-visualizer
description: Use when Jenny has a data-rich markdown file (research findings, brand listening report, data analysis, post-mortem, performance summary) and wants it rendered as a self-contained HTML deck with embedded charts. Produces a sober consulting-grade visual deliverable using Apache ECharts. Invoke with a path to the markdown file.
---

# Data Visualizer

Turns a data-rich markdown file into a self-contained HTML deck with embedded ECharts visualizations. Designed for consulting deliverables, brand listening reports, data summaries, post-mortems, performance summaries.

## Origin

Built during a private consulting engagement where the FINDINGS.md was dense prose and the client needed a deck. The pattern of inductively detecting chart-worthy data, generating a small set of ECharts in parallel, and shipping a self-contained HTML file proved reusable enough to codify across any future research deliverable.

## When to use

Trigger on any of these:
- Jenny has a `FINDINGS.md` or similar data-rich markdown file and asks to "visualize," "make readable," "build a deck," "make charts," "make a visual summary"
- Jenny shows a data-heavy markdown file and the prose is hard to scan
- Any consulting deliverable, post-mortem, or performance summary that has stats, distributions, cross-tabs, time series, or rankings buried in prose

Do NOT use for:
- Long-form articles or blog posts (those need editorial design, not data viz)
- Single-chart asks (just write one ECharts snippet inline, don't run the whole skill)
- Presentations (those need slide structure, not a scrolling HTML page)

## The 4-step workflow

### Step 1: Read the data file + identify chart-worthy data

Read the data file end to end. Extract every data point that's chart-worthy:

| Data shape | Chart type |
|---|---|
| Cross-tab / 2D matrix of percentages or counts | Heatmap |
| Distribution / histogram (one variable, count by bucket) | Vertical bar |
| Ranking / top-N list | Horizontal bar |
| Time series (counts or rates over time) | Line chart with optional markArea bands |
| Segment comparison (same metric across 3-5 groups) | Grouped bar |
| Single composite metric per segment | Horizontal bar (sorted) |
| Brand or entity neighborhood / co-occurrence | Horizontal bar OR network graph |
| Score / rating distribution | Vertical bar (ordinal) |

Pick 4-7 charts. Fewer than 4 means the prose is fine without visuals; more than 7 makes the deck a chart dump and loses narrative. The headline finding should always have a chart.

Also extract 3-5 headline stats for a "stats card row" at the top of the deck. These are the numbers a reader should remember if they read nothing else.

### Step 2: Write CHART_SPECS.md and _template.html

Create a working directory next to the data file (typical location: `data_dir/charts/`). Save:

- `CHART_SPECS.md` — design system + per-chart spec
- `_template.html` — full HTML scaffolding with ECharts CDN, layout, prose sections, and `__CHART_N_INJECTION__` placeholder per chart

Use the design system below verbatim unless Jenny has specified different brand colors. Use the template structure in `templates/_template_base.html` as a starting point and adapt the prose sections to the data.

#### Design system (default — sober consulting palette)

```
Primary text:        #1a1a1a (near-black)
Secondary text:      #5a5a5a (muted gray)
Background:          #ffffff (white)
Section divider:     #e8e8e8 (very light gray)
Card background:     #fafafa
Accent (key data):   #c63d3d (warm red, sober)
Series palette:      ['#c63d3d', '#1f3a5f', '#7a8a99', '#3c5a7a', '#9bb0c4', '#d97e5c', '#5c7a8a', '#a55a4a']
Heatmap scale:       #ffffff → #f4ebd9 → #c63d3d (low → high)
Pale highlight:      #fff8e8 (markArea bands)
```

```
Font family:    -apple-system, BlinkMacSystemFont, "Helvetica Neue", "PingFang SC", "Microsoft YaHei", sans-serif
Chart title:    16px / 600 / #1a1a1a
Chart subtitle: 13px / 400 / #5a5a5a
Axis labels:    12px / 400 / #5a5a5a
Data labels:    11px / 500 / #1a1a1a
```

```
Default chart height: 440px (single chart)
Tall chart (heatmap, multi-axis time series): 540px
Animation duration: 600ms
NO 3D, NO gradient fills, NO drop shadows
Numbers: comma-separated thousands ("1,295" not "1295")
```

#### Per-chart spec template

For each chart, the spec must include:

```
### Chart N: [chart name]

**Data source:** [path to data file + section name OR specific data points]
**Type:** [Heatmap / Vertical bar / Horizontal bar / Line / Grouped bar]
**Layout:**
  - X axis: [...]
  - Y axis: [...]
  - Bar/cell color logic: [...]
  - Labels: [...]
  - Legend: [...]
  - Annotations: [markLine / markArea / graphic if applicable]

**Title:** "[exact title text]"
**Subtitle:** "[exact subtitle text — should state the takeaway, not just describe the chart]"

**Output file:** `chart_N_topic.js`
**Variable name:** `chartNOption`
```

### Step 3: Spawn parallel chart agents

One agent per chart. Each agent:
- Reads CHART_SPECS.md (its assigned chart) + the source data file
- Writes a single self-contained `const chartNOption = { ... }` declaration
- Saves to `charts/chart_N_topic.js`
- Validates syntax via `node --check` before declaring done

Always run agents in PARALLEL via background mode. With 6 charts, wall time drops from ~60 min sequential to ~10 min parallel.

Agent prompt template (adapt per chart):

```
You are building Chart N of M for [project] FINDINGS.html visualization. Apache ECharts 6.0.

INPUT
- Read design system + chart spec: [path to CHART_SPECS.md]
- Read source data: [path to data file + section]

YOUR JOB
[Specific chart description from CHART_SPECS.md]

OUTPUT FILE
[path to chart_N_topic.js]

The file must contain ONLY:
const chartNOption = {
  // ... full ECharts option ...
};

CRITICAL REQUIREMENTS
- Use the exact data from the source file (no fabrication)
- Color palette from design system only, no improvisation
- Title and subtitle exactly as in CHART_SPECS.md (subtitle should state takeaway, not describe chart)
- Font family: system stack with PingFang SC fallback
- No 3D, no gradient, no shadow
- Tooltip on hover with appropriate detail
- Self-contained const declaration only
- ECharts 6.0 syntax
- Validate with `node --check` before saving
```

### Step 4: Assemble final HTML

After all chart agents complete, run a Python script (or do it inline) to:
1. Read `_template.html`
2. For each chart_N_topic.js, replace `__CHART_N_INJECTION__` placeholder with its content
3. Verify no placeholders remain
4. Strip em dashes from prose and chart titles (Jenny's voice rule)
5. Write final `FINDINGS.html` next to the source markdown file

Verify:
- File size 30-100 KB (typical for 4-7 charts)
- All chart variables present (chart1Option through chartNOption)
- No `__CHART_` placeholders remaining
- No em dashes (`grep "—" FINDINGS.html` returns nothing)

## Voice rules (Jenny-specific)

- NO em dashes anywhere (titles, subtitles, prose, code comments). Replace with periods, colons, or restructure.
- Hyphens OK in technical contexts (kebab-case-names, 4-axis, 5-segment, lcc-weighted) but NOT in prose where a period or comma works.
- Subtitles state the takeaway, not the chart description. Bad: "Distribution of integration scores." Good: "Median 1/12. Zero posts at 9+. Integration is essentially absent."
- Prose sections: short, declarative, fragment-friendly. No "It is important to note that" filler.
- Skeptic check at the bottom of every deck: where might the analysis be wrong?

## Output deliverable

Single self-contained HTML file at `[data_dir]/FINDINGS.html`. Opens in any browser. No server needed. Jenny opens with `open path/to/FINDINGS.html`.

## Success criteria

- Jenny can read the headline finding in the first 5 seconds (stats row + bottom line + first section title)
- Charts honor the design system across the deck (visual coherence)
- Every chart has a takeaway subtitle, not a description
- File loads and renders in <1 second
- No em dashes anywhere
- Chinese characters render correctly (PingFang SC fallback)

## Skeptic check on this skill

This skill produces a deck, not a strategy. The visual is only as good as the underlying data — if the source file is weak, the deck just makes the weakness easier to see. Use this AFTER the analysis is solid (e.g., after running phase synthesizer agents), not as a substitute for analysis. Also: ECharts has a learning curve for niche chart types (sankey, sunburst, parallel coordinates); if the right chart type is exotic, sanity-check the agent's output before assembly.

## Version notes

- v1.0 (2026-05-01) — initial skill, extracted from a real consulting deliverable workflow.
