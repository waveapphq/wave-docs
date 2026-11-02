#### 3.8.2 Compilation and Automatic Layout

Compilation turns a recipe into a genuine canvas test: the same document shape, the same block and connection model, the same storage and versioning as a test built by hand in the workbench. A compiled test carries no capability a hand-built one lacks and is fully editable in the workbench afterwards. Compilation is also where the platform's guarantees attach - a recipe that does not produce a valid test is rejected, never persisted.

##### REQ-5.2.7.4 Generated wiring and automatic layout

**Wiring generation.** The compiler derives all connectivity from the recipe's structure: sequential steps chain in order, construct children wire into their construct's sections and branches, and construct completions rejoin the parent flow. Multi-section constructs receive their section anchoring from the construct's own declared structure, so generated wiring is always anchored the way the canvas expects. A conditional whose sections do not cover every outcome - an if without an else, a switch without a default - still continues: the uncovered path is wired explicitly to the step that follows, exactly as a person would wire it, so no authored step is ever left unreachable. The compiler refuses to emit any graph containing a block that cannot be reached from the start of the flow - a wiring fault surfaces as a validation finding, never as a silently dead block.

**Deterministic layout.** The compiler places every block using a layout engine that follows the canvas's natural conventions: the main flow descends top to bottom on a steady vertical rhythm, branches fan out into side-by-side columns beneath their construct and rejoin below it, and loop and section bodies are grouped visibly within their construct's column. The engine uses each block kind's real rendered dimensions - wider function blocks, multi-section constructs whose width grows with their section count - so blocks never overlap and connections remain readable. Layout is a pure function of the flow's structure: the same recipe always yields the same picture.

**Block sizing.** A block a recipe creates is sized to fit its name, bounded by the canvas resize limits and never narrower than the block's own handle row, so a step's name is legible on the canvas the moment it is generated rather than after a person widens it. A block placed by hand still starts at the canvas minimum width, because its name is not known when it is dropped.

**Stability under amendment.** When a revision amends an existing test, layout is stability-preserving: blocks untouched by the revision keep their positions, including positions a person dragged into place, and only new or structurally moved steps are placed. A width a person set by resizing is kept for the same reason, while a block still at its generated width is re-sized when its name changes. The author's spatial memory of a test survives assisted edits.

**Geometry is generated, never authored.** Positions and sizes are the compiler's output, not part of the recipe: a recipe never carries them, and a readback of a compiled test does not emit them.

**Layout as a platform capability.** The layout engine belongs to the workflow domain, not to the MCP. It is defined over the canvas model itself, so any surface that holds a workflow can invoke it; the MCP compiler is one consumer.

##### REQ-5.2.7.5 Validation relay

Submissions pass two validation stages. Recipe-level validation checks the document against the format and the catalog: unknown block kinds, missing or mistyped settings, references to undeclared variables, malformed structure. Platform-level validation is the same save pipeline the UI runs, applied to the compiled result. Errors from both stages are reported in recipe coordinates - which step, which setting, what is wrong - with a proposed fix wherever the platform knows one and a plainly stated blocker where it does not. The authoring path over MCP is therefore a fully validated ingress: nothing persists in a state the UI's own save would refuse.
