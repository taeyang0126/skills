---
name: architecture-diagram
description: Create professional technical diagrams as standalone HTML files with SVG graphics. Supports architecture diagrams, sequence diagrams, flowcharts, ER diagrams, state machine diagrams, and more. Dark/light theme with auto-detection and manual toggle. Use when the user asks for any visual diagram showing system components, interactions, data models, processes, or state transitions.
license: MIT
metadata:
  version: "6.0"
  author: Cocoon AI (hello@cocoon-ai.com)
---

# Technical Diagram Skill

Self-contained HTML + inline SVG diagrams. Dark/light theme with system auto-detection and manual toggle.

## Diagram Type Router

Identify diagram type from user request, then read the corresponding guide BEFORE drawing:

| User says | Diagram type | Guide file |
|-----------|-------------|------------|
| 架构图、部署图、网络拓扑、infrastructure | Architecture | `guides/architecture.md` |
| 序列图、时序图、交互流程、sequence | Sequence | `guides/sequence.md` |
| 流程图、泳道图、flowchart、swimlane | Flowchart | `guides/flowchart.md` |
| ER图、数据模型、entity relationship | ER Diagram | `guides/er-diagram.md` |
| 状态图、状态机、state machine | State Machine | `guides/state-machine.md` |

**MUST read the guide file before generating any SVG.** The guide contains layout rules, SVG patterns, and best practices specific to that diagram type.

## Universal Rules (Apply to ALL diagram types)

### Theme Support

1. CSS variables on `:root` (dark default) and `[data-theme="light"]`
2. System detection via `@media (prefers-color-scheme: light)`
3. Manual toggle ☀️/🌙 button, ~15 lines JS only
4. All SVG colors use `var(--xxx)`. No hardcoded hex

### CSS Variables

```
--bg-page / --bg-card / --bg-grid / --border
--text-primary / --text-secondary / --text-muted
--arrow / --mask-bg
--{type}-fill / --{type}-stroke
```

Types: frontend, backend, database, cloud, security, msgbus, external

### Color Palette

| Type | Dark Fill | Light Fill | Stroke |
|------|-----------|------------|--------|
| Frontend | `rgba(8,51,68,0.4)` | `rgba(8,145,178,0.12)` | `#22d3ee` |
| Backend | `rgba(6,78,59,0.4)` | `rgba(16,185,129,0.12)` | `#34d399` |
| Database | `rgba(76,29,149,0.4)` | `rgba(139,92,246,0.12)` | `#a78bfa` |
| Cloud | `rgba(120,53,15,0.3)` | `rgba(245,158,11,0.12)` | `#fbbf24` |
| Security | `rgba(136,19,55,0.4)` | `rgba(244,63,94,0.10)` | `#fb7185` |
| MsgBus | `rgba(251,146,60,0.3)` | `rgba(251,146,60,0.12)` | `#fb923c` |
| External | `rgba(30,41,59,0.5)` | `rgba(100,116,139,0.10)` | `#94a3b8` |

Page: `--bg-page` #020617/#f8fafc, `--mask-bg` #0f172a/#f1f5f9

### Typography

JetBrains Mono via Google Fonts.

| Element | English | CJK (中日韩) |
|---------|---------|-------------|
| Name | 11px | 12px |
| Description | 8px | 9px |
| Type label / tech | 7px | 8px |
| Connection label | 7px | 8px |

When content is CJK-heavy, use the CJK column sizes. Mixed content uses English sizes.

### Component Sizing

- **Min width:** 150px (English), 175px (CJK). Ensure longest text line fits with ≥15px horizontal padding per side
- **Min height:** 65px. Ensure all text lines fit with ≥8px vertical padding top/bottom
- **Text must not overflow the box.** Before finalizing coordinates, estimate text width: CJK characters ≈ font-size × 1.0 per char, Latin ≈ font-size × 0.6 per char. If estimated width > (box width − 30px), widen the box
- **Same-row components:** identical height. Same-column: identical width

### Orthogonal Lines

All connections MUST be orthogonal (H+V segments only). Use `<path>` with L commands. Straight `<line>` only when endpoints share exact same x or y.

### Connection Routing

