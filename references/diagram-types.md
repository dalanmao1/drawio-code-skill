# Diagram Type Presets

When the user requests a specific diagram type, apply the matching preset below for shapes, styles, and layout conventions. These presets set **structural** style keywords (e.g. ERD's `shape=table;childLayout=tableLayout`); a user style preset (see `references/style-presets.md`) layers color/font/edge/extras on top.

Read this file when:
- The user names one of these diagram types (ERD, UML class, sequence, architecture, ML/DL model, flowchart)
- You're choosing shape vocabulary or layout direction for a new diagram

## ERD (Entity-Relationship Diagram)

| Element | Style | Notes |
|---------|-------|-------|
| Table | `shape=table;startSize=30;container=1;collapsible=1;childLayout=tableLayout;fixedRows=1;rowLines=0;fontStyle=1;strokeColor=#6c8ebf;fillColor=#dae8fc;` | Each table is a container |
| Row (column) | `shape=tableRow;horizontal=0;startSize=0;swimlaneHead=0;swimlaneBody=0;fillColor=none;collapsible=0;dropTarget=0;points=[[0,0.5],[1,0.5]];portConstraint=eastwest;fontSize=12;` | Child of table, `parent=tableId` |
| PK column | Bold text: `fontStyle=1` on the row | Mark with `PK` prefix or key icon |
| FK relationship | Dashed edge: `dashed=1;endArrow=ERmandOne;startArrow=ERmandOne;` | Use ER notation arrows |
| Layout | TB, tables spaced 300px apart | Group related tables vertically |

## UML Class Diagram

| Element | Style | Notes |
|---------|-------|-------|
| Class box | `swimlane;fontStyle=1;align=center;startSize=26;html=1;` | 3-section: title / attributes / methods |
| Separator | `line;strokeWidth=1;fillColor=none;align=left;verticalAlign=middle;spacingTop=-1;spacingLeft=3;spacingRight=10;rotatable=0;labelPosition=left;points=[];portConstraint=eastwest;` | Between sections |
| Inheritance | `endArrow=block;endFill=0;` | Hollow triangle arrow |
| Implementation | `endArrow=block;endFill=0;dashed=1;` | Dashed + hollow triangle |
| Composition | `endArrow=diamondThin;endFill=1;` | Filled diamond |
| Aggregation | `endArrow=diamondThin;endFill=0;` | Hollow diamond |
| Layout | TB, classes 250px apart | Interfaces above implementations |

## Sequence Diagram

Two ways to author sequence diagrams in draw.io:

- **A. `umlLifeline` shape** — built-in UML container. Good for small/simple diagrams.
- **B. Primitive composition (recommended for non-trivial diagrams)** — header rectangle + dashed edge as lifeline + plain rectangles as activation bars + plain edges as messages. More positional control, easier to align bars/arrows precisely, works at any scale (8+ lifelines, 40+ messages, multiple scenes stacked vertically). The rules below assume primitive composition.

### Basic element table

| Element | Style | Notes |
|---------|-------|-------|
| Lifeline header (top) | `rounded=1;whiteSpace=wrap;html=1;fillColor=<color>;strokeColor=<color>;fontSize=12;fontStyle=1;` | Title box. Width 200–240px,height 50px |
| Lifeline (vertical dashed line) | `endArrow=none;html=1;dashed=1;strokeColor=<color>;strokeWidth=1;` as `edge="1"` | Source/target points share x (the lifeline center `cx`), span the full diagram height |
| Activation bar | `rounded=0;fillColor=<color>;strokeColor=<color>;html=1;` as `vertex="1"` | Width = 12, `x = cx - 6` so bar is centered on the lifeline x. z-order MUST be above lifeline (place bar `mxCell` **after** lifeline `mxCell` in the file) |
| Sync message | `endArrow=classic;html=1;strokeColor=<color>;strokeWidth=2;fontSize=11;` as `edge="1"` | Solid arrow with filled head |
| Return message | Same as sync but `dashed=1` | Dashed line for returns |
| Self-call | Edge from lifeline back to itself with two waypoints forming a hook | Source x = `cx + 6` (bar right edge), target x = `cx + 150` (free space), waypoints make the right-angle turn |
| Scene band (separator) | `text;html=1;align=left;verticalAlign=middle;fontSize=13;fontStyle=1;fillColor=<scene-bg>;strokeColor=<scene-stroke>;spacingLeft=10;` | Horizontal band spanning all lifelines, used to label and visually separate scenes |
| Summary box (per scene) | `rounded=1;whiteSpace=wrap;html=1;align=left;verticalAlign=top;` + scene-tinted fill | Single-paragraph takeaway placed under a scene |
| Layout | LR, lifelines spaced **220–280px** apart | Time flows top to bottom; allow ≥30px per message line |

### Lifeline / bar geometry rules (CRITICAL)

These are the rules that prevent visual misalignment in dense sequence diagrams:

1. **Lifeline `sourcePoint.x == targetPoint.x == cx`.** The dashed edge is strictly vertical — never put it at `cx + 6` or `cx - 6`.
2. **Activation bar `x = cx - 6, width = 12`.** This centers the 12-px-wide bar on the lifeline. Bar fill is **opaque** (no `opacity` attribute) — the bar's purpose is to visually replace the dashed line during the "active" interval.
3. **z-order: bar above lifeline, edges above bar.** Draw.io z-order is the order `mxCell`s appear in the XML. Required ordering inside `<root>`:
   ```
   <mxCell id="ll1" edge="1" ...> ← lifeline (dashed line, bottom)
   <mxCell id="A_bar_*" vertex="1" ...> ← activation bars (middle)
   <mxCell id="A1" edge="1" ...> ← message arrows (top)
   ```
   If you accidentally place bars before lifelines, the dashed line will draw over the bar.
4. **Bar fill color should be light tint of the lifeline's owning layer color** (e.g. lifeline stroke `#1565c0` → bar fill `#bbdefb`). Bar stroke matches the lifeline stroke. This makes "which actor is active" readable at a glance.
5. **Bar y-range = first-incoming-message-y to last-outgoing-message-y.** Don't pad — the bar should start exactly when the actor is called and end exactly when it returns.

### Message arrows must NOT overlap activation bars

When a message ends on a lifeline that already has an activation bar, naively pointing the arrow to `cx` puts the arrowhead **inside** the bar (the arrow disappears visually). Always anchor message endpoints to **bar edges**:

- **Left → right arrow:** `sourcePoint.x = src_cx + 6` (right edge of source bar), `targetPoint.x = tgt_cx - 6` (left edge of target bar).
- **Right → left arrow:** `sourcePoint.x = src_cx - 6`, `targetPoint.x = tgt_cx + 6`.
- **Self-call:** both points at `cx + 6` (bar right edge), waypoints define the hook.

If you generate edges first with `cx`/`cx` endpoints, an `awk`/`sed` post-pass can shift all source/target points by ±6 based on direction. Be careful **not** to shift the lifeline edges themselves (their `sourcePoint.x == targetPoint.x` — detect with that equality and skip).

### Cross-layer / specific-to-specific arrows

In layered architectures, draw two complementary arrow sets:

- **Hub arrows (`*_ops_center` → `*_ops_center`)**: the abstraction-layer story. Thick, solid, primary path color. Each layer has one center node representing its `struct *_ops`. Hub arrows go layer-to-layer at the center.
- **Specific-to-specific arrows**: the actual call story. Thin, **dashed**, layer color but darker font color. Connect a specific function at layer N (e.g. `dma_map_page` in L1) directly to its real callee at layer N+1 (e.g. `iommu_dma_map_page` in L2), often skipping the center node. Use these to show "what really happens at runtime" without losing the abstract structure shown by hub arrows.

Example coexistence in the same diagram:
```
L1_center ──━━━━━━━━► L2_center        (hub: solid, strokeWidth=3)
L1_alloc ─ ─ ─ ─ ─ ─ ► L2_iommu_dma   (specific: dashed, strokeWidth=1, with label "→ iommu_dma_alloc")
```

Also draw **return / callback arrows** when control reverses naturally (e.g. `io-pgtable.tlb_flush_walk` is a callback from L5 back to L4). Use a distinct color (red dashed works well) and an explicit label to make the inversion obvious.

### Long labels: break with `&#xa;`

Draw.io edge labels don't auto-wrap. For any label longer than ~35 ASCII chars / ~20 CJK chars, insert `&#xa;` at natural break points so the label stays within the lifeline-to-lifeline horizontal slot.

Good break points:
- After a function-call's opening paren: `dma_alloc_coherent(dev, size,&#xa;&dma_handle, GFP_KERNEL)`
- Before a `→` chain step: `iommu_dma_alloc_iova&#xa;→ alloc_iova_fast → iova_rcache_get`
- Before a parenthetical clarification: `★ arch_sync_dma_for_device&#xa;(phys, size, DMA_TO_DEVICE)`

Don't break **inside** an identifier (`dma_map_&#xa;page` is unreadable).

### Color conventions (layer-aligned)

For diagrams that pair a sequence view with a layered-architecture view (e.g. both `iommu_dma_layered.drawio` and `dma_api_sequence.drawio` of the same system), **reuse the same color per layer** across both diagrams. This makes the two views composable — the reader sees a purple bar in the sequence and immediately knows it's L3 IOVA.

Reference palette used in `dma_api_sequence`:

| Layer | Stroke | Light fill (bar) |
|-------|--------|------------------|
| Driver / external client | `#1565c0` | `#bbdefb` |
| L1 DMA API | `#2e7d32` | `#c8e6c9` |
| L2 ops dispatch (iommu_dma_ops) | `#c2185b` | `#f8bbd0` |
| L3 IOVA / allocator | `#6a1b9a` | `#e1bee7` |
| L4 platform IOMMU ops | `#1565c0` | `#bbdefb` |
| L5 page-table / format | `#e65100` | `#ffe0b2` |
| Cache / utility / buddy | `#f9a825` | `#fff59d` |
| HW / device | `#212121` | `#9e9e9e` |
| Unmap / error / fault | `#c62828` | `#ffcdd2` |

### Multi-scene sequence diagram

When showing several API calls in the same diagram (e.g. `alloc` + `map` + `unmap` of one subsystem), stack scenes vertically rather than making one mega-flow:

- One **scene band** (full-width colored bar with scene title) at the top of each scene
- Activation bars **per scene** (start/end y matches scene's first/last message)
- A **summary box** under each scene with the "what just happened" takeaway
- Lifelines are shared across all scenes (one set of vertical dashes spanning the page)
- Reserve at least 60px of vertical gap between adjacent scenes (band + summary takes ~120px on its own)

Total page height: `~600px per scene + 300px for legend/comparison footer`. For 4 scenes, plan a `pageHeight` of around 3000.

### Footer (always include)

After all scenes, add:
1. **Legend** (left): line-style key, arrow-style key, color-to-layer key.
2. **Comparison table** (right, Courier New font): rows = APIs, columns = "alloc?", "alloc IOVA?", "iommu_map?", "cache maintenance?", "lifetime", "typical use" — concise at-a-glance contrast.

## Architecture Diagram

| Element | Style | Notes |
|---------|-------|-------|
| Layer/tier | `swimlane;startSize=30;` | Containers for grouping: Client / API / Service / Data |
| Service | `rounded=1;whiteSpace=wrap;html=1;` + tier color | Use color palette by tier |
| Database | `shape=cylinder3;whiteSpace=wrap;html=1;` | Green palette |
| Queue/Bus | `rounded=1;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;` | Yellow — place centrally for hub pattern |
| Gateway/LB | `shape=mxgraph.aws4.resourceIcon;` or `rounded=1;` with orange | Orange palette |
| External | `rounded=1;dashed=1;fillColor=#f5f5f5;strokeColor=#666666;` | Dashed border for external systems |
| Layout | TB or LR by tier count; ≥4 tiers → TB | Hub nodes centered |

## ML / Deep Learning Model Diagram

For neural network architecture diagrams — ideal for papers targeting NeurIPS, ICML, ICLR.

| Element | Style | Notes |
|---------|-------|-------|
| Layer block | `rounded=1;whiteSpace=wrap;html=1;` + type color | Main building block |
| Input/Output | `fillColor=#d5e8d4;strokeColor=#82b366;` | Green |
| Conv / Pooling | `fillColor=#dae8fc;strokeColor=#6c8ebf;` | Blue |
| Attention / Transformer | `fillColor=#e1d5e7;strokeColor=#9673a6;` | Purple |
| RNN / LSTM / GRU | `fillColor=#fff2cc;strokeColor=#d6b656;` | Yellow |
| FC / Linear | `fillColor=#ffe6cc;strokeColor=#d79b00;` | Orange |
| Loss / Activation | `fillColor=#f8cecc;strokeColor=#b85450;` | Red/Pink |
| Skip connection | `dashed=1;endArrow=block;curved=1;` | Dashed curved arrow |
| Tensor shape label | Add shape annotation as secondary label: `value="Conv2D&#xa;(B, 64, 32, 32)"` | Use `&#xa;` for multi-line |
| Layout | TB (data flows top→bottom), layers 150px apart | Group encoder/decoder as swimlanes |

**Tensor shape convention:** annotate each layer with input/output tensor dimensions in `(B, C, H, W)` or `(B, T, D)` format. Place dimensions as the second line of the label using `&#xa;`.

## Flowchart (enhanced)

| Element | Style | Notes |
|---------|-------|-------|
| Start/End | `ellipse;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;` | Green oval |
| Process | `rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;` | Blue rectangle |
| Decision | `rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;` | Yellow diamond |
| I/O | `shape=parallelogram;perimeter=parallelogramPerimeter;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;` | Orange parallelogram |
| Subprocess | `rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;` + double border | Purple |
| Yes/No labels | `value="Yes"` / `value="No"` on decision edges | Always label decision branches |
| Layout | TB, 200px vertical gap | Decisions branch LR, merge back to center |
