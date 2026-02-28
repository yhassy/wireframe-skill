---
name: wireframe
description: Generate JSON wireframe with HTML preview
---

# /wireframe

Generate a JSON wireframe definition and render it as a self-contained HTML preview.

## Mode: Generate

## Input

Two modes:

1. **Conversation**: Describe the layout you want to wireframe. Claude generates JSON from the description.
2. **design-id argument**: `wireframe {design-id}` — reads `designs/{design-id}/output/design-spec.md` and extracts layout structure from the Information Architecture section.

## Output

Two files per screen:

- `{name}.wireframe.json` — AI-readable wireframe definition
- `{name}.wireframe.html` — self-contained HTML preview (open in browser)

**HTML preview features:**
- Collapsible right panel showing live JSON (syncs on every edit)
- Drag-and-drop reordering within the same parent
- Save button (Chrome/Edge — uses File System Access API, first save opens picker, subsequent saves write directly)
- Copy JSON and Download JSON buttons (all browsers)
- Panel collapse state persists across sessions via localStorage

**Output path:**
- With design-id: `designs/{design-id}/output/wireframe/` (automatic)
- Without: prompted — ask user for directory path

## Execution Steps

### Step 1: Determine input

If a `{design-id}` argument is provided:
1. Read `designs/{design-id}/output/design-spec.md`
2. Extract layout structure from the Information Architecture section
3. Identify all views/screens described

If no argument:
1. Use the conversation context to understand the desired layout
2. Ask clarifying questions if the layout description is ambiguous

### Step 2: Load cognitive model and generate wireframe JSON

1. Load the wireframe designer cognitive model from `.claude/skills/wireframe/wireframe-designer.md`
2. Follow the perception sequence (Phases 1–7) to reason about the layout before producing JSON
3. Produce a JSON object following the schema below

**Schema rules** (mechanical — these constrain the JSON format, not design judgment):

- Every node MUST have a `name`
- Layout containers MUST have `direction` and `children`
- Leaf elements SHOULD have a `type`
- The JSON maps 1:1 to CSS flexbox — think in flex terms
- Apply the sizing constraints from the Element types table. In particular: buttons, links, inputs, tags, and inline text must never have fixed `width` — use `height` only and let content determine width

### Step 3: Determine output location

Derive the filename from the JSON `name` field (slugified, e.g., "KAI 回答ビュー" → `kai-answer-view`).

**If `design-id` was provided:**
- Output directory is `designs/{design-id}/output/wireframe/` (no prompt needed)

**If no `design-id`:**
- Ask the user where to save the JSON: "Where should I save the wireframe JSON? Provide a directory path."
- Use `AskUserQuestion` with a simple text prompt
- Create the directory if it doesn't exist

Both `.wireframe.json` and `.wireframe.html` go in the same output directory.

### Step 4: Generate HTML preview

Use a Bash command to splice the JSON into the template. This avoids outputting the ~1000-line template through the Write tool (which is slow because every line becomes output tokens):

```bash
python3 -c "
t = open('.claude/skills/wireframe/wireframe-template.html').read()
d = open('{json_path}').read()
open('{html_path}', 'w').write(t.replace('const WIREFRAME_DATA = null;', 'const WIREFRAME_DATA = ' + d + ';'))
"
```

Replace `{json_path}` and `{html_path}` with the actual output paths from Step 3. Use absolute paths.

**Do NOT** read the template with the Read tool or write the HTML with the Write tool. The Python script handles both file operations directly on disk.

### Step 5: Output

1. Write both `.wireframe.json` and `.wireframe.html` files to the output directory from Step 3
2. Open the HTML preview in the browser: `open {path}.wireframe.html`
3. Confirm to the user:
   ```
   Wireframe generated:
   - JSON: {path}.wireframe.json
   - Preview: {path}.wireframe.html (opened in browser)
   ```

---

## JSON Wireframe Schema

```json
{
  "name": "Screen Name",
  "viewport": { "width": 1280, "height": 800 },
  "root": {
    "name": "Root",
    "direction": "vertical",
    "children": []
  }
}
```

### Top-level fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Screen/wireframe name |
| `viewport` | `{ width, height }` | Yes | Canvas dimensions in px |
| `root` | node | Yes | Root layout node |

