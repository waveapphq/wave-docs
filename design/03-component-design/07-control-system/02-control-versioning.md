#### 3.7.2 Control Versioning

Controls follow semantic versioning enabling version evolution while maintaining stable workflow references. Controls are a fixed set of built-in constructs where versioning applies to behavior and parameter changes within existing control types.

##### REQ-3.5.0.5 Independent semantic versioning per control type

Each control type maintains its own version lineage independent of other controls. The Delay control may reach version 2.0.0 while If/Else remains at 1.0.0, reflecting independent evolution trajectories. Version progression follows MAJOR.MINOR.PATCH conventions: MAJOR for breaking changes (parameter removal, altered execution semantics), MINOR for backward-compatible additions (new optional parameters), PATCH for bug fixes without interface changes. Each version includes metadata documenting release date, changelog entries, and compatibility notes.

##### REQ-3.5.0.6 Test/Block models support control references (controlId + controlVersion)

Workflows reference controls through explicit identifier and version pairs, enabling version pinning and controlled upgrades. Older workflows continue using pinned versions regardless of system updates, ensuring test reproducibility.

##### REQ-3.5.0.7 Multiple control versions available simultaneously

The control registry supports multiple versions of each control type concurrently. Each version maintains its own complete definition including metadata and parameter schema. Different workflows can execute using different versions of the same control within the same session without interference. Teams evaluate new versions in isolation and update workflows incrementally without forced synchronization.
