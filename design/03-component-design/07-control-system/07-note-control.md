#### 3.7.7 Note Control

The note control is a canvas annotation element that serves as visual documentation within workflows. It belongs to the `misc` category - a dedicated palette section for non-executable utility elements. The note does no work of its own: it has an input and an output handle so it can sit in a flow as a placeholder for a future block, and the run passes straight through it.

**Control Purpose and Use Cases**: The note control allows test designers to annotate workflows with contextual information directly on the canvas. Primary use cases include documenting complex flow logic near the relevant blocks, marking TODO or placeholder areas in workflows under construction, providing instructions or caveats for other team members viewing the workflow, and labeling sections of large workflows for navigation clarity. Notes are visible in the canvas and do nothing when a run passes through them.

**Parameter Schema**: The control declares two optional fields, kept, stored and save-checked like any Block field (REQ-3.1.1.5) and edited on the canvas, since a Note has no configuration window:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | string | `""` | Free-form note content (multiline) |
| `color` | string enum | `"slate"` | Visual color preset for the note card |

The `color` parameter accepts one of five values: `yellow`, `pink`, `blue`, `green`, or `slate`. The save check refuses any other color, as it refuses any value outside a field's allowed values; a stored color it does not know renders as `slate`.

**Execution Behavior**: A note does nothing when the run reaches it. Connected in a flow through its input and output handles, it passes the flow straight on, producing no output variables and no side effects, at the top of the flow and inside any body, section, branch or lane. Where paths join, as after a conditional, it shows one input for each connection coming in, as a block does. A note with no connections is never reached.

**Validation**: The note has minimal validation. The `text` parameter is fully optional - an empty note is valid. The `color` parameter is validated against the enum of accepted values. No required-parameter checks apply since both parameters have sensible defaults.

**Category and Palette Placement**: The note is the first control in the `misc` category, which appears as a separate section in the block palette below CONTROLS and PLUGINS. The category uses slate coloring (`text-slate-400`) to convey its utility/non-executable nature.