### Node fields

**Identity:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Node label (shown in wireframe) |
| `type` | string | No | Element type for placeholder rendering |
| `variant` | string | No | Text hierarchy: `"display"` (32px), `"heading"` (24px), `"caption"` (12px). Default body (16px). Only for `type: "text"`. |

**Layout (for containers with children):**

| Field | Type | Default | CSS equivalent |
|-------|------|---------|---------------|
| `direction` | `"vertical"` / `"horizontal"` | `"vertical"` | flex-direction |
| `children` | node[] | — | child elements |
| `gap` | number | 0 | gap (px) |
| `padding` | number or [v, h] | 0 | padding (px) |
| `justify` | string | `"flex-start"` | justify-content |
| `align` | string | `"stretch"` | align-items |
| `wrap` | boolean | false | flex-wrap: wrap |

**Sizing:**

| Field | Type | Default | CSS equivalent |
|-------|------|---------|---------------|
| `width` | number | — | width (px) |
| `height` | number | — | height (px) |
| `grow` | boolean | false | flex: 1 |
| `maxWidth` | number | — | max-width (px) |

### Element types

| Type | Rendered as | Use for | Sizing |
|------|------------|---------|--------|
| `"text"` | Gray text label | Labels, headings, body text | `height` only. No `width` — text reflows naturally |
| `"image"` | Gray box with image icon | Photos, thumbnails, hero images | `width` + `height`, or `height` + `grow` |
| `"icon"` | Small gray circle | Action icons, status indicators | Fixed `width` + `height` always |
| `"button"` | Rounded rectangle | CTAs, primary actions | `height` only. Never `width` — label length varies |
| `"link"` | Underlined text | Navigation items, secondary actions, text links | `height` only. Never `width` — label length varies |
| `"input"` | Bordered rectangle | Search fields, form inputs, dropdowns | `height` + `grow` or `maxWidth`. Never fixed `width` alone |
| `"card"` | Bordered box with shadow | Content cards, list items | `height` + `grow`, or `height` alone in vertical lists |
| `"table"` | Header row + placeholder data rows | Data tables, lists with columns | `grow` or omit. Column widths via `columns[].width` / `columns[].grow` |
| `"divider"` | Thin horizontal line | Section separators | No sizing needed |
| (none) | Transparent container | Structural grouping only | `grow` for flexible, `width` for fixed (e.g., sidebar) |

### Table properties

Tables use `columns` and `rows` instead of `children`:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `columns` | array | — | Column definitions (required for table) |
| `columns[].name` | string | — | Column header text |
| `columns[].grow` | boolean | false | Flexible width (flex: 1) |
| `columns[].width` | number | — | Fixed width in px |
| `rows` | number | 3 | Number of placeholder data rows |

```json
{
  "name": "Users Table",
  "type": "table",
  "columns": [
    { "name": "Name", "grow": true },
    { "name": "Email", "grow": true },
    { "name": "Role", "width": 120 },
    { "name": "Status", "width": 100 }
  ],
  "rows": 5
}
```

### Style properties (optional)

Visual properties for downstream consumers. These are **purely additive** — a wireframe JSON with zero style properties is 100% valid and renders as a monochrome wireframe (current behavior, unchanged). The wireframe HTML renderer ignores style properties.

The `/wireframe` skill itself produces plain wireframes. Style properties are added by `/ui-prompts` when a design system is specified, creating a "styled wireframe" that downstream consumers (pencil-generate, stitch-generate, Figma Make) use directly.

| Field | Type | Description | Applies to |
|-------|------|-------------|------------|
| `fill` | string (hex) | Background fill color | Containers, cards, buttons |
| `color` | string (hex) | Text/icon foreground color | Text, icons, links |
| `fontSize` | number | Font size in px | Text |
| `fontWeight` | number | Font weight (400, 600) | Text |
| `borderRadius` | number | Corner radius in px | Containers, cards, buttons, inputs |
| `stroke` | `{ color, width }` | Border definition | Containers, cards, inputs |
| `shadow` | `{ x, y, blur, color }` | Drop shadow | Containers, cards |
| `opacity` | number (0–1) | Element opacity | Any node |

