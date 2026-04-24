# ER Diagram Guide

Covers: entity-relationship diagrams, database schema visualization, data model diagrams.

Sources: Crow's Foot notation (industry standard), Peter Chen notation principles.

## Layout

- **Entities** arranged in a grid or clustered by domain
- Related entities placed adjacent to each other
- Primary entity (most relationships) in center
- Min 60px gap between entities
- Relationship lines connect entity edges, never pass through other entities

## Notation: Crow's Foot

Use Crow's Foot notation (most widely understood in industry):

| Symbol | Meaning | SVG |
|--------|---------|-----|
| `\|` (single line) | Exactly one (mandatory) | Short perpendicular line |
| `O` (circle) | Zero (optional) | Small circle |
| `<` (crow's foot) | Many | Three-line fork |
| `\|O<` | Zero or many | Circle + fork |
| `\|\|` | Exactly one (mandatory) | Two perpendicular lines |

## SVG Patterns

### Entity Box

Entities have: name header + attribute list. Primary keys underlined or marked PK.

```svg
<!-- Entity: User, x=100, y=80, w=160, h=120 -->
<rect x="100" y="80" width="160" height="120" rx="6" fill="var(--mask-bg)"/>
<rect x="100" y="80" width="160" height="120" rx="6" fill="var(--database-fill)" stroke="var(--database-stroke)" stroke-width="1.5"/>
<!-- Header bar -->
<rect x="100" y="80" width="160" height="28" rx="6" fill="var(--database-stroke)" opacity="0.15"/>
<text x="180" y="99" fill="var(--text-primary)" font-size="11" font-weight="600" text-anchor="middle">User</text>
<!-- Attributes -->
<text x="112" y="122" fill="var(--database-stroke)" font-size="8" font-weight="600">PK</text>
<text x="135" y="122" fill="var(--text-primary)" font-size="8" text-decoration="underline">id</text>
<text x="210" y="122" fill="var(--text-muted)" font-size="7">BIGINT</text>
<text x="135" y="137" fill="var(--text-primary)" font-size="8">username</text>
<text x="210" y="137" fill="var(--text-muted)" font-size="7">VARCHAR(64)</text>
<text x="112" y="152" fill="var(--text-secondary)" font-size="8">FK</text>
<text x="135" y="152" fill="var(--text-primary)" font-size="8">role_id</text>
<text x="210" y="152" fill="var(--text-muted)" font-size="7">BIGINT</text>
```

### Relationship Line with Crow's Foot

One-to-Many (User has many Orders):

```svg
<!-- User(right x=260,y=140) ──||──O<── Order(left x=400,y=140) : 1:N -->
<line x1="260" y1="140" x2="400" y2="140" stroke="var(--database-stroke)" stroke-width="1.5"/>
<!-- "One" side: two short perpendicular lines at User end -->
<line x1="270" y1="133" x2="270" y2="147" stroke="var(--database-stroke)" stroke-width="1.5"/>
<line x1="276" y1="133" x2="276" y2="147" stroke="var(--database-stroke)" stroke-width="1.5"/>
<!-- "Many" side: crow's foot at Order end -->
<line x1="390" y1="133" x2="398" y2="140" stroke="var(--database-stroke)" stroke-width="1.5"/>
<line x1="390" y1="147" x2="398" y2="140" stroke="var(--database-stroke)" stroke-width="1.5"/>
<line x1="390" y1="140" x2="398" y2="140" stroke="var(--database-stroke)" stroke-width="1.5"/>
<!-- Optional circle (zero-or-many) -->
<circle cx="384" cy="140" r="4" fill="var(--mask-bg)" stroke="var(--database-stroke)" stroke-width="1"/>
<!-- Relationship label -->
<text x="330" y="133" fill="var(--text-secondary)" font-size="7" text-anchor="middle">places</text>
```

### Cardinality Patterns

```
||──||   One-to-one (mandatory both sides)
||──O<   One-to-many (mandatory one, optional many)
O|──O<   Zero-or-one to zero-or-many
||──<    One-to-many (mandatory both sides)
```

## Rules

1. **Every relationship labeled** with verb phrase (e.g., "places", "belongs to", "contains")
2. **Crow's Foot notation** — industry standard, most readable
3. **PK/FK clearly marked** in attribute lists
4. **Data types shown** in muted text next to attributes
5. **Group related entities** visually — use subtle background regions if needed
6. **Max 10-12 entities** per diagram. Split large schemas by domain/module
7. **Avoid crossing lines** — rearrange entity positions to minimize crossings
8. **Legend** must explain: Crow's Foot symbols (one, many, zero, mandatory), PK/FK notation
