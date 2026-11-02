##### REQ-3.4.2.4 Plugin versioning

Plugins follow semantic versioning enabling version evolution.

**Semantic Version Enforcement**: All plugins must declare semantic versions (MAJOR.MINOR.PATCH) at creation and increment versions appropriately when changes are made. Interface changes (parameter additions, removals, type changes) require MAJOR version increment signaling breaking changes. Behavior additions (new optional parameters, new capabilities maintaining existing behavior) require MINOR version increment. Bug fixes and internal improvements without behavior changes require PATCH version increment. This disciplined versioning enables consumers to understand compatibility implications from version numbers alone.

**Version Metadata**: Each plugin version includes metadata documenting the version's characteristics. Version release date tracks when the version was created. Changelog entries describe what changed compared to previous versions. Compatibility notes document minimum required system version, deprecated features in this version, and migration guidance for breaking changes. This metadata helps users understand version progression and plan upgrades.

**Multiple Version Coexistence**: The system supports loading multiple versions of the same plugin simultaneously. Workflows reference specific plugin versions. This coexistence enables gradual migration where old workflows continue using stable plugin versions while new workflows adopt newer versions.

**Deprecation Workflow**: Plugins can mark versions as deprecated. Deprecated versions include deprecation notices explaining why deprecation occurred and recommended replacement versions.

**Version Independence**: When a developer adds a new major version of a plugin, the previous versions remain available in the system. Tests and blocks continue using their pinned plugin versions without requiring immediate updates. Test designers can evaluate new plugin versions independently and update their workflows when appropriate, enabling gradual adoption without forcing synchronized updates across the test library.