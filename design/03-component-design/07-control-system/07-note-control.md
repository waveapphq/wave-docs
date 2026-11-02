#### 3.7.7 Note Control

The note control is a canvas annotation element that serves as visual documentation within workflows. It belongs to the `misc` category - a dedicated palette section for non-executable utility elements. Unlike other controls, the note does not perform any computation or side effects during execution; it passes through silently, allowing edges to flow through it without affecting workflow behavior.

**Control Purpose and Use Cases**: The note control allows test designers to annotate workflows with contextual information directly on the canvas. Primary use cases include documenting complex flow logic near the relevant blocks, marking TODO or placeholder areas in workflows under construction, providing instructions or caveats for other team members viewing the workflow, and labeling sections of large workflows for navigation clarity. Notes are visible in the canvas but invisible to the execution engine - they execute as a no-op pass-through.

**Parameter Schema**: The control accepts two optional parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | string | `""` | Free-form note content (multiline) |
| `color` | string enum | `"slate"` | Visual color preset for the note card |

The `color` parameter accepts one of five values: `yellow`, `pink`, `blue`, `green`, or `slate`. Invalid color values fall back to `slate` at render time; validation reports them as warnings on load context and errors on run context.

**Execution Behavior**: The note control's `execute()` method is a no-op - it returns immediately with no output variables and no side effects. When connected in a flow via its input and output handles, execution passes through transparently. This matches the "invisible to engine" design intent: the note occupies canvas space and participates in edge routing but contributes nothing to execution state.

**Validation**: The note has minimal validation. The `text` parameter is fully optional - an empty note is valid. The `color` parameter is validated against the enum of accepted values. No required-parameter checks apply since both parameters have sensible defaults.

**Category and Palette Placement**: The note is the first control in the `misc` category, which appears as a separate section in the block palette below CONTROLS and PLUGINS. The category uses slate coloring (`text-slate-400`) to convey its utility/non-executable nature.
