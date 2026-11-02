## 10. Testing Strategy

### 10.3 Real-Service E2E Architecture

Functional end-to-end tests that exercise an external protocol run against real backing
services, never against in-process protocol fakes. A fake proves plumbing at best; only a
real server proves protocol compatibility, real execution semantics, and failure behavior.

**Service stack**: real services run as a compose-orchestrated container stack shared by
the whole E2E run. Each service is defined once (image, provisioning, configuration
variants) in a service catalog under the E2E tree. Services come in two visibility
classes:

- **Published services** listen on host loopback ports and are the direct targets of
  workflow fixtures (for example the SSH endpoint a Session Open connects to).
- **Internal services** are reachable only on the stack's private network. They exist to
  prove genuine reachability boundaries: a nested host that can only be reached through a
  jump hop, and a traffic target that can only be reached through a tunnel or from a
  remote execution context.

**Configuration-variant services** express server-side test dimensions (for example an
SSH endpoint without an SFTP subsystem, or one that forbids TCP forwarding) as separate
service instances on distinct ports, so tests select a server behavior by endpoint rather
than by mutating shared server state.

**Fault-injection endpoints** speak the real protocol on the wire but answer, on request,
with the failures a healthy server will not produce: an unavailable status, a gateway page
where an API answer belongs, a stall long enough to expire a deadline, or an add-on API
that is not there. They stand beside the real service, never in place of it: the happy
path is always proved against the real server, and only the failure path is asked of the
injector, so a client's error classification is tested without pretending a fake proves
compatibility. The fault wanted is selected by the endpoint the fixture connects to, the
same way a configuration variant is, so nothing is mutated and parallel shards ask for
different faults at the same time. The Kubernetes endpoint is the worked example: it
answers API discovery successfully and fails only the resource read, a split no real
cluster can be talked into, which is the only way to reach the read-error paths at all.

**Lifecycle**: the host side owns the stack lifecycle. The CI entry point brings the full
stack up before test shards launch and guarantees teardown on every exit path. Local scripted
runs also bring up the full service stack; bring-up is idempotent, so an
already-running stack is detected and reused. Test shards themselves never manage
services - they detect readiness through health gates and consume endpoints.

**Isolation model**: test shards are fully isolated containers; the service stack is the
one deliberately shared surface. Shared services are therefore either read-mostly by
design, or consumed through per-test unique state (unique remote paths, per-run
credentials, connection-scoped sessions) so parallel shards cannot interfere. Test
determinism requirements (fixed host keys, fixed credentials, pinned hostnames) are baked
into the service images, and the deterministic key material lives in the repository like
other test certificates. File-addressed blocks prove their remote path against the SSH host
of the stack under a per-test scratch directory the test itself removes.

**Endpoint resolution**: tests and fixtures resolve service endpoints through shared
configuration helpers that account for execution mode - direct loopback when the suite
runs on the host, the container-to-host gateway when the suite runs inside CI shard
containers. Fixtures carry the host-mode default; the import path injects the
mode-correct endpoint.

**Coverage discipline**: when a feature that talks to an external system lands, its
functional tests target the corresponding stack service; if the service does not exist
yet, adding it to the catalog is part of that feature's test scope. Simulated servers are
not added for protocols the stack can serve for real; a fault-injection endpoint is not
one of them, because it exists for the failures the real service cannot be made to produce
and never answers for it on the happy path.

---