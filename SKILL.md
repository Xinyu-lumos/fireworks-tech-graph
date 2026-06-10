---
name: fireworks-tech-graph
description: 生成技术图（架构图、流程图、序列图、Agent/记忆架构、思维导图等），输出 SVG+PNG。触发词："画图" "帮我画" "生成图" "做个图" "架构图" "流程图" "可视化一下" "出图" "generate diagram" "draw diagram" "visualize" 或任何需要可视化的系统/流程描述。
---

# Fireworks Tech Graph

Generate production-quality SVG technical diagrams exported as PNG via `rsvg-convert`.

## Helper Scripts (Recommended)

Three bash scripts in `scripts/` directory provide stable SVG generation and validation:

### 1. `generate-diagram.sh` - Generate SVG + PNG
```bash
./scripts/generate-diagram.sh <output-name> [width] [height] [output-dir]
```
- Generates SVG with predefined template
- Validates syntax with xmllint
- Exports PNG at 2x resolution
- Example: `./scripts/generate-diagram.sh my-arch 960 800`

### 2. `validate-svg.sh` - Validate SVG syntax
```bash
./scripts/validate-svg.sh <svg-file>
```
- Checks XML syntax
- Verifies tag c- Validates marker references
- Checks attribute completeness
- Validates path data

### 3. `test-all-styles.sh` - Batch test all styles
```bash
./scripts/test-all-styles.sh [output-dir]
```
- Tests multiple diagram sizes
- Validates all generated SVGs
- Generates test report

**When to use scripts:**
- Use scripts when generating complex SVGs to avoid syntax errors
- Scripts provide automatic validation and error reporting
- Recommended for production diagrams

**When to generate SVG directly:**
- Simple diagrams with few elements
- Quick prototypes
- When you need full control over SVG structure

## Workflow (Always Follow This Order)

1. **Classify** the diagram type (see Diagram Types below)
2. **Extract structure** — identify layers, nodes, edges, flows, and semantic groups from user description
3. **Plan layout** — apply the layout rules for the diagram type
4. **Load style reference** — always load `references/style-1-flat-icon.md` unless user specifies another; load the matching `references/style-N.md` for exact color tokens and SVG patterns
5. **Map nodes to shapes** — use Shape Vocabulary below
6. **Check icon needs** — load `references/icons.md` for known products
7. **Write SVG** with adaptive strategy (see SVG Generation Strategy below)
8. **Validate**: Run `rsvg-convert file.svg -o NUL 2>&1` to check syntax (Windows: NUL, Git Bash: /dev/null)
9. **Export PNG**: `rsvg-convert -w 1920 file.svg -o file.png`
10. **Report** output paths

## Diagram Types & Layout Rules

### Architecture Diagram
Nodes = services/components. Group into **horizontal layers** (top→bottom or left→right).
- Typical layers: Client → Gateway/LB → Services → Data/Storage
- Use `<rect>` dashed containers to group related services in the same layer
- Arrow direction follows data/request flow
- ViewBox: `0 0 960 600` standard, `0 0 960 800` for tall stacks

### Data Flow Diagram
Emphasizes **what data moves where**. Focus on data transformation.
- Label every arrow with the data type (e.g., "embeddings", "query", "context")
- Use wider arrows (`stroke-width: 2.5`) for primary data paths
- Dashed arrows for control/trigger flows
- Color arrows by data category (not just Agent/RAG — use semantics)

### Flowchart / Process Flow
Sequential decision/process steps.
- Top-to-bottom preferred; left-to-right for wide flows
- Diamond shapes for decisions, rounded rects for processes, parallelograms for I/O
- Keep node labels short (≤3 words); put detail in sub-labels
- Align nodes on a grid: x positions snap to 120px intervals, y to 80px

### Agent Architecture Diagram
Shows how an AI agent reasons, uses tools, and manages memory.
Key conceptual layers to always consider:
- **Input layer**: User, query, trigger
- **Agent core**: LLM, reasoning loop, planner
- **Memory layer**: Short-term (context window), Long-term (vector/graph DB), Episodic
- **Tool layer**: Tool calls, APIs, search, code execution
- **Output layer**: Response, action, side-effects
Use cyclic arrows (loop arcs) to show iterative reasoning. Separate memory types visually.

