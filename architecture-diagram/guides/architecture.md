# Architecture Diagram Guide

Covers: system architecture, container diagrams, deployment diagrams, network topology.

Sources: C4 Model notation, Microsoft Azure Well-Architected Framework, IcePanel best practices.

## Layout Planning

### Step 1: Define Grid

Primary flow direction: **left→right**. All main-flow components on the same row.

Columns (left→right) for flow stages, rows for layers. Document as SVG comment:

```svg
<!-- Layout Grid:
     Col A (Clients):     x=20..195    w=175
     Col B (Gateway):     x=225..400   w=175
     Col C (Application): x=430..605   w=175
     Col D (Data Stores): x=635..810   w=175
     Wire channels: x=210, x=415, x=620
-->
```

### Step 2: Size and Place

- **Same-row components:** identical y and height — strict horizontal alignment
- **Same-column components:** identical x and width
- Min horizontal gap between components in same row: **30px**
- Min vertical gap between rows: **40px** (prefer 50px)
- Min horizontal gap between columns: **50px**
- Component min width: **175px** (CJK content), **150px** (English)
- Component min height: **70px** (with 4 text lines), **50px** (with 2 text lines)

### Step 3: Plan Wire Routes

Reserve x-coordinates between columns for vertical wire runs. Multiple wires spaced **≥15px** apart.

**Connection exit/entry rules:**
- Horizontal flow: exit from **right edge**, enter from **left edge**
- Vertical flow (e.g., main→sub layer): exit from **bottom edge**, enter from **top edge**
- Feedback loops: exit from **bottom edge** of source, route through a dedicated feedback channel below the main row, enter from **bottom edge** of target

## Component Box: Three-Line Convention

Every box MUST have:

1. **Type label** — `[Container]`, `[Database]`, `[Person]`, `[External]`, `[Cache]` — muted color, font-size 7 (8 for CJK)
2. **Name** — font-size 11 (12 for CJK), font-weight 600, primary color
3. **Description** — font-size 8 (9 for CJK), secondary color
4. **Technology** — font-size 7 (8 for CJK), stroke color of component type

```svg
<!-- API Server: x=380, y=130, w=200, h=70 [Container] -->
<rect x="380" y="130" width="200" height="70" rx="6" fill="var(--mask-bg)"/>
<rect x="380" y="130" width="200" height="70" rx="6" fill="var(--backend-fill)" stroke="var(--backend-stroke)" stroke-width="1.5"/>
<text x="480" y="145" fill="var(--text-muted)" font-size="7" text-anchor="middle">[Container]</text>
<text x="480" y="159" fill="var(--text-primary)" font-size="11" font-weight="600" text-anchor="middle">API Server</text>
<text x="480" y="172" fill="var(--text-secondary)" font-size="8" text-anchor="middle">Handles REST API requests</text>
<text x="480" y="185" fill="var(--backend-stroke)" font-size="7" text-anchor="middle">Spring Boot 2.1 / Java 8</text>
```

## Connection Labels

Every line MUST have a label. Inter-process connections MUST include protocol.

**Label style:**
- Background rect with `fill="var(--mask-bg)"` and 3px padding for readability
- Horizontal text only — no rotated labels
- Placed at midpoint of the longest segment, offset 8px above (horizontal) or right (vertical)

Good: `"Reads orders [JDBC/TLS]"`, `"Publishes events [Kafka]"`
Bad: `"Uses"`, `"Calls"`, `"Data"`

## Arrow Direction

Arrow = request/dependency direction (initiator → dependency). Declare in legend: `→ Request / dependency direction`

**No bidirectional arrows.** If truly bidirectional, draw two separate lines or annotate with "req/resp". (Source: Microsoft Azure WAF)

## Line Styles

| Style | Meaning | Usage |
|-------|---------|-------|
| Solid 1.5px | Runtime data-path dependency | Main flow, synchronous calls |
| Dashed `5,3` 1.2px | Config pull, async event, optional | Feedback loops, tool injection |
| Dashed `5,3` 1px | Weak/supporting dependency | Tool layer → main flow |

**Line width hierarchy:** main flow (1.5px) > feedback (1.2px) > supporting (1px). This creates visual depth.

## Boundaries

- **Cloud region:** `stroke-dasharray="8,4"`, cloud-stroke, `rx="12"`, very low opacity fill (`0.02-0.03`)
- **Security group:** `stroke-dasharray="4,4"`, security-stroke, transparent fill
- **Module group:** `stroke-dasharray="6,3"`, backend-stroke, very low opacity fill
- **Boundary opacity:** stroke opacity `0.4-0.6` — boundaries should be subtle, not compete with components

## SVG Rendering Order

1. Background grid → 2. Arrows → 3. Boundaries → 4. Mask rects → 5. Styled rects → 6. Text → 7. Connection label backgrounds + text → 8. Legend

## Summary Cards

Architecture diagrams include 3 info cards below the SVG summarizing key aspects (e.g., module layers, core business, infrastructure).
