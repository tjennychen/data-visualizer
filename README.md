# Data Visualizer

A Claude Code skill that turns any data-rich markdown file into a self-contained HTML deck with embedded Apache ECharts visualizations. Built for consulting deliverables, brand listening reports, post-mortems, performance summaries.

## Why this exists

Research findings, post-mortems, and analytical reports often hide their most important data inside dense prose. Reading them feels like grep through a paragraph. The headline number sits in a sentence on page 4. The cross-tab that explains everything is buried as a markdown table you have to count with your finger.

This skill auto-detects chart-worthy data, builds 4 to 7 ECharts, and ships a self-contained HTML file with sober consulting-grade aesthetics. One file, opens in any browser, no server, no build step. The reader sees the headline number in five seconds.

## What it produces

- Single HTML file, ECharts 6.0 from CDN, opens in any browser
- 4 to 7 charts (auto-pruned to avoid chart-dump)
- Stats card row at the top with the headline numbers a reader should remember
- Each chart cites its source data (the markdown section or backup CSV)
- Sober consulting palette (warm red accent, muted grays)
- Chinese font support (PingFang SC fallback)
- File size typically under 100 KB

## What it auto-detects

| Data shape in your markdown | Chart type generated |
|---|---|
| Cross-tab / 2D matrix | Heatmap |
| Distribution / histogram | Vertical bar |
| Top-N rankings | Horizontal bar |
| Time series | Line chart with optional markArea bands |
| Segment comparison | Grouped bar |
| Brand or entity co-occurrence | Horizontal bar or network |
| Score distribution | Vertical bar (ordinal) |

## Install

```bash
git clone https://github.com/tjennychen/data-visualizer ~/.claude/skills/data-visualizer
```

## How to use

In a Claude Code session:

```
/data-visualizer path/to/findings.md
```

Or, in a sentence:

```
Visualize this findings file: ~/research/q4_review.md
```

The skill reads your markdown, identifies 4 to 7 chart-worthy data points, generates ECharts specs, and assembles a self-contained HTML deck.

## Design system

Sober consulting palette baked in. Override at top of `_template_base.html` if you need brand colors.

```
Primary text:     #1a1a1a
Secondary text:   #5a5a5a
Accent:           #c63d3d (warm red)
Series palette:   8 muted blues, grays, and warm earth tones
```

## Architecture

4-step internal workflow:

1. Read the markdown end-to-end
2. Identify chart-worthy data (4 to 7 charts max, headline finding always charted)
3. Generate CHART_SPECS.md and per-chart JS files
4. Assemble the HTML from the template

Each chart is a separate JS file under `charts/`, so re-runs are fast and individual charts can be tweaked without rebuilding the deck.

## Origin

Built during a private consulting engagement where the FINDINGS.md was dense prose and the client needed a deck. The pattern of inductively detecting chart-worthy data, building a small set of ECharts in parallel, and shipping a self-contained HTML file proved reusable enough to codify.

## License

MIT