### Memory Architecture Diagram (Mem0, MemGPT-style)
Specialized agent diagram focused on memory operations.
- Show memory **write path** and **read path** separately (different arrow colors)
- Memory tiers: Working Memory → Short-term → Long-term → External Store
- Label memory operations: `store()`, `retrieve()`, `forget()`, `consolidate()`
- Use stacked rects or layered cylinders for storage tiers

### Sequence Diagram
Time-ordered message exchanges between participants.
- Participants as vertical **lifelines** (top labels + vertical dashed lines)
- Messages as horizontal arrows between lifelines, top-to-bottom time order
- Activation boxes (thin filled rects on lifeline) show active processing
- Group with `<rect>` loop/alt frames with label in top-left corner
- ViewBox height = 80 + (num_messages × 50)

### Comparison / Feature Matrix
Side-by-side comparison of approaches, systems, or components.
- Column headers = systems, row headers = attributes
- Row height: 40px; column width: min 120px; header row height: 50px
- Checked cell: tinted background (e.g. `#dcfce7`) + `✓` checkmark; unsupported: `#f9fafb` fill
- Alternating row fills (`#f9fafb` / `#ffffff`) for readability
- Max readable columns: 5; beyond that, split into two diagrams

### Timeline / Gantt
Horizontal time axis showing durations, phases, and milestones.
- X-axis = time (weeks/months/quarters); Y-axis = items/tasks/phases
- Bars: rounded rects, colored by category, labeled inside or beside
- Milestone markers: diamond or filled circle at specific x position with label above
- ViewBox: `0 0 960 400` typical; wider for many time periods: `0 0 1200 400`

### Mind Map / Concept Map
Radial layout from central concept.
- Central node at `cx=480, cy=280`
- First-level branches: evenly distributed around center (360/N degrees)
- Second-level branches: branch off first-level at 30-45° offset
- Use curved `<path>` with cubic bezier for branches, not straight lines

### Class Diagram (UML)
Static structure showing classes, attributes, methods, and relationships.
- **Class box**: 3-compartment rect (name / attributes / methods), min width 160px
  - Top compartment: class name, bold, centered (abstract = *italic*)
  - Middle: attributes with visibility (`+` public, `-` private, `#` protected)
  - Bottom: method signatures, same visibility notation
- **Relationships**:
  - Inheritance (extends): solid line + hollow triangle arrowhead, child → parent
  - Implementation (interface): dashed line + hollow triangle, class → interface
  - Association: solid line + open arrowhead, label with multiplicity (1, 0..*, 1..*)
  - Aggregation: solid line + hollow diamond on container side
  - Composition: solid line + filled diamond on container side
  - Dependency: dashed line + open arrowhead
- **Interface**: `<<interface>>` stereotype above name, or circle/lollipop notation
- **Enum**: compartment rect with `<<enumeration>>` stereotype, values in bottom
- Layout: parent classes top, children below; interfaces to the left/right of implementors
- ViewBox: `0 0 960 600` standard; `0 0 960 800` for deep hierarchies

### Use Case Diagram (UML)
System functionality from user perspective.
- **Actor**: stick figure (circle head + body line) placed outside system boundary
  - Label below figure, 13-14px
  - Primary actors on left, secondary/supporting on right
- **Use case**: ellipse with label centered inside, min 140×60px
  - Keep names verb phrases: "Create Order", "Process Payment"
- **System boundary**: large rect with dashed border + system name in top-left
- **Relationships**:
  - Include: dashed arrow `<<include>>` from base to included use case
  - Extend: dashed arrow `<<extend>>` from extension to base use case
  - Generalization: solid line + hollow triangle (specialized → general)
- Layout: system boundary centered, actors outside, use cases inside
- ViewBox: `0 0 960 600` standard

### State Machine Diagram (UML)
Lifecycle states and transitions of an entity.
- **State**: rounded rect with state name, min 120×50px
  - Internal activities: small text `entry/ action`, `exit/ action`, `do/ activity`
  - **Initial state**: filled black circle (r=8), one outgoing arrow
  - **Final state**: filled circle (r=8) inside hollow circle (r=12)
  - **Choice**: small hollow diamond, guard labels on outgoing arrows `[condition]`
- **Transition**: arrow with optional label `event [guard] / action`
  - Guard conditions in square brackets
  - Actions after `/`
- **Composite/nested state**: larger rect containing sub-states, with name tab
- **Fork/join**: thick horizontal or vertical black bar (synchronization)
- Layout: initial state top-left, final state bottom-right, flow top-to-bottom
- ViewBox: `0 0 960 600` standard

