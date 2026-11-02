### 15.3 WebSocket Client

#### 15.3.1 WebSocket Client Architecture

##### REQ-5.2.1.4 WebSocket client for real-time updates

The WebSocket client provides bidirectional, real-time communication between the frontend and backend, enabling instant delivery of execution events, status updates, and system notifications without HTTP polling overhead. This infrastructure supports live flow visualization, real-time dashboards, and interactive debugging where immediate feedback is essential.

**Service Layer Position**: The WebSocket client operates alongside the REST client within the service layer, sharing backend connectivity configuration while maintaining separate connection state. Components subscribe to event streams through state management rather than WebSocket primitives directly, enabling the transport to evolve independently.

**Connection Lifecycle**: The client manages connection establishment, graceful disconnection, and automatic reconnection after failures. Connection attempts include client type identification for backend tracking. Session identifiers persist across reconnection attempts, allowing the backend to restore subscriptions and deliver queued messages accumulated during disconnection. Connection state propagates to the UI for appropriate feedback during connectivity transitions.

**Lazy Connection Strategy**: The WebSocket store implements lazy connection - connecting only when needed (e.g., when entering the Workbench page) and disconnecting when leaving. This approach balances server resource usage with responsive real-time updates. The store maintains a single client instance, preventing multiple concurrent connections that could occur during React Strict Mode double-renders or rapid navigation.

**Singleton Client Pattern**: The Zustand store ensures only one WebSocket client instance exists. Calling `connect()` when already connected or connecting returns early. Calling `connect()` when disconnected reuses the existing client if available, reconnecting rather than creating a new instance. This prevents connection proliferation and ensures consistent subscription state.

**Reconnection Strategy**: Automatic reconnection employs exponential backoff with optional jitter to prevent thundering herd scenarios. Attempts continue until successful connection or explicit disconnection request. Session-based reconnection preserves subscription state, eliminating re-subscription after brief disconnections.

**Subscription Management**: Clients subscribe to topics organized into several categories: execution events scoped to specific identifiers, system events for status and configuration changes, debug events for breakpoints and stepping, variable events for value monitoring, log events for real-time console output, and error events for failure notifications. Topic subscriptions support hierarchical matching where subscribing to a parent topic (e.g., "execution") receives all child topic events (e.g., "execution:abc123"). Active subscriptions persist locally for restoration after reconnection. Subscription state synchronizes with the backend through explicit subscribe and unsubscribe messages.

**Topic Categories**:

| Category | Topic Pattern | Purpose |
|----------|---------------|---------|
| execution | `execution:{id}` | Execution lifecycle events (started, updated, completed) |
| system | `system:*` | Status and configuration changes |
| debug | `debug:{execution_id}` | Breakpoint hits, stepping events |
| variable | `variable:{execution_id}` | Variable value changes |
| log | `log:{session_id}` | Real-time log entries for console panel |
| error | `error:*` | Failure notifications |

**Event Dispatch**: Incoming events route to registered handlers based on event type, supporting multiple handlers per type for scenarios where different components respond to the same events. Handler registration returns cleanup functions following established subscription patterns. Event payloads transform into typed domain objects before dispatch.

**Message Protocol**: The client implements the JSON-based protocol handling standardized envelopes with event type, timestamp, sequence number, and payload. Message sequence numbers enable gap detection for missed events. For critical messages requiring delivery confirmation, the client sends acknowledgment responses when messages include an acknowledgment identifier.

**Heartbeat Protocol**: The client responds to server pings with pongs, maintaining connection liveness. Missed heartbeats trigger proactive reconnection before server-side timeout. The mechanism also provides latency measurements for connection quality indicators.

**Error Handling**: Connection errors classify into categories determining recovery actions: network issues trigger backoff reconnection, server rejection surfaces as specific error types, protocol errors log without terminating the connection, and subscription errors propagate to the subscribing component. The classification aligns with REST client patterns for consistency.

**State Integration**: WebSocket events integrate with state management, updating stores with real-time data that triggers reactive UI updates. The integration ensures components receive updates through the same reactive mechanisms used for REST-fetched data, maintaining consistency in data flow patterns.

**Type Safety**: TypeScript interfaces define all event types, subscription parameters, and connection state structures. Event handlers receive strongly-typed payloads matching backend definitions. Discriminated unions enable exhaustive handling of event categories.
