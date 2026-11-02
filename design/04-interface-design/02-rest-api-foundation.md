### 4.2 REST API Foundation

#### 4.2.1 FastAPI Application Architecture

##### REQ-5.2.3.1 FastAPI application instance with ASGI server support

The application instance serves as the central entry point for the WAVE backend service. It provides:

**Application Instance**
- Singleton application object managing the entire REST API lifecycle
- ASGI-compatible interface enabling deployment with production servers (uvicorn, hypercorn, gunicorn with uvicorn workers)
- Lifecycle event handlers for initialization and shutdown operations
- Centralized router registration for all API endpoint groups

**Server Support**
- Development mode with auto-reload for rapid iteration
- Graceful shutdown handling with cleanup operations

**Application Lifecycle**
- Startup phase: initialize system resources, validate configuration, prepare storage connections
- Request handling: dispatch to appropriate routers and middleware
- Shutdown phase: cleanup resources, close connections, finalize pending operations

The application instance integrates all middleware layers, router modules, and configuration management into a cohesive service ready for ASGI server deployment.

##### REQ-5.2.3.2 Application configuration management

The configuration management system provides centralized control over application settings and environment-specific parameters. It enables environment variable integration.

**Configuration Architecture**
- Centralized configuration object managing all application settings
- Hierarchical configuration structure supporting nested settings groups
- Type-safe configuration with validation at load time
- Default values overridden by the configuration file, then by environment variables

**Configuration Sources**
- Environment variables for deployment-specific settings (port, host, log level)
- One configuration file for structured application settings (YAML), written back when an admin saves a settings page (REQ-4.7.16)
- Computed configuration values derived from multiple sources

**Key Configuration Categories**
- Server settings: API host, port, timeout configurations
- Storage settings: artifacts directory path, file storage locations, backup paths
- Logging settings: log level, format
- Integration settings: external service endpoints, timeouts, retry policies
- Security settings: CORS origins, credential storage paths, encryption parameters

**Configuration Access Patterns**
- Read-only access for most application components via configuration service
- Configuration validation before applying changes to prevent invalid states

**Environment Integration**
- Automatic environment variable detection and parsing
- Standard naming convention for environment variables (WAVE_* prefix)
- Type coercion for environment variable values (string to int, bool, list)
- One configuration file per WAVE, found in its installation folder or named by an environment variable (REQ-4.7.15)

**Runtime Management**
- API endpoints for reading current configuration state

The configuration system ensures consistent behavior across installations while allowing deployment-specific customization through environment variables and configuration files.

#### 4.2.2 Middleware Stack

##### REQ-5.2.3.3 CORS middleware for cross-origin requests

The CORS (Cross-Origin Resource Sharing) middleware enables the React frontend to communicate with the FastAPI backend across different origins during development and deployment. It provides secure cross-origin access control while maintaining security boundaries.

**CORS Architecture**
- Middleware layer integrated into the FastAPI application stack
- Configuration-driven origin allowlist
- Preflight request handling for complex cross-origin operations
- Credential support

**Allowed Origins Management**
- Origins configured via application configuration system
- Environment variable override capability (WAVE_SECURITY_CORS_ORIGINS)
- Explicit origin list for production deployments
- Configuration file specification for deployment-specific origins

**Request Handling**
- Automatic preflight (OPTIONS) request responses
- Header injection for allowed origins, methods, and credentials
- Method allowlist covering standard REST operations (GET, POST, PUT, DELETE, PATCH)
- Header allowlist for content negotiation

**Configuration Integration**
- CORS origins stored in security configuration section
- One origin list per WAVE, kept in its config file (REQ-4.7.15)
- Validation of origin format and protocol during configuration load

The CORS middleware ensures that the React frontend can communicate with the backend API while maintaining appropriate security boundaries.

##### REQ-5.2.3.4 Global error handling middleware with standardized error responses