### ER Diagram (Entity-Relationship)
Database schema and data relationships.
- **Entity**: rect with entity name in header (bold), attributes below
  - Primary key attribute: underlined
  - Foreign key: italic or marked with (FK)
  - Min width: 160px; attribute font-size: 12px
- **Relationship**: diamond shape on connecting line
  - Label inside diamond: "has", "belongs to", "enrolls in"
  - Cardinality labels near entity: `1`, `N`, `0..1`, `0..*`, `1..*`
- **Weak entity**: double-bordered rect with double diamond relationship
- **Associative entity**: diamond + rect hybrid (rect with diamond inside)
- Line style: solid for identifying relationships, dashed for non-identifying
- Layout: entities in 2-3 rows, relationships between related entities
- ViewBox: `0 0 960 600` standard; wider `0 0 1200 600` for many entities

### Network Topology
Physical or logical network infrastructure.
- **Devices**: icon-like rects or rounded rects
  - Router: circle with cross arrows
  - Switch: rect with arrow grid
  - Server: stacked rect (rack icon)
  - Firewall: brick-pattern rect or shield shape
  - Load Balancer: horizontal split rect with arrows
  - Cloud: cloud path (overlapping arcs)
- **Connections**: lines between device centers
  - Ethernet/wired: solid line, label bandwidth
  - Wireless: dashed line with WiFi symbol
  - VPN: dashed line with lock icon
- **Subnets/Zones**: dashed rect containers with zone label (DMZ, Internal, External)
- **Labels**: device hostname + IP below, 12-13px
- Layout: tiered top-to-bottom (Internet → Edge → Core → Access → Endpoints)
- ViewBox: `0 0 960 600` standard

## UML Coverage Map

Full mapping of UML 14 diagram types to supported diagram types:

| UML Diagram | Supported As | Notes |
|-------------|-------------|-------|
| Class | Class Diagram | Full UML notation |
| Component | Architecture Diagram | Use colored fills per component type |
| Deployment | Architecture Diagram | Add node/instance labels |
| Package | Architecture Diagram | Use dashed grouping containers |
| Composite Structure | Architecture Diagram | Nested rects within components |
| Object | Class Diagram | Instance boxes with underlined name |
| Use Case | Use Case Diagram | Full actor/ellipse/relationship |
| Activity | Flowchart / Process Flow | Add fork/join bars |
| State Machine | State Machine Diagram | Full UML notation |
| Sequence | Sequence Diagram | Add alt/opt/loop frames |
| Communication | — | Approximate with Sequence (swap axes) |
| Timing | Timeline | Adapt time axis |
| Interaction Overview | Flowchart | Combine activity + sequence fragments |
| ER Diagram | ER Diagram | Chen/Crow's foot notation |

## Shape Vocabulary

Map semantic concepts to consistent shapes across all diagram types:

| Concept | Shape | Notes |
|---------|-------|-------|
| User / Human | Circle + body path | Stick figure or avatar |
| LLM / Model | Rounded rect with brain/spark icon or gradient fill | Use accent color |
| Agent / Orchestrator | Hexagon or rounded rect with double border | Signals "active controller" |
| Memory (short-term) | Rounded rect, dashed border | Ephemeral = dashed |
| Memory (long-term) | Cylinder (database shape) | Persistent = solid cylinder |
| Vector Store | Cylinder with grid lines inside | Add 3 horizontal lines |
| Graph DB | Circle cluster (3 overlapping circles) | |
| Tool / Function | Gear-like rect or rect with wrench icon | |
| API / Gateway | Hexagon (single border) | |
| Queue / Stream | Horizontal tube (pipe shape) | |
| File / Document | Folded-corner rect | |
| Browser / UI | Rect with 3-dot titlebar | |
| Decision | Diamond | Flowcharts only |
| Process / Step | Rounded rect | Standard box |
| External Service | Rect with cloud icon or dashed border | |
| Data / Artifact | Parallelogram | I/O in flowcharts |

## Arrow Semantics

Always assign arrow meaning, not just color:

