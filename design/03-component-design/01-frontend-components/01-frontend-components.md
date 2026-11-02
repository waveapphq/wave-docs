### 3.1 Frontend Components

#### 3.1.1 Web GUI Architecture (Primary Test Engineering Interface)
The Web GUI is the **primary interface for test engineering** and is built as a single-page application with comprehensive test creation and management capabilities:

- **Workflow Management**: Visual workflow canvas using React Flow for drag-and-drop test composition and parameter configuration - this is where all test creation happens
- **Execution Monitoring**: Real-time execution visualization, debug controls, variable watching, and breakpoint management
- **Configuration Editors**: Multi-format editors for YAML/JSON/XML with syntax validation
- **Reporting & Analytics**: HTML report viewer, artifact browser, and test result analytics

#### 3.1.2 Key Frontend Technologies
- **React 19**: Latest features for concurrent rendering
- **React Flow**: Visual workflow builder
- **Zustand**: Lightweight state management
- **TypeScript**: Type safety and IDE support
- **Vite**: Fast build tooling
- **CodeMirror**: Code editing capabilities