The global error handling middleware provides consistent, structured error responses across all API endpoints. It intercepts exceptions and HTTP errors throughout the request lifecycle, transforming them into standardized JSON responses that clients can reliably parse and present to users.

**Error Handling Architecture**
- Middleware layer intercepting all unhandled exceptions in the request pipeline
- Exception-to-HTTP status code mapping for common error categories
- Automatic error response formatting matching the standardized schema
- Request context preservation for error tracking and debugging
- Every response that error handling produces passes through request logging, so a failed request is logged like any other

**Error Response Structure**
- Standardized JSON envelope with success flag, error details, and metadata
- Error detail object containing error code, human-readable message, and optional context
- Response metadata including timestamp, API version, and unique request identifier
- The identifier is the request's one identifier: the same value is in the response headers and in every log line of that request
- Consistent schema across all error conditions for predictable client handling
- Type-safe error codes enabling programmatic error classification

**Exception Categorization**
- Validation errors: HTTP 400 with detailed field-level validation failures
- Authorization errors: HTTP 403 when admin mode is not active
- Resource not found: HTTP 404 with resource type and identifier context
- Method not allowed: HTTP 405 for unsupported HTTP methods on valid endpoints
- Conflict errors: HTTP 409 for state conflicts and duplicate resources
- Internal server errors: HTTP 500 with sanitized error details for production safety

**Error Context and Details**
- Field-level validation error details for client-side form feedback
- Resource identifiers for not found and conflict errors
- Stack trace inclusion in development environments for debugging
- Stack trace sanitization in production to prevent information leakage
- Request ID correlation for support requests

**Production Safety**
- Sensitive information filtering from error messages and stack traces
- Generic error messages for internal server errors in production
- Detailed error information exposure only in development mode
- Secure logging of full error context for operational debugging

**Integration Points**
- Centralized exception handling for all FastAPI routes and dependencies
- Custom exception classes with automatic status code mapping
- Validation error transformation from Pydantic models
- Third-party library exception wrapping for consistent error presentation

**Operational Monitoring**
- Request ID generation for each request

The global error handling middleware ensures that all API errors, whether from validation, business logic, or infrastructure failures, are presented to clients in a consistent, informative, and production-safe format.

##### REQ-5.2.3.5 Request/response logging middleware

The request/response logging middleware provides comprehensive audit trail capabilities by capturing all HTTP traffic flowing through the API. It enables operational visibility, security monitoring, performance analysis, and compliance tracking by logging request details, response characteristics, and execution metrics.

**Logging Architecture**
- Middleware layer capturing all incoming requests and outgoing responses
- Cross-origin handling wraps request logging, which wraps error handling
- One identifier per request, made once and shared by request logging and error handling
- Integration with application logging system for unified log management

**Request Logging**
- HTTP method and full request path with query parameters
- Request headers with configurable filtering for sensitive data
- Client information including IP address and user agent
- Request body logging for non-GET requests with size limits
- Request timestamp with high-precision timing for performance analysis
- Correlation ID assignment for request lifecycle tracking

**Response Logging**
- HTTP status code and response reason phrase
- Response headers with sensitive data filtering
- Response body logging with configurable size limits
- Response timestamp and total request duration
- Error details for non-2xx responses with stack trace correlation
- Performance metrics including processing time breakdown

**Sensitive Data Handling**
- Automatic filtering of authentication headers and tokens
- Password and credential masking in request/response bodies

**Performance Considerations**
- Request/response body size limits to prevent memory exhaustion

**Filtering and Configuration**
- One log level per WAVE, set in its config file and changed in Settings (REQ-4.7.18)

**Correlation and Tracing**
- One identifier per request, made once and shared by request logging and error handling
- Error correlation across multiple log entries

The request/response logging middleware provides the foundation for operational visibility, enabling teams to investigate issues while protecting sensitive data and maintaining system performance.

#### 4.2.3 API Documentation

##### REQ-5.2.3.6 OpenAPI/Swagger documentation auto-generation

