# State Machine Diagram Guide

Covers: UML state machine diagrams, state transition diagrams, lifecycle diagrams.

Sources: UML 2.5 state machine specification, Harel statecharts.

## Layout

- **States** arranged in a logical flow (typically left-to-right or top-to-bottom following the lifecycle)
- **Initial state** (filled circle) at top-left
- **Final state** (bullseye) at bottom-right
- Related states grouped together
- Min 50px gap between states
- Transitions should not cross each other where possible

## SVG Patterns

### Initial Pseudo-State

```svg
<!-- Initial state: cx=50, cy=100 -->
<circle cx="50" cy="100" r="8" fill="var(--text-primary)"/>
```

### Final Pseudo-State (Bullseye)

```svg
<!-- Final state: cx=800, cy=400 -->
<circle cx="800" cy="400" r="10" fill="none" stroke="var(--text-primary)" stroke-width="1.5"/>
<circle cx="800" cy="400" r="6" fill="var(--text-primary)"/>
```

### State Box

States have: name + optional internal actions (entry/do/exit).

```svg
<!-- State: "Processing" x=200, y=80, w=150, h=55 -->
<rect x="200" y="80" width="150" height="55" rx="8" fill="var(--mask-bg)"/>
<rect x="200" y="80" width="150" height="55" rx="8" fill="var(--backend-fill)" stroke="var(--backend-stroke)" stroke-width="1.5"/>
<text x="275" y="100" fill="var(--text-primary)" font-size="11" font-weight="600" text-anchor="middle">Processing</text>
<line x1="200" y1="107" x2="350" y2="107" stroke="var(--backend-stroke)" stroke-width="0.5" opacity="0.3"/>
<text x="210" y="120" fill="var(--text-secondary)" font-size="7">entry / startTimer()</text>
<text x="210" y="131" fill="var(--text-secondary)" font-size="7">do / processData()</text>
```

### Simple State (no internal actions)

```svg
<!-- State: "Idle" x=200, y=80, w=120, h=40 -->
<rect x="200" y="80" width="120" height="40" rx="8" fill="var(--mask-bg)"/>
<rect x="200" y="80" width="120" height="40" rx="8" fill="var(--backend-fill)" stroke="var(--backend-stroke)" stroke-width="1.5"/>
<text x="260" y="105" fill="var(--text-primary)" font-size="11" font-weight="600" text-anchor="middle">Idle</text>
```

### Transition Arrow

Transitions labeled with: `event [guard] / action`

```svg
<!-- Transition: Idle → Processing, trigger=submit [valid] / log() -->
<!-- Idle right(x=320,y=100) → Processing left(x=400,y=100) -->
<line x1="320" y1="100" x2="398" y2="100" stroke="var(--arrow)" stroke-width="1.5" marker-end="url(#ah)"/>
<text x="360" y="93" fill="var(--text-secondary)" font-size="7" text-anchor="middle">submit [valid] / log()</text>
```

### Self-Transition

```svg
<!-- Self-transition on "Processing": retry -->
<path d="M 310 80 L 310 60 L 350 60 L 350 80" fill="none" stroke="var(--arrow)" stroke-width="1.5" marker-end="url(#ah)"/>
<text x="330" y="55" fill="var(--text-secondary)" font-size="7" text-anchor="middle">retry [count<3]</text>
```

### Composite State (with sub-states)

```svg
<!-- Composite: "Active" containing sub-states -->
<rect x="180" y="60" width="300" height="180" rx="10" fill="rgba(52,211,153,0.04)" stroke="var(--backend-stroke)" stroke-width="1.5"/>
<text x="195" y="78" fill="var(--backend-stroke)" font-size="10" font-weight="600">Active</text>
<!-- Sub-states drawn inside -->
```

## Transition Label Format

```
event [guard] / action
```

- **event** — what triggers the transition (e.g., `submit`, `timeout`, `cancel`)
- **guard** — boolean condition in brackets (e.g., `[valid]`, `[retries < 3]`)
- **action** — side effect (e.g., `/ log()`, `/ notify()`)

All three parts are optional but at least event or guard should be present.

## Color Coding States

Use different fill colors to indicate state categories:

| Category | Color |
|----------|-------|
| Normal/active states | backend (green) |
| Error/failed states | security (rose) |
| Waiting/pending states | cloud (amber) |
| Initial/final | text-primary (no fill) |
| Composite boundary | backend stroke, low opacity fill |

## Rules

1. **Exactly one initial state** per diagram (or per region in composite states)
2. **Every transition labeled** — at minimum the triggering event
3. **Guards in square brackets** — `[condition]`
4. **Actions after slash** — `/ action()`
5. **Use `rx="8"`** for state boxes (more rounded than architecture boxes) to visually distinguish
6. **Entry/do/exit actions** separated by a thin line inside the state box
7. **Max 10-12 states** per diagram. Use composite states to manage complexity
8. **No orphan states** — every state must be reachable and must have at least one outgoing transition (except final state)
9. **Legend** must explain: initial/final symbols, state colors, transition label format `event [guard] / action`
