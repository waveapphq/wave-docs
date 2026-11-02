### 15.2 API Client Layer

#### 15.2.1 REST Client Architecture

##### REQ-5.2.1.3 REST API client layer with error handling

The REST API client layer provides a unified abstraction for all HTTP communication between the frontend application and the backend API server, isolating UI components from network complexities while ensuring consistent behavior across all backend interactions.

**Service Layer Position**: The API client resides within the service layer, positioned between state management stores and the network boundary. Components interact through domain-oriented interfaces that express operations in business terms rather than HTTP mechanics, enabling the API implementation to evolve independently without cascading changes throughout the codebase.

**Request Pipeline**: The client maintains centralized configuration for base URL, default headers, timeout thresholds, and retry policies. Individual requests can override these defaults for specific needs such as extended timeouts for long-running operations. The response pipeline validates status codes, parses bodies according to content type, and transforms data into TypeScript domain models.

**Response Envelope Handling**: The backend returns standardized response envelopes containing success flag, data payload, response metadata (timestamp, API version, request ID), and pagination information for list operations. The client unwraps these envelopes, extracting the data for consumers while preserving metadata for debugging, tracing, and pagination state management. Request IDs propagate through the client for correlation in logging and error reporting.

**Pagination Support**: The client handles both page-based and cursor-based pagination patterns exposed by the backend. For page-based pagination, the client manages page number and page size parameters, extracting total counts and page metadata from responses. For cursor-based pagination used in high-modification endpoints, the client tracks cursor tokens for subsequent page requests. Pagination state integrates with list views to enable infinite scroll, load-more patterns, and traditional page navigation.

**ETag Support**: For resources supporting optimistic concurrency, the client tracks ETag values from response headers and includes If-Match headers on update requests. When the backend returns 412 Precondition Failed indicating a concurrent modification conflict, the client surfaces this as a specific error type enabling the UI to prompt users to refresh and retry.

**Error Classification**: Failures are classified into distinct categories with different handling approaches. Network errors from connectivity issues or timeouts indicate environmental problems that may resolve on retry. Client errors (4xx) indicate request problems requiring user intervention, with 409 Conflict and 412 Precondition Failed requiring specific handling for concurrent modification scenarios. Server errors (5xx) indicate backend issues warranting retry with exponential backoff. The classification determines retry behavior, user messaging, and recovery options.

**Structured Error Handling**: Backend error responses are transformed into a consistent internal format preserving error codes, messages, and validation details. The standardized error envelope from the backend includes field-level validation errors for form feedback, resource identifiers for not-found conditions, and request IDs for support correlation. Error objects carry sufficient context for both programmatic handling and user-facing message generation.

**Failures Are Never Defaults**: A request that fails is reported to its caller as a failure. It is never substituted with an empty collection, a zero, or any other value the caller could read as an answer from the server. Callers routinely draw conclusions from what comes back - that a list is empty, that a count is nil - and a substituted default turns a transport problem into a false statement about the user's data. No request is exempt: how small the result is, or how decorative the thing it feeds, decides nothing, because the surface presents it as fact either way. Deciding what a failure means to the user belongs to the surface that requested it, which is the only layer that knows whether the page can continue without the data.

**Resilience Patterns**: Configurable retry logic handles transient failures with exponential backoff, distinguishing between idempotent operations (GET, PUT, DELETE) that can safely retry and non-idempotent mutations (POST) where retry could cause duplicates. Circuit breaker patterns prevent continued requests to failing endpoints, preserving responsiveness during outages.

**Request Interceptors**: Lifecycle hooks enable cross-cutting concerns including authentication credential attachment, request/response logging with request ID propagation for debugging, and centralized error handling for common patterns like auth failures or server errors triggering toast notifications.

**State Integration**: The client coordinates with state management for data fetching and updates. Loading and error states propagate through the state layer for appropriate UI feedback. Pagination state synchronizes with stores to maintain scroll position and loaded data across navigation.

**Type Safety**: TypeScript types for request parameters, response bodies, pagination info, and error structures provide compile-time contract verification, catching mismatches during development and serving as executable API documentation.