**Rules:**
- Values MUST be resolved (hex colors, px numbers) — never token names
- Absence means "use consumer defaults" — omit properties you don't need to specify
- `fill` on a container = background; `color` on text = foreground (avoid ambiguity)
- `stroke` and `shadow` use object notation for clean parsing

**Annotation fields:**

| Field | Type | Description |
|-------|------|-------------|
| `_solves` | string | Traces this element to a PRD problem or user need. Metadata only — ignored by all renderers. |

**Canonical schema:** This JSON schema is the canonical layout format used across the pipeline. The `## Layout` section in UI prompt files (`prompts/*.md`) uses this same schema inline, optionally with style properties. Skills that generate prompts (`/ui-prompts`, `/style-exploration`, `/figma-variation`) produce wireframe JSON. Skills that consume prompts (`/pencil-generate`, `/stitch-generate`) parse it directly.

---

## Example

A dashboard with proper typographic hierarchy, spacing rhythm, and proportional heights:

```json
{
  "name": "Dashboard",
  "viewport": { "width": 1280, "height": 800 },
  "root": {
    "name": "Page",
    "direction": "vertical",
    "children": [
      {
        "name": "Header",
        "direction": "horizontal",
        "height": 56,
        "padding": [0, 24],
        "justify": "space-between",
        "align": "center",
        "children": [
          { "name": "Logo", "type": "image", "width": 100, "height": 28 },
          { "name": "Search", "type": "input", "grow": true, "maxWidth": 400, "height": 40 },
          { "name": "Avatar", "type": "icon", "width": 32, "height": 32 }
        ]
      },
      {
        "name": "Body",
        "direction": "horizontal",
        "grow": true,
        "children": [
          {
            "name": "Sidebar",
            "width": 240,
            "direction": "vertical",
            "padding": 16,
            "gap": 4,
            "children": [
              { "name": "Overview", "type": "link", "height": 40 },
              { "name": "Projects", "type": "link", "height": 40 },
              { "name": "Settings", "type": "link", "height": 40 }
            ]
          },
          {
            "name": "Main",
            "grow": true,
            "direction": "vertical",
            "padding": 32,
            "gap": 24,
            "children": [
              { "name": "Dashboard", "type": "text", "variant": "display", "height": 48 },
              {
                "name": "Stats Row",
                "direction": "horizontal",
                "gap": 16,
                "children": [
                  { "name": "Total Users", "type": "card", "grow": true, "height": 100 },
                  { "name": "Revenue", "type": "card", "grow": true, "height": 100 },
                  { "name": "Active Today", "type": "card", "grow": true, "height": 100 }
                ]
              },
              {
                "name": "Recent Activity",
                "direction": "vertical",
                "gap": 12,
                "children": [
                  { "name": "Recent Activity", "type": "text", "variant": "heading", "height": 36 },
                  {
                    "name": "Activity Table",
                    "type": "table",
                    "columns": [
                      { "name": "User", "grow": true },
                      { "name": "Action", "grow": true },
                      { "name": "Status", "width": 120 },
                      { "name": "Time", "width": 140 }
                    ],
                    "rows": 6
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

**Cognitive reasoning behind this wireframe:**

- **Phase 3 (Task & Scanning):** Dashboard = monitoring intent → F-pattern → top-to-bottom primary content flow, important information along left edge and top
- **Phase 4 (Grid):** Two major zones — sidebar (240px, fixed, persistent navigation) + content area (`grow: true`). Sidebar exists because content inventory includes navigation that aids the monitoring task.
- **Phase 5 (Hierarchy):** Three levels — `display` for page identity ("Dashboard"), `heading` for section identity ("Recent Activity"), body (default) for data content. Caption reserved for table column headers (chrome, not content). Sidebar nav uses `link` (navigation aids the task but isn't the task itself). Primary CTAs in the content area would use `button`.
- **Phase 6 (Spacing):** Proximity tightens inward — root padding 32 > section gap 24 > card gap 16 > internal gap 12. Each step inward signals increasing relatedness. Container padding (32) ≥ child gap (24) everywhere.
- **Phase 7 (Rhythm):** Progressive density — spacious at top (title, 48px display text), medium in middle (stat cards, 100px), dense at bottom (table with 6 data rows). Heights accommodate content proportionally.