| Flow Type | Color | Stroke | Dash | Meaning |
|-----------|-------|--------|------|---------|
| Primary data flow | blue `#2563eb` | 2px solid | none | Main request/response path |
| Control / trigger | orange `#ea580c` | 1.5px solid | none | One system triggering another |
| Memory read | green `#059669` | 1.5px solid | none | Retrieval from store |
| Memory write | green `#059669` | 1.5px | `5,3` | Write/store operation |
| Async / event | gray `#6b7280` | 1.5px | `4,2` | Non-blocking, event-driven |
| Embedding / transform | purple `#7c3aed` | 1px solid | none | Data transformation |
| Feedback / loop | purple `#7c3aed` | 1.5px curved | none | Iterative reasoning loop |

Always include a **legend** when 2+ arrow types are used.

## Layout Principles

**Alignment**: Snap node centers to **8px base grid**. Horizontal: 120px intervals. Vertical: 120px intervals (consistent layer spacing).

**Grouping**: Use `<rect>` with dashed stroke + semi-transparent fill + small label in top-left to visually group related nodes. Don't group more than 5-6 nodes in one box.

**Layering**: For complex diagrams, divide into named swim-lanes with subtle background fills.

**Spacing**: 
- Same-layer nodes: 80px horizontal spacing (consistent)
- Between layers: 120px vertical spacing (consistent)
- Canvas margins: 40px minimum on all sides
- Minimum 60px between node edges

**Visual Hierarchy**:
- **Core/primary nodes**: 20-30% larger than standard, double border or 3-4px stroke
- **Secondary nodes**: Standard size, 2-2.5px stroke
- **Tertiary nodes**: Standard size, 1.5-2px stroke
- **Color hierarchy**: Core = high saturation, Secondary = medium saturation, Tertiary = low saturation/grayscale

**Labels on arrows**: 
- Short (≤3 words), placed mid-arrow
- **MUST have background rect**: `<rect>` with fill matching canvas background, placed behind text
- Background rect padding: 4px horizontal, 2px vertical
- Never overlap with nodes (maintain 10px safety distance)
- When multiple arrows converge, stagger label vertical positions by 15-20px

**Arrow Routing**:
- **Prefer orthogonal routing**: vertical then horizontal segments to minimize crossings
- **Layer-based routing**: different semantic flows use different "channels" (y-offsets)
- **Crossing minimization**: route arrows around dense node clusters
- **Jump-over arcs**: when crossings unavoidable, use small arc (5px radius) at intersection point

## SVG Technical Rules

- ViewBox: `0 0 960 600` default; `0 0 960 800` tall; `0 0 1200 600` wide
- Fonts: embed via `<style>font-family: ...</style>` — no external `@import` (breaks rsvg-convert)
- `<defs>`: arrow markers, gradients, filters, clip paths
- Text: minimum 12px, prefer 13-14px labels, 11px sub-labels, 16-18px titles
- All arrows: `<marker>` with `markerEnd`, sized `markerWidth="10" markerHeight="7"`
- Drop shadows: `<feDropShadow>` in `<filter>`, apply sparingly (key nodes only)
- Curved paths: use `M x1,y1 C cx1,cy1 cx2,cy2 x2,y2` cubic bezier for loops/feedback arrows
- Clip content: use `<clipPath>` if text might overflow a node box

## SVG Generation Strategy

**CRITICAL: Tool Call Discipline**
- **NEVER call a tool without all required parameters** - this is the #1 cause of error loops
- Before ANY tool call, verify you have all required parameters ready
- If you encounter a tool error, STOP and analyze the root cause before retrying
- **Max 2 retries** for the same approach - if it fails twice, switch methods immediately

**Step 1: Estimate complexitnt nodes: N
- Count arrows: A  
- Estimated SVG lines: L = 50 (header) + N×15 + A×3 + 20 (legend)

**Step 2: Choose generation method**
- **If L < 150**: Direct Write tool (single call, most reliable)
- **If 150 ≤ L < 300**: Python script via Bash (avoids heredoc issues)
- **If L ≥ 300**: Chunked generation with Write + multiple appends

**Step 3A: Direct generation (L < 150)**
1. Generate complete SVG content as string variable
2. Self-check before writing (see SVG Self-Check Rules below)
3. Use Write tool with `file_path` and `content` parameters
4. If Write fails once: switch to Python method immediately

