### 4.3 System Health and Dashboard Endpoints

#### 4.3.1 System Health Endpoints

##### REQ-5.2.4.11 System health and status endpoints

System health and status endpoints provide operational visibility into the WAVE backend service state, enabling monitoring systems and operational teams to assess health, detect degradation, and make deployment decisions based on real-time system state.

**Health Check Architecture**: The system exposes three endpoint levels for different operational needs. Liveness checks confirm basic service availability with minimal overhead, returning HTTP 200 for responsive processes without dependency validation to avoid false negatives during transient failures. Readiness checks validate full operational capability including storage accessibility, configuration validity, and resource availability, returning HTTP 200 when ready for traffic or HTTP 503 when dependencies fail or resources are insufficient. Detailed status endpoints expose component-level health with uptime, CPU, memory and disk for operational diagnostics.

**Health State Classification**: Components report health as healthy when fully operational, degraded when functional but experiencing issues or performance reduction, unhealthy when critical failures prevent normal operation, or unknown when assessment cannot complete. The overall service health aggregates component states with conservative evaluation where any unhealthy component renders the service unhealthy, and any degraded component without unhealthy components renders it degraded.

**Component Health Assessment**: Health checks evaluate storage accessibility for artifact directories and file system operations, configuration validity ensuring required parameters are present and values are valid, resource availability with disk space and memory thresholds, and application state confirming initialization completed without critical errors. Assessment runs asynchronously with non-blocking evaluation to prevent request queue buildup and configurable timeouts preventing hung checks from affecting service responsiveness.

**Performance and Caching**: Liveness checks target sub-10ms response optimized for high-frequency polling. Readiness checks target sub-100ms response balancing comprehensive validation with responsiveness. Detailed status allows up to 500ms for thorough assessment. Health check results are cached with configurable TTL to prevent excessive re-evaluation while maintaining freshness for operational decisions. Cache invalidation occurs on configuration changes or detected state transitions.

**Lifecycle Integration**: Services start in unhealthy state until initialization completes including configuration load, resource validation, and storage preparation. State transitions to ready when all components report healthy. Graceful shutdown transitions to not ready before stopping request processing. Health endpoints remain available throughout the lifecycle for monitoring continuity. Version information updates on deployment for tracking and rollback verification.

**Monitoring and Observability**: Structured health responses enable parsing by monitoring systems with standardized status indicators and component breakdowns. Alert threshold guidance helps operations teams configure degraded and unhealthy state notifications.

#### 4.3.2 Status Dashboard Endpoint

##### REQ-5.2.1.43 Status Dashboard screen (system overview with entity counts, recent activity feed, running executions list)

The status dashboard endpoint provides an aggregated view of system state for the main navigation dashboard. It consolidates multiple data sources into a single response, eliminating multiple client requests and ensuring consistent point-in-time snapshots.

**Aggregation Architecture**: The endpoint aggregates four data sources: entity counts (tests, suites, regressions) retrieved without loading full entity data; execution statistics within a configurable time window (default 24 hours) with running, passed, and failed breakdowns; recent activity listing the most recently modified tests, suites and regressions; and running executions with progress percentages and current block information.

**Response Structure**: The response contains three main sections plus metadata. The metrics section provides entity counts as objects containing count properties and execution statistics including the time window in days, total count, and status breakdowns for running, passed, and failed executions. The recentActivity section lists recently modified entities with identifier, entity type, display name, status context string (formatted as "Type · ParentName" for frontend composition with relative timestamps), and ISO 8601 timestamp. The runningExecutions section lists active executions with identifier, environment name, entity name, entity type, progress percentage, start timestamp, and status. Response metadata includes generation timestamp and errors array for partial failure reporting. Empty state detection is derived client-side from zero entity counts.

**Performance and Caching**: The endpoint targets sub-200ms response using optimized count queries and indexed execution lookups. Results cache with short TTL (default 5 seconds) balancing freshness with performance. Cache invalidation occurs on entity or execution state changes.

**Real-time Integration**: The dashboard page polls this endpoint every 5 seconds.

**Error Handling**: Complete storage unavailability results in HTTP 503. Partial failures (e.g., one entity type unreadable) return HTTP 200 with available data; failed sections return null instead of empty values, and an errors array in response metadata describes what failed. This allows the frontend to distinguish "zero items" (empty array) from "failed to fetch" (null) and display appropriate messaging.
