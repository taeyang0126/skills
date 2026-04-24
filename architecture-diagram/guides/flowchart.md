# Flowchart & Swimlane Guide

Covers: process flowcharts, decision trees, swimlane (cross-functional) diagrams, activity diagrams.

Sources: ISO 5807 flowchart standard, BPMN conventions, swimlane best practices.

## Layout

### Standard Flowchart

- **Top-to-bottom** flow (preferred) or left-to-right
- Start/End at top/bottom (or left/right)
- Decision diamonds branch left/right (or up/down), with Yes/No labels on branches
- Consistent spacing: 50px vertical gap between steps, 80px horizontal for branches

### Swimlane Diagram

- **Horizontal lanes** (rows) for each participant/department, or **vertical lanes** (columns)
- Lane header on the left (or top) with participant name
- Steps flow top-to-bottom within and across lanes
- Lane boundaries: subtle dashed lines

```svg
<!-- Swimlane: "Frontend Team" y=50..200 -->
<rect x="20" y="50" width="860" height="150" rx="0" fill="none" stroke="var(--border)" stroke-width="1" stroke-dasharray="4,4"/>
<text x="30" y="130" fill="var(--text-primary)" font-size="10" font-weight="600" transform="rotate(-90,30,130)">Frontend</text>
```

## SVG Shapes

### Start/End (Rounded pill)

```svg
<!-- Start: x=200, y=30, w=100, h=36 -->
<rect x="200" y="30" width="100" height="36" rx="18" fill="var(--mask-bg)"/>
<rect x="200" y="30" width="100" height="36" rx="18" fill="var(--backend-fill)" stroke="var(--backend-stroke)" stroke-width="1.5"/>
<text x="250" y="52" fill="var(--text-primary)" font-size="10" font-weight="600" text-anchor="middle">Start</text>
```

### Process Step (Rectangle)

```svg
<!-- Step: "Validate Input" x=200, y=100, w=140, h=45 -->
<rect x="200" y="100" width="140" height="45" rx="6" fill="var(--mask-bg)"/>
<rect x="200" y="100" width="140" height="45" rx="6" fill="var(--backend-fill)" stroke="var(--backend-stroke)" stroke-width="1.5"/>
<text x="270" y="120" fill="var(--text-primary)" font-size="10" font-weight="600" text-anchor="middle">Validate Input</text>
<text x="270" y="135" fill="var(--text-secondary)" font-size="8" text-anchor="middle">Check required fields</text>
```

### Decision Diamond

Use a rotated square (polygon):

```svg
<!-- Decision: "Valid?" center=(270,210) size=50 -->
<polygon points="270,185 295,210 270,235 245,210" fill="var(--mask-bg)"/>
<polygon points="270,185 295,210 270,235 245,210" fill="var(--cloud-fill)" stroke="var(--cloud-stroke)" stroke-width="1.5"/>
<text x="270" y="213" fill="var(--text-primary)" font-size="9" font-weight="600" text-anchor="middle">Valid?</text>
```

### Branch Labels

```svg
<!-- Yes branch: right from diamond -->
<line x1="295" y1="210" x2="380" y2="210" stroke="var(--backend-stroke)" stroke-width="1.5" marker-end="url(#ah-green)"/>
<text x="337" y="203" fill="var(--backend-stroke)" font-size="8" text-anchor="middle">Yes</text>

<!-- No branch: down from diamond -->
<line x1="270" y1="235" x2="270" y2="280" stroke="var(--security-stroke)" stroke-width="1.5" marker-end="url(#ah-rose)"/>
<text x="280" y="260" fill="var(--security-stroke)" font-size="8">No</text>
```

### Subprocess (Double-bordered rectangle)

```svg
<rect x="200" y="100" width="140" height="45" rx="6" fill="var(--mask-bg)"/>
<rect x="200" y="100" width="140" height="45" rx="6" fill="var(--backend-fill)" stroke="var(--backend-stroke)" stroke-width="1.5"/>
<rect x="204" y="104" width="132" height="37" rx="4" fill="none" stroke="var(--backend-stroke)" stroke-width="0.5"/>
<text x="270" y="127" fill="var(--text-primary)" font-size="10" font-weight="600" text-anchor="middle">Process Order</text>
```

## Rules

1. **One entry, one exit** per flowchart (unless explicitly showing error paths)
2. **Every decision diamond** must have labeled branches (Yes/No, True/False, or condition text)
3. **Flow direction consistent** — don't mix top-down and bottom-up in same diagram
4. **Avoid crossing lines** — rearrange steps or use orthogonal folds to prevent crossings
5. **Max 15 steps** per flowchart. Split complex processes into sub-flowcharts
6. **Swimlane lanes** should be ordered by frequency of interaction (most active lane in center)
7. **Color-code by lane** in swimlane diagrams — each lane uses a different component-type color
8. **Legend** must explain: shape meanings (pill=start/end, rect=process, diamond=decision), lane colors
