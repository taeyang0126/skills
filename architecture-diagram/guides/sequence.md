# Sequence Diagram Guide

Covers: UML sequence diagrams, interaction flows, API call chains.

Sources: UML 2.5 specification, Visual Paradigm sequence diagram guides.

## Layout

### Horizontal: Lifelines

- Each participant is a vertical **lifeline**: header box at top, dashed vertical line extending downward
- Lifelines arranged left-to-right in order of first interaction
- Equal horizontal spacing between lifelines (typically 150-180px apart)
- Leftmost lifeline = initiator (actor/client)

### Vertical: Time Axis

- Time flows top-to-bottom
- Each message occupies a horizontal row, spaced 40-50px apart vertically
- Number messages sequentially: `1.`, `2.`, `3.` (optional but recommended for complex flows)

## SVG Patterns

### Lifeline Header

```svg
<!-- Participant: Client, x=100, lifelineX=100 -->
<rect x="50" y="30" width="100" height="40" rx="6" fill="var(--mask-bg)"/>
<rect x="50" y="30" width="100" height="40" rx="6" fill="var(--frontend-fill)" stroke="var(--frontend-stroke)" stroke-width="1.5"/>
<text x="100" y="47" fill="var(--text-muted)" font-size="7" text-anchor="middle">[Actor]</text>
<text x="100" y="60" fill="var(--text-primary)" font-size="10" font-weight="600" text-anchor="middle">Client</text>
<!-- Lifeline dashed vertical -->
<line x1="100" y1="70" x2="100" y2="500" stroke="var(--text-muted)" stroke-width="1" stroke-dasharray="4,4"/>
```

### Activation Bar

Thin rectangle on lifeline showing active processing period:

```svg
<!-- Activation: Client active y=90..150 -->
<rect x="94" y="90" width="12" height="60" rx="2" fill="var(--frontend-fill)" stroke="var(--frontend-stroke)" stroke-width="1"/>
```

### Synchronous Message (solid arrow)

```svg
<!-- 1. Client(x=100,y=100) → API(x=280,y=100): POST /orders [HTTPS] -->
<line x1="106" y1="100" x2="274" y2="100" stroke="var(--backend-stroke)" stroke-width="1.5" marker-end="url(#ah-green)"/>
<text x="190" y="93" fill="var(--text-secondary)" font-size="8" text-anchor="middle">1. POST /orders [HTTPS]</text>
```

### Return Message (dashed arrow)

```svg
<!-- Return: API(x=280,y=140) → Client(x=100,y=140): 201 Created -->
<line x1="274" y1="140" x2="106" y2="140" stroke="var(--backend-stroke)" stroke-width="1" stroke-dasharray="5,3" marker-end="url(#ah-green)"/>
<text x="190" y="133" fill="var(--text-secondary)" font-size="8" text-anchor="middle">201 Created</text>
```

### Asynchronous Message (open arrowhead)

Use a different marker with open triangle:

```svg
<marker id="ah-open" markerWidth="8" markerHeight="5" refX="7" refY="2.5" orient="auto" markerUnits="strokeWidth">
  <polyline points="0 0.5, 7 2.5, 0 4.5" fill="none" stroke="var(--msgbus-stroke)" stroke-width="1.5"/>
</marker>
```

### Self-Call

```svg
<!-- Self-call: API validates input -->
<path d="M 286 180 L 320 180 L 320 200 L 286 200" fill="none" stroke="var(--backend-stroke)" stroke-width="1.5" marker-end="url(#ah-green)"/>
<text x="330" y="193" fill="var(--text-secondary)" font-size="7">validate()</text>
```

### Combined Fragment (alt/loop/opt)

```svg
<!-- alt fragment: y=160..240 -->
<rect x="80" y="160" width="400" height="80" rx="4" fill="none" stroke="var(--text-muted)" stroke-width="1"/>
<text x="88" y="173" fill="var(--text-primary)" font-size="8" font-weight="600">alt</text>
<text x="130" y="173" fill="var(--text-secondary)" font-size="7">[order valid]</text>
<!-- Divider line for else -->
<line x1="80" y1="200" x2="480" y2="200" stroke="var(--text-muted)" stroke-width="1" stroke-dasharray="4,4"/>
<text x="88" y="213" fill="var(--text-secondary)" font-size="7">[else]</text>
```

## Rules

1. **Every message labeled** with operation name + protocol for cross-process calls
2. **Solid arrow** = synchronous call; **dashed arrow** = return/response; **open arrow** = async
3. **Activation bars** show processing duration — start at incoming message, end at return
4. **Left-to-right** = request direction; **right-to-left** = response
5. **Number messages** for complex diagrams (>5 messages)
6. **Max 6-8 lifelines** per diagram. Split if more
7. **Legend** must explain: solid vs dashed vs open arrows, activation bars, fragment types
