## 9. Monitoring and Observability

### 9.2 Unified Backend Logging

##### REQ-3.7.2.5 Custom logging from tests/blocks
##### REQ-3.7.2.6 Log levels (DEBUG, INFO, WARN, ERROR, FATAL)
##### REQ-5.2.3.5 Request/response logging middleware for audit trail

**Two Log Streams**: Backend produces two distinct log streams for different audiences. The test console stream delivers execution logs to the frontend via WebSocket - format is `HH:MM:SS.mmm LEVEL [SOURCE] message` where SOURCE names what wrote the line: `SYSTEM` for the engine's own lines, `FUNCTION` for the lines a Function writes about itself, and for a block the block type's name as the palette shows it, alone when the block's Name matches it (`SQL`) and followed by the Name when it does not (`SQL : Run PSQL Proc`). A block's Description never appears in SOURCE. The engine's own lines name a block by its Name (`Block 'PSQL' failed: ...`), never by its Description. The backend terminal stream shows server operations to developers via stdout - format is `HH:MM:SS.mmm LEVEL message` with self-describing messages (no artificial source category). Both use 24-hour timestamps with milliseconds and right-padded 5-character log levels.

**Test Console Format**: `[SYSTEM]` for execution lifecycle messages (start, complete, errors). `[Type]` for a block whose Name matches its type's name, where Type is the name the palette shows for the block type. `[Type : Name]` for a block whose Name differs from its type's name. This format enables filtering by source in the UI.

**Backend Terminal Format**: Messages are self-describing without forced source brackets. Every HTTP request, failed ones included, shows `[1a2b3c4d] GET /path → 200 (1.23ms)`. The bracketed value is the start of the request's identifier and prefixes every line logged while the request is handled. Execution events show `Execution[abc12] Test 'name' started (3 blocks)`. Server lifecycle shows `Server started on http://...`. ASGI access logs are suppressed - all HTTP logging flows through middleware only.

**Lifecycle Events**: Block execution emits start and completion logs with duration. Plugin execution logs initialization, outcome, and cleanup. Parameters and return values are summarized with automatic secret masking. Errors include stack traces and relevant variable context.

**Log Level Control**: Log verbosity is a setting of the config file, overridden at startup by an environment variable, and changed at runtime from Settings > General, where Save applies the new level at once and writes it back to the config file (REQ-4.7.18). Supported levels are DEBUG, INFO, WARN, ERROR, FATAL. Changes apply to all backend loggers including HTTP middleware and uvicorn. WebSocket lifecycle messages are DEBUG level to reduce noise at INFO.

**Report Content**: Both persisted streams - framework entries and custom test entries - merge chronologically into generated report content (see section 4.14.2 Report Content).

### 9.3 Metrics Collection

**Execution Metrics**
- Test execution time
- Block execution duration
- Success/failure rates

### 9.4 Log Streaming Integration

**Real-time Broadcasting**: Backend logs flow through the WebSocket-based streaming service for real-time frontend delivery. The unified format applies to both console output and streamed entries. Context fields enable frontend filtering by block, plugin, or log level. The existing two-layer filtering architecture continues to apply - plugin-level thresholds filter at source, execution-level thresholds filter during streaming.

---