**Step 3B: Python script method (150 ≤ L < 300) - RECOMMENDED**
```bash
python << 'EOF'
svg_content = '''<svg xmlns="http://www.w3.org/2000/svg">
...complete SVG here...
</svg>'''

with open('output.svg', 'w', encoding='utf-8') as f:
    f.write(svg_content)
print("Generated output.svg")
EOF
```
- **Advantage**: Avoids Bash heredoc truncation/corruption issues
- **Triple quotes**: Handles complex SVG content reliably
- **No escaping needed**: Python handles quotes and special chars correctly
1. Chunk 1: Header (defs, style, background) → Write to file
2. Chunk 2-N: Content sections → Append with `echo '...' >> file.svg`
3. Final chunk: Legend + `</svg>` closing tag → Append
4. Each chunk max 80 lines to avoid truncation
5. **Verify each append**: Check file size increases after each append

**Step 4: Validation**
- Run `rsvg-convert file.svg -o NUL 2>&1` (Windows) or `rsvg-convert file.svg -o /dev/null 2>&1` (Git Bash/macOS/Linux)
- If error: apply Quick Fix Protocol (see below), retry once
- If still fails: report error to user with line number and stop

**Error Recovery Protocol**
1. **First error**: Analyze root cause, apply targeted fix
2. **Second error (same type)**: Switch generation method entirely
3. **Third error**: Stop and report to user - do not loop endlessly

## SVG Self-Check Rules

Before writing SVG, perform quick validation:

**Rule 1: Tag Balance**
- Count opening tags: `<rect`, `<text`, `<line`, `<path`, `<circle`
- Count closing: `/>` self-closing + `</rect>`, `</text>`, etc.
- Ensure balance: every opening has a closing

**Rule 2: Quote Check**
- All attribute values must be in quotes: `fill="#9dd4c7"` not `fill=#9dd4c7`
- Check pattern: `\w+=["'][^"']*["']`

**Rule 3: Special Characters**
- Text content must not contain unescaped `<`, `>`, `&`
- If present, replace with `&lt;`, `&gt;`, `&amp;`

**Rule 4: Marker References**
- All `marker-end="url(#xxx)"` must have matching `<marker id="xxx">` in defs
- Check before writing arrows section

**Rule 5: Closing Tag**
- Always end with `</svg>` on its own line
- In chunked generation, include in final chunk

## Quick Fix Protocol

When `rsvg-convert` validation fails, parse error and apply fix:

**Error: "Extra content at the end of the document"**
- Cause: Missing `</svg>` closing tag
- Fix: `echo '</svg>' >> file.svg`

**Error: "Couldn't find end of Start Tag" (line N)**
- Cause: Truncated attribute value
- Fix: Read line N, identify incomplete attribute, use Edit tool to complete it

**Error: "Invalid character" (line N)**
- Cause: Unescaped `<`, `>`, or `&` in text
- Fix: Read line N, replace with HTML entities using Edit tool

**Error: "Undefined marker" or "Failed to "**
- Cause: `marker-end="url(#arrow-xxx)"` but no `<marker id="arrow-xxx">` in defs
- Fix: Add missing marker definition to defs section

**Max retry: 1 time per error**
- If quick fix doesn't resolve, report to user with:
  - Full error message
  - Line number
  - Suggested manual fix

## Output

- **Default**: `./[derived-name].svg` and `./[derived-name].png` in current directory
- **Custom**: user specifies path with `--output /path/` or `输出到 /path/`
- **PNG export**: `rsvg-convert -w 1920 file.svg -o file.png` (1920px = 2x retina)

## Styles

| # | Name | Background | Best For |
|---|------|-----------|----------|
| 1 | **Flat Icon** (default) | White | Blogs, docs, presentations |
| 2 | **Dark Terminal** | `#0f0f1a` | GitHub, dev articles |
| 3 | **Blueprint** | `#0a1628` | Architecture docs |
| 4 | **Notion Clean** | White, minimal | Notion, Confluence, Wiki |
| 5 | **Glassmorphism** | Dark gradient | Product sites, keynotes |
| 6 | **Claude Official** | Warm cream `#f8f6f3` | Anthropic-style diagrams |
| 7 | **OpenAI Official** | Pure white `#ffffff` | OpenAI-style diagrams |

Load `references/style-N.md` for exact color tokens and SVG patterns.

## Style-to-Diagram-Type Adaptation Guide

Not all styles work equally well for every diagram type. Use this guide to pick the best style.

### Architecture Diagram
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Default choice; colorful node fills, clear layering |
| 2 Dark Terminal | Excellent | Popular for dev blogs; use colored borders on dark bg |
| 3 Blueprint | Excellent | Perfect for formal architecture docs |
| 4 Notion Clean | Good | Minimal, works for inline docs |
| 5 Glassmorphism | Good | Striking for presentations and product pages |
| 6 Claude Official | Good | Warm aesthetic, Anthropic-style presentations |
| 7 OpenAI Official | Good | Clean, precise; minimal borders, brand green accents |

