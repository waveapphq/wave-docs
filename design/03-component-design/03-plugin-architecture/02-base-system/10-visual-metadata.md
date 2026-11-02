#### 3.3.2.10 Plugin Visual Metadata

##### REQ-3.4.4.1 Plugin icon name field

Each plugin declares a visual representation through an icon name, enabling consistent rendering across all user interfaces including the workflow canvas, palette panel, and library browsers.

**Icon Declaration**: Plugins include an icon identifier in their definitions that references a visual representation from the system's icon library. This identifier is a string name that maps to a specific icon in the predefined icon set. The icon appears wherever the plugin is displayed, creating visual consistency and rapid recognition across different contexts.

**Built-in Plugin Definitions**: Built-in plugins follow the same architectural pattern as controls, with definitions hardcoded in the frontend. Each built-in plugin definition includes its identifier, display name, category, icon name, and color class. This approach ensures immediate availability without API calls and maintains consistency with the control system architecture.

**Palette Integration**: The workflow canvas palette renders plugins using their declared icons. Users see plugins organized by category with consistent visual treatment, where each plugin's icon provides immediate recognition of its functionality. Network plugins display connectivity-related imagery, database plugins show data-centric icons, and system plugins use terminal or file-related visuals.

**Canvas Rendering**: When plugins are dropped onto the workflow canvas, the icon appears prominently within the node representation. This visual anchor helps users quickly identify plugin types within complex workflows, particularly when reviewing execution history or debugging test failures.

##### REQ-3.4.4.2 Icon names from predefined set

Plugin icons must reference names from a controlled vocabulary that maps to the system's icon library, ensuring consistent availability and rendering across all platforms.

**Lucide Icon Library**: The system standardizes on Lucide React icons as the icon source, providing a comprehensive set of consistently styled icons suitable for technical and workflow visualization. Plugin definitions select icon names from this predefined set rather than providing custom imagery, ensuring visual consistency across all plugins.

**Centralized Icon Registry**: All icon names are registered in a centralized registry that maps string names to icon components. This registry serves both controls and plugins, providing a single source of truth for icon availability. The registry includes icons for all built-in plugin categories: network, remote, database, system, kubernetes, helm, and data.

**Category-Specific Icons**: Icons align with plugin categories to create intuitive visual groupings. Network plugins use icons like Globe, Radio, and Cable. Database plugins use icons like Database, Users, and Search. System plugins use icons like FileTerminal, Terminal and FileCode. Kubernetes plugins pair a connection icon with one distinct icon per verb, such as Cable for the connection and PackageSearch, Gauge, Terminal and ArrowRightLeft for verbs. Data plugins use file format icons. This categorical consistency aids user navigation and recognition.

**Fallback Handling**: When a plugin references an unknown or empty icon name, the system renders a neutral fallback icon rather than failing. This graceful degradation ensures configuration errors do not disrupt the user interface while still surfacing an indication that icon configuration may need attention.
