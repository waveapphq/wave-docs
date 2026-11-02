### 4.1 REST API Design

The WAVE REST API provides comprehensive access to all system functionality through a well-organized, RESTful interface. The running server serves the complete API specification in OpenAPI format at `/openapi.json` and `/openapi.yaml`, with interactive views at `/docs` and `/redoc`.

#### 4.1.1 API Structure

The API is organized into the following functional groups:

**Foundation & Infrastructure**
- Health & Status: System health checks, version info, metrics, and dashboard summaries
- Configuration: System configuration management with key-value access

**Core Entity Management**
- Tests: Test workflow operations, versioning, variable management, and workflow validation
- Suites: Test suite management, versioning, and test ordering
- Regressions: Regression suite operations, versioning, and suite composition

**Execution & Monitoring**
- Executions: Execution control (start/stop), progress monitoring, logs, and statistics
- Debug: Interactive debugging with breakpoints, step-over/step-into, and block re-execution

**Supporting Services**
- Variables: Hierarchical variable management, expression evaluation, and validation
- Profiles: Connection profile management for target environments

**Data Operations**
- Import/Export: Workspace and entity import/export with validation
- Artifacts: Test artifact management and download
- Reports: Report generation from execution artifacts

Each API group provides consistent CRUD operations with standard HTTP methods (GET, POST, PUT, PATCH, DELETE) and follows RESTful conventions for resource naming and status codes.