The OpenAPI documentation system provides machine-readable API specifications and interactive documentation that enables client developers, API consumers, and testing tools to understand and interact with the WAVE REST API. It leverages FastAPI's built-in OpenAPI schema generation capabilities to automatically produce comprehensive, accurate, and always up-to-date API documentation.

**Documentation Architecture**
- Automatic OpenAPI 3.0+ schema generation from FastAPI route definitions
- Interactive Swagger UI for human-readable API exploration and testing
- Alternative ReDoc interface for clean, print-friendly API reference
- JSON/YAML schema export endpoints for programmatic access and code generation
- Schema validation ensuring documentation accuracy matches implementation

**API Specification Coverage**
- Complete endpoint catalog with HTTP methods, paths, and operation identifiers
- Request parameter documentation including path, query, header, and cookie parameters
- Request body schemas with validation rules and example payloads
- Response schemas for all status codes with success and error examples
- Type definitions for all data models with field descriptions and constraints

**Endpoint Metadata**
- Endpoint grouping by functional tags for logical organization
- Human-readable summaries and detailed descriptions for each operation
- Operation identifiers for stable client code generation
- External documentation links for complex operations requiring additional context

**Type System Integration**
- Pydantic model integration for automatic schema extraction from Python types
- Field-level validation rules including required fields, formats, and constraints
- Enum definitions for constrained value sets
- Nested object structures with recursive schema generation
- Array and dictionary types with item schema specifications
- Union types for polymorphic request/response handling

**Request Documentation**
- Parameter location specification (path, query, header, cookie)
- Required vs optional parameter distinction
- Default values for optional parameters
- Parameter validation constraints (min/max, pattern, enum)
- Example values demonstrating proper parameter usage
- Content type negotiation for request bodies

**Response Documentation**
- Status code semantics with clear explanations of success, client error, and server error conditions
- Response body schemas for each status code
- Content type specifications for response formats
- Header documentation for metadata and pagination
- Example responses for common scenarios and error cases
- Response model reuse across similar endpoints

**Interactive Documentation Features**
- Swagger UI at dedicated path for browser-based API exploration
- "Try it out" functionality allowing live API calls from documentation interface
- Request/response examples with syntax highlighting
- Response visualization with formatted JSON and status code highlighting

**Schema Export and Integration**
- JSON schema endpoint exposing complete OpenAPI specification
- YAML schema endpoint for human-readable specification format
- Schema versioning aligned with API version for compatibility tracking
- Client code generation support through standardized OpenAPI format
- Integration with API testing tools and mock servers
- Documentation download capability for offline reference

**Documentation Quality**
- Description completeness requirements for all endpoints and models
- Example value requirements demonstrating realistic API usage
- Tag consistency enforcing logical endpoint grouping
- Schema validation preventing documentation drift from implementation
- Automated tests verifying documentation accuracy against live endpoints

**Development and Maintenance**
- Automatic documentation updates when route definitions change
- No manual schema maintenance reducing documentation drift risk
- Inline documentation through Python docstrings and field descriptions
- Type hint enforcement ensuring accurate schema generation
- Validation during development preventing incomplete documentation deployment

**Documentation Access Patterns**
- Swagger UI served at standard path for interactive exploration
- ReDoc interface at alternative path for clean reading experience
- OpenAPI JSON available programmatically for tooling integration
- OpenAPI YAML for configuration management and version control

**Production Considerations**
- Performance optimization through schema caching
- Example data sanitization preventing sensitive information exposure
- External documentation embedding for operational guides

**Compliance and Standards**
- OpenAPI 3.0+ specification compliance for tool compatibility
- JSON Schema Draft 7+ adherence for validation consistency
- HTTP status code semantics following RFC 7231 standards
- Content type handling per RFC 6838 media type specifications

The OpenAPI documentation system ensures that WAVE's REST API is self-documenting, discoverable, and easy to integrate with through comprehensive, accurate, and always-current machine-readable specifications and human-friendly interactive interfaces.