1. **Exit/entry edges:** arrows exit from the edge closest to the target. Horizontal flow → exit right, enter left. Vertical flow → exit bottom, enter top. Never exit from the same edge as entry on the same component unless it's a self-loop
2. **Wire channels:** reserve dedicated x or y coordinates between columns/rows for routing. Multiple parallel wires in the same channel spaced **≥15px** apart (not 10px)
3. **No crossing:** rearrange component positions or use different wire channels to avoid line crossings. If unavoidable, the crossing line should have a small gap or bridge
4. **Feedback loops:** route below (or above) the main flow row, using a dedicated feedback channel y-coordinate. All feedback arrows share the same channel band but are spaced ≥15px apart vertically

### Connection Labels

Every line MUST have a label. Label placement rules:

1. **Position:** at the midpoint of the longest segment of the path
2. **Offset:** 8-10px above (horizontal lines) or to the right (vertical lines) of the line
3. **Background:** wrap label text in a `<rect>` with `fill="var(--mask-bg)"` and 3px padding to prevent overlap with grid/other elements:

```svg
<!-- Label with background -->
<rect x="145" y="58" width="60" height="12" rx="2" fill="var(--mask-bg)"/>
<text x="175" y="67" fill="var(--backend-stroke)" font-size="7" text-anchor="middle">需求确认</text>
```

4. **No rotation:** labels should be horizontal. Avoid `transform="rotate()"` on labels — rotated text is hard to read

### Coordinate Comments

Every component and connection MUST have a comment documenting coordinates:

```svg
<!-- API Server: x=380, y=130, w=200, h=65 -->
<!-- API right(x=580) → fold x=610 → DB left(x=640,y=90) -->
```

### Arrow Markers

Define one `<marker>` per color. Include default `--arrow` plus one per stroke color used.

**Arrow shape:** slim elongated triangle, NOT stubby equilateral. Use `markerWidth="8" markerHeight="5"` with `refX="7" refY="2.5"` and polygon `"0 0.5, 7 2.5, 0 4.5"`:

```svg
<marker id="ah" markerWidth="8" markerHeight="5" refX="7" refY="2.5" orient="auto" markerUnits="strokeWidth">
  <polygon points="0 0.5, 7 2.5, 0 4.5" fill="var(--arrow)"/>
</marker>
```

**Rules:**
- `markerUnits="strokeWidth"` ensures arrow scales proportionally with line thickness
- Short connections (<40px gap between components): increase gap to ≥40px or merge components. Never let the arrow overlap the source/target box
- Forward and reverse arrows between the same pair MUST be vertically offset ≥20px so arrows don't overlap. Route them through different y-coordinates, not the same line

### Legend

Every diagram MUST have a legend covering all visual distinctions: colors, line styles, arrow semantics, boundary styles. Place OUTSIDE all boundary boxes, at least 20px below the lowest element.

### Metadata

- Header: title + subtitle (diagram type and scope)
- Footer: `title · author · YYYY-MM · version`

### Accessibility

Color MUST NOT be the only distinguisher. Pair with type labels, line patterns, or shape differences.

### Complexity Control

Single diagram: 15-20 elements max. If more, split into multiple diagrams at different abstraction levels.

### Spacing Minimums

| Gap type | Minimum |
|----------|---------|
| Between same-row components (horizontal) | 30px |
| Between same-column components (vertical) | 40px |
| Between columns | 50px |
| Boundary padding around contained components | 15px |
| Legend below lowest boundary | 20px |
| Connection label to line | 8px |

### File Writing Strategy

Split large HTML into segments via `fsWrite` + `fsAppend`:

1. DOCTYPE through `</defs>` + background grid
2. All arrows/connections
3. Boundaries + components
4. Legend + closing SVG, cards, footer, JS

### Layout Principles

1. **Flow direction first:** decide the primary flow direction (left→right or top→bottom) and commit to it. All main-flow components MUST be on the same row (horizontal flow) or same column (vertical flow)
2. **Parallel alignment:** components at the same abstraction level MUST share the same y (horizontal flow) or x (vertical flow). No staggering
3. **Secondary flows below/beside:** quality gates, feedback loops, and supporting layers go on separate rows/columns from the main flow
4. **viewBox sizing:** calculate from content. Sum all component widths + gaps + boundary padding + legend width. Add 40px margin on each side. Do NOT use a fixed viewBox — size it to fit the content

### HTML Layout Structure

1. Header — title + pulsing dot, subtitle
2. Theme toggle — top-right
3. SVG diagram — in rounded border card
4. Summary cards — 3-column grid (optional, mainly for architecture diagrams)
5. Footer — metadata

## Template

`assets/template.html` demonstrates architecture diagram patterns. Each guide file contains SVG patterns specific to its diagram type.