### Class Diagram / ER Diagram
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Good | Colored headers per class category |
| 2 Dark Terminal | Good | High contrast for code-like diagrams |
| 3 Blueprint | Excellent | Best for formal UML documentation |
| 4 Notion Clean | Excellent | Clean, minimal; ideal for Notion-embedded diagrams |
| 5 Glassmorphism | Poor | Glass effects distract from structural content |
| 6 Claude Official | Excellent | Warm, readable; good for documentation |
| 7 OpenAI Official | Excellent | Minimal aesthetic matches UML precision |

### Sequence Diagram
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Good | Clear lifelines; activation boxes visible |
| 2 Dark Terminal | Good | Good for dev articles; dashed lifelines visible |
| 3 Blueprint | Excellent | Formal, technical documentation |
| 4 Notion Clean | Excellent | Best for Notion-embedded sequence diagrams |
| 5 Glassmorphism | Poor | Glass effects make lifelines hard to read |
| 6 Claude Official | Excellent | Warm bg, good contrast |
| 7 OpenAI Official | Excellent | Minimal, precise; ideal for API docs |

### Flowchart / Process Flow
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Default; colorful decision diamonds |
| 2 Dark Terminal | Good | Works well for dev workflow diagrams |
| 3 Blueprint | Good | Formal process documentation |
| 4 Notion Clean | Good | Clean for SOPs and inline docs |
| 5 Glassmorphism | Good | Striking for product demos |
| 6 Claude Official | Good | Warm aesthetic for presentations |
| 7 OpenAI Official | Good | Clean and minimal |

### Mind Map / Concept Map
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Colorful branches, engaging |
| 2 Dark Terminal | Good | Neon-like branches on dark bg |
| 3 Blueprint | Poor | Blueprint grid conflicts with radial layout |
| 4 Notion Clean | Excellent | Ideal for Notion brainstorming |
| 5 Glassmorphism | Excellent | Stunning visual for presentations |
| 6 Claude Official | Good | Warm, readable |
| 7 OpenAI Official | Good | Clean and minimal |

### Data Flow Diagram
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Color-coded arrows by data type |
| 2 Dark Terminal | Excellent | Glowing data paths on dark bg |
| 3 Blueprint | Excellent | Formal data flow documentation |
| 4 Notion Clean | Good | Minimal, clean |
| 5 Glassmorphism | Poor | Distracts from flow semantics |
| 6 Claude Official | Good | Readable |
| 7 OpenAI Official | Good | Precise, minimal |

### Use Case Diagram
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Good | Colorful use case ellipses |
| 2 Dark Terminal | Poor | Stick figures less visible on dark bg |
| 3 Blueprint | Excellent | Classic UML aesthetic |
| 4 Notion Clean | Excellent | Perfect for product requirement docs |
| 5 Glassmorphism | Poor | Unnecessary visual noise |
| 6 Claude Official | Excellent | Warm, professional |
| 7 OpenAI Official | Excellent | Clean, precise UML |

### State Machine Diagram
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Good | Colorful states |
| 2 Dark Terminal | Good | Glowing states and transitions |
| 3 Blueprint | Excellent | Best for formal UML state machines |
| 4 Notion Clean | Excellent | Clean for documentation |
| 5 Glassmorphism | Poor | Distracts from state transitions |
| 6 Claude Official | Excellent | Readable |
| 7 OpenAI Official | Excellent | Minimal, precise |

### Network Topology
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Colorful device icons |
| 2 Dark Terminal | Excellent | Cyberpunk-style network maps |
| 3 Blueprint | Excellent | Ideal for infrastructure docs |
| 4 Notion Clean | Good | Clean for IT documentation |
| 5 Glassmorphism | Good | Striking for presentations |
| 6 Claude Official | Good | Professional network diagrams |
| 7 OpenAI Official | Good | Clean infrastructure diagrams |

### Comparison / Feature Matrix
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Color-coded checkmarks |
| 2 Dark Terminal | Good | Works for dev tool comparisons |
| 3 Blueprint | Poor | Grid conflicts with table layout |
| 4 Notion Clean | Excellent | Perfect for Notion-embedded tables |
| 5 Glassmorphism | Poor | Distracts from tabular data |
| 6 Claude Official | Excellent | Clean, warm |
| 7 OpenAI Official | Excellent | Minimal, precise |

### Timeline / Gantt
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Colorful bars by category |
| 2 Dark Terminal | Good | Works for dev roadmaps |
| 3 Blueprint | Good | Formal project plans |
| 4 Notion Clean | Excellent | Ideal for Notion project docs |
| 5 Glassmorphism | Good | Striking for keynote presentations |
| 6 Claude Official | Good | Warm, professional |
| 7 OpenAI Official | Good | Clean timeline |

### Agent / Memory Architecture
| Style | Suitability | Notes |
|-------|------------|-------|
| 1 Flat Icon | Excellent | Colorful layers, engaging |
| 2 Dark Terminal | Excellent | Popular for AI/ML blog posts |
| 3 Blueprint | Good | Formal AI system documentation |
| 4 Notion Clean | Good | Clean for AI research notes |
| 5 Glassmorphism | Excellent | Stunning for AI product presentations |
| 6 Claude Official | Excellent | Anthropic AI aesthetic |
| 7 OpenAI Official | Excellent | OpenAI AI aesthetic |

These patterns appear frequently — internalize them:

**RAG Pipeline**: Query → Embed → VectorSearch → Retrieve → Augment → LLM → Response
**Agentic RAG**: adds Agent loop with Tool use between Query and LLM
**Agentic Search**: Query → Planner → [Search Tool / Calculator / Code] → Synthesizer → Response
**Mem0 / Memory Layer**: Input → Memory Manager → [Write: VectorDB + GraphDB] / [Read: Retrieve+Rank] → Context
**Agent Memory Types**: Sensory (raw input) → Working (context window) → Episodic (past interactions) → Semantic (facts) → Procedural (skills)
**Multi-Agent**: Orchestrator → [SubAgent A / SubAgent B / SubAgent C] → Aggregator → Output
**Tool Call Flow**: LLM → Tool Selector → Tool Execution → Result Parser → LLM (loop)

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Arrows crossing over nodes | Route around with orthogonal paths or bezier detour |
| Too many arrow colors | Max 4 semantic colors per diagram; combine similar flows |
| Unlabeled arrows | Always label; put text mid-arrow on white/dark bg rect |
| No grouping on complex graphs | Add swim-lane rects for groups >4 nodes |
| Text overflow | `text-anchor="middle"` + clip or shorten label |
| PNG missing | Run rsvg-convert immediately after SVG write; report if unavailable |
| Style mismatch | Load style reference file before generating any SVG |

## Best Practices (2026 Update)

### Text Size Hierarchy
- **Title**: 24-28px, font-weight: 700
- **Layer labels**: 16-18px, font-weight: 600
- **Node main labels**: 16-18px, font-weight: 600
- **Node sub-labels**: 13-14px, font-weight: 400
- **Arrow labels**: 12-13px, font-weight: 400
- **Legend**: 11-12px, font-weight: 400

### Arrow Label Background
**CRITICAL**: All arrow labels MUST have a background rect to prevent overlap with other elements.
```xml
<!-- Background rect (place before text) -->
<rect x="label_x - 4" y="label_y - 14" width="lh + 8" height="18" 
      fill="canvas_background_color" opacity="0.95"/>
<text x="label_x" y="label_y">label_text</text>
```

### Arrow Semantics
- **Primary flow**:  solid, high saturation color
- **Secondary flow**: 2px stroke, solid, medium saturation  
- **Async/optional**: 1.5px stroke, dashed (4,2), low saturation

### Legend Requirements
- **Placement**: Right-upper corner (x=760, y=40) or left-lower corner (x=40, y=520)
- **Content**: Must include arrow types, node types (if >2 types), color semantics
- **Style**: Match diagram style but use subtle background to distinguish

### Orthogonal Arrow Routing
To minimize crossings, use orthogonal (L-shaped) paths instead of diagonal lines:
```xml
<!-- Bad: diagonal crossing -->
<line x1="100" y1="100" x2="300" y2="300"/>

<!-- Good: orthogonal routing -->
<path d="M 100,100 L 100,200 L 300,200 L 300,300"/>
```

### Node Size Hierarchy
- **Core nodes**: 1.25x standard size, 3-4px stroke or double border
- **Secondary nodes**: Standard size (180x90), 2-2.5px stroke
- **Tertiary nodes**: Standard size, 1.5-2px stroke

