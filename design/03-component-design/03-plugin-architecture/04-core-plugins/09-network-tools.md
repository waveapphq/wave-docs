##### REQ-3.4.1.9 Network tools (ping, nc)

The Ping block sends ICMP echo requests to one host and reports what came back, the way `ping` does.

**Purpose and Locus**: Ping works like `ping` (PING-1). Every setting maps to a ping flag, is named for it and takes ping's default; the only settings WAVE adds are Pass when and Log Output. The block takes one Host, as `ping` takes one destination: there is no host list, no host range and no sweep, because a sweep is `fping`, a different tool (PING-2). It runs on the WAVE server, and on a remote host when placed inside a Remote Execution body, where it is remotable and carries the R badge. It has no SSH tunnel support, because a port-forward carries TCP and ICMP is not TCP. One core, written against the Python standard library only, runs the block on the WAVE server and travels inside the remote bundle, so the block behaves the same on both (PING-14). No third-party ICMP library is used, because the block must run from the pushed bundle on a host where WAVE installs nothing. The block's documentation page carries one version, 1.0.0.

**Privilege**: The block never needs root on any machine (PING-4). It sends its echo requests over the operating system's unprivileged ICMP datagram path. A raw socket is used only when the process already holds the capability for one, and is never requested: there is no sudo, no setuid, and no silent fallback to a TCP connect, which would report reachability without an ICMP packet ever leaving. A machine that refuses the unprivileged path fails the block at once, before any request is sent (PING-5). The message names the machine the block ran on, the WAVE server or the execution host by name, and the one setting to change on that machine. Flood mode is not offered, because `ping -f` is the one ping capability that genuinely needs privilege (PING-13).

**Rows**: The top rows stand ungrouped in the order Host, Count, Interval (seconds), Packet size (bytes), Timeout, Pass when and Log Output (PING-6). Host names the one host to reach, a name or an IP address, and is required. Count, ping's `-c`, is 4 by default and runs from 1 to 1000 (PING-12). Interval (seconds), ping's `-i`, is 1 by default and must be at least 0.2. Packet size (bytes), ping's `-s`, is 56 by default, ping's own default, and runs from 0 to 65507, the most an ICMP echo payload can carry; the packet on the wire is 28 bytes larger over IPv4. Timeout is 30 seconds by default and must be positive. Pass when decides what counts as success (PING-3): Any reply, the default and ping's own exit rule, passes when at least one echo is answered; All replies passes when every request sent was answered; At least N replies reveals Minimum replies, an integer, 1 by default, from 1 to Count; and At least X% replies reveals Minimum rate (%), a number, 100 by default, from 0 to 100. What counts as an answer, and how each choice resolves against the counts, is set out under REQ-3.3.4.4 Ping tests for reachability. There is no max-allowed-loss setting: the Condition block already does thresholds, and ping has no such flag. There is no run-forever mode either, because the Wait Until control already repeats a block until a condition holds (PING-13). Log Output, off by default, writes each reply to the run console as it lands.

**Advanced**: The Advanced group is collapsed and holds four settings (PING-7). IP version selects Auto, the default, IPv4 or IPv6, ping's `-4` and `-6`, and forces the family the Host resolves to. TTL, ping's `-t`, is empty by default and runs from 1 to 255; it sets how many hops a request may take. Don't fragment, ping's `-M do`, is off by default and makes the block the standard MTU probe: a packet that does not fit the path is not broken up, so the path reports the size that does fit. Source address, ping's `-I`, is empty by default and chooses which local address the echo requests leave from. It takes an address and never an interface name (PING-8), because binding to an interface needs a capability the block refuses to require, while binding to an address does not.

**Timeout**: Timeout is the real bound on the whole block, whatever Count and Interval would otherwise take, and it covers resolving the Host as well (PING-6). When it runs out the block ends there, and what was measured up to that point is still reported: Summary and Replies carry the requests that were sent and the replies that came back.

**Outputs and Failure Handling**: Success, a Boolean, is always an output and reads true when the pass rule held (PING-9). Summary, a Table, is always an output and holds exactly one row, with the columns `sent`, `received`, `loss_percent`, `rtt_min_ms`, `rtt_avg_ms`, `rtt_max_ms` and `rtt_mdev_ms`. It is a Table rather than a Map because the summary is mostly read in the run log, where a table prints the way ping's own summary lines do (PING-10). The round-trip mdev column is kept because it is the only measure of how steady the latency was, and the block's documentation page explains in plain words that it is the standard deviation of the round-trip times (PING-11). Replies, a Table, is always an output and holds one row per request, with the columns `seq`, `source`, `bytes`, `ttl`, `time_ms` and `status`. The status column carries the ICMP detail (PING-15): answered, timed out, the ICMP error with the address that sent it when a router replied instead of the target, or the discovered path MTU when Don't fragment blocked the packet; an ICMP error is loss, as set out under REQ-3.3.4.4 Ping tests for reachability. Error and Reason are outputs only while On error is Record and continue, the shared output core. The Failure Handling group's On error setting decides what a run-time failure does: Fail the block, the default, makes the failure a block error. Validation refuses a misconfigured block on WAVE before it runs, wherever it runs and whatever On error says:

| Failure | Message |
|---------|---------|
| Host is missing or blank | `Missing required parameter: host` |
| Count is not a whole number | `Parameter 'count' must be an integer` |
| Count is outside 1 to 1000 | `Parameter 'count' must be between 1 and 1000` |
| Interval is not a number | `Parameter 'interval' must be a number` |
| Interval is below 0.2 | `Parameter 'interval' must be at least 0.2` |
| Packet size is not a whole number | `Parameter 'packet_size' must be an integer` |
| Packet size is outside 0 to 65507 | `Parameter 'packet_size' must be between 0 and 65507` |
| Timeout is not a whole number | `Parameter 'timeout' must be an integer` |
| Timeout is zero or negative | `Parameter 'timeout' must be positive` |
| Pass when is At least N replies and Minimum replies is missing or blank | `Missing required parameter: min_replies` |
| Minimum replies is not a whole number | `Parameter 'min_replies' must be an integer` |
| Minimum replies is below 1 or above Count | `Parameter 'min_replies' must be between 1 and {count}` |
| Pass when is At least X% replies and Minimum rate (%) is missing or blank | `Missing required parameter: min_rate` |
| Minimum rate (%) is not a number | `Parameter 'min_rate' must be a number` |
| Minimum rate (%) is outside 0 to 100 | `Parameter 'min_rate' must be between 0 and 100` |
| TTL is set and is not a whole number | `Parameter 'ttl' must be an integer` |
| TTL is set and is outside 1 to 255 | `Parameter 'ttl' must be between 1 and 255` |
| Source address is set and is not an IP address | `Parameter 'source_address' must be an IP address` |
| Don't fragment or Log Output is not true or false | `Parameter '{name}' must be a boolean` |
| Pass when, IP version or On error holds a value outside its choices | `Parameter '{name}' must be one of: {values}; Actual value was '{value}'.` |

Only the settings in effect are validated: Minimum replies is checked only while Pass when is At least N replies, and Minimum rate (%) only while Pass when is At least X% replies. Host and Source address are read with the spaces around them removed, so a value of spaces only counts as blank.

A failure while the block runs follows On error and carries the reason shown:

| Failure | Message | Reason |
|---------|---------|--------|
| The machine refuses an unprivileged ICMP socket | `{The WAVE server / The execution host '{host}'} may not send ICMP echo requests: the operating system refused an unprivileged ICMP socket. Add this machine's group to net.ipv4.ping_group_range.` | `Unprivileged` |
| The Host does not resolve, or does not resolve within Timeout | `Cannot resolve host '{host}'` | `Unresolved` |
| IP version is IPv4 or IPv6 and the Host has no address of that family | `Host '{host}' has no {IPv4/IPv6} address` | `Unresolved` |
| Source address cannot be used to send | `Cannot send from source address '{address}': {operating system's own text}` | `Source` |
| Don't fragment is on and the packet does not fit the path | `A {n}-byte packet does not fit the path to {host}: the path is {mtu} bytes` | `TooLarge` |
| Timeout runs out before Count requests have been sent and answered | `Ping did not finish within {n} seconds` (singular "second" when {n} is 1) | `Timeout` |
| Pass when is Any reply and nothing answered | `No reply from {host}` | `NoReply` |
| Pass when is All replies and at least one request went unanswered | `{received} of {sent} requests answered` | `NotAllReplied` |
| Pass when is At least N replies and fewer answered | `{received} of {sent} requests answered, fewer than the {min_replies} required` | `TooFewReplies` |
| Pass when is At least X% replies and the rate was lower | `{rate}% of requests answered, below the {min_rate}% required` | `RateTooLow` |

Under Record and continue the block finishes: Success reads false, Error carries the message, Reason carries the reason above, and Summary and Replies still carry what was measured.

**Console Output**: The summary reaches the run log at the end of any run that sent a request, the way ping prints its own summary lines (PING-16); a block that fails before the first request leaves - the machine refusing the socket, the Host not resolving, the Source address not usable - reports that failure alone, as ping does. Log Output adds one line per reply as it lands, in ping's own per-reply shape, and a warning line for a request that was not answered.

**Remote Execution**: Inside a Remote Execution body Ping runs on the remote host and carries the R badge (PING-1), with the same core as on the WAVE server, run from the pushed bundle (PING-14), so the echo requests leave the remote host. Validation still runs on WAVE before the request leaves. The privilege check runs on the execution host and names that host when it refuses (PING-5). A remote block is stopped the way every other remote block is.

**Stopping**: Stop ends the block at once, whatever it is doing: resolving the Host, waiting between requests, or waiting for a reply.

The Netcat block checks whether ports on a host accept a connection, or connects to a port, sends a payload and captures what the port answers, the way `nc` does.

**Purpose and Locus**: Netcat works like `nc` (NETCAT-1). Every setting maps to an nc flag or argument, is named for it and takes nc's default; the only settings WAVE adds are the Ports switch and Pass when. The block does two jobs: it checks ports, as `nc -vz` does, and it talks to a port, connecting, sending a payload and capturing the reply. It has no listen mode and no TLS. It runs on the WAVE server, on a remote host when placed inside a Remote Execution body, where it is remotable and carries the R badge, and through an SSH tunnel. One core, written against the Python standard library only, runs the block on the WAVE server and travels inside the remote bundle, so the block behaves the same on both (NETCAT-11). The ports of a Range are dialed concurrently, at most 64 at a time. The block's documentation page carries one version, 1.0.0, and explains every port state in plain words (NETCAT-8).

**Ports and Pass Rule**: Host names the host to reach and is required. Ports selects Single, the default, or Range (NETCAT-2). Single shows Port, an integer from 1 to 65535, required. Range shows Port range, a required text that holds nc's range form and a list together, for example `22, 80, 8000-8010`. Port range is read as text and split at its commas, and each entry has the spaces around it removed. An entry is a port number written in the digits 0 to 9, or two such numbers joined by a hyphen with nothing between them; leading zeros are allowed. Anything else is an invalid entry, including an empty one left by two commas in a row or by a trailing comma. Each entry is checked for its shape first, then for both ends lying between 1 and 65535, then for running forward, so an entry that is both out of bounds and backward is reported as out of bounds. A range expands in ascending order, the entries keep the order they are written in, and a port named more than once is checked once, at its first position; the rows of Results follow that order (NETCAT-11). Protocol selects TCP, the default, or UDP. Pass when is shown only for Range (NETCAT-3): One succeeds, the default and nc's own rule, passes when any port succeeds; All succeed passes only when every port succeeds. Single passes when its one port succeeds. Which states count as success is set out under REQ-3.3.4.3 Port connectivity tests.

**Check and Talk**: Mode selects Check, the default, or Talk (NETCAT-4). Check is nc's `-z`: over TCP it connects and sends nothing; over UDP, where there is no connection to make, it does what `nc -u -z` does, sending one datagram that holds the single byte `X` and then waiting up to Timeout for a reply or a refusal (NETCAT-13). Talk connects, sends a payload and captures the reply; over TCP it reads the reply while it sends, as nc does. Send, shown only in Talk, selects Nothing, Text, the default, or File. Nothing sends no payload and reads what a server sends on connect, such as a banner. Over UDP, Send offers Text and File only, because a UDP port never answers a datagram that was not sent. Range supports Talk: each port gets a connection of its own, the same payload goes to each, and each port's reply lands in its own Results row. The Talk options sit in an Advanced group, collapsed, that exists only in Talk mode (NETCAT-5). CRLF line endings, nc's `-C`, off by default, sends every line ending in the payload as carriage return plus line feed, whatever the payload's source. Close after sending, nc's `-N`, off by default, shuts down the sending side once the payload is sent, so a server that waits for the end of its input answers at once. It applies to TCP only and is hidden when Protocol is UDP; with Send Nothing it shuts the sending side right after connecting (NETCAT-16). Max reply size (bytes), an integer, 16777216 by default, must be positive: reading stops once a port's reply reaches that many bytes, the rest is not read, and a WARN line says so.

**Payload**: Text is typed in the Edit value sheet and is sent as UTF-8 (NETCAT-16). Over TCP an empty Text sends nothing. Over UDP the payload travels as one datagram, so an empty Text sends one empty datagram, and a payload larger than 65507 bytes, the most one datagram can carry, fails the block with reason StartFailed (NETCAT-16). File is required when Send is File: a Test Data or Artifacts file picked with Browse, or written as a `testdata://` reference, an `artifact://` reference or a path; its value sheet is pinned to plain text. A payload file resolves the way the Python block's script file does (NETCAT-10, PYRUN-2): a `testdata://` or `artifact://` file is read on WAVE and sent along with the request, so it works inside a Remote Execution body; a plain path is read where the block runs, on the WAVE server or, inside a body, on the remote host. Browse is hidden when the block runs on a remote host, under the shared file-row rule (TDATA-8). A payload file failure follows the rules of the Python block's script file (NETCAT-19): the message names the value as written; a missing file, a missing Test Data or Artifacts target and a path that names something other than a file all read as not found; a file that cannot be read is reported with the error's own text; either fails the block with reason StartFailed. A plain relative path starts at the home folder of the user the WAVE server runs as when the block runs on WAVE, and at the SSH user's home folder on a remote host.

**Timeout**: Timeout is nc's `-w`, in seconds, 10 by default, and must be positive (NETCAT-6). It bounds resolving the Host (NETCAT-17) and each connection attempt: a TCP connect, a UDP wait for a reply or a refusal, and the opening of an SSH tunnel channel. In Talk over TCP it also bounds each wait for the server to take the next part of the payload, and when a part is not taken in time the send stops there. In Talk the reply ends when the server closes its side over TCP, once the server has been silent for Timeout after the send is over, or when it reaches Max reply size (bytes); over UDP the reply datagrams are joined in the order they arrive (NETCAT-5, NETCAT-16).

**Port States**: Each port ends in one of four states, `open`, `closed`, `timed out` and `open or filtered`, and Results carries the state and a reason for every port (NETCAT-8); what each state means is set out under REQ-3.3.4.3 Port connectivity tests. The reason for a refusal is the operating system's text for the error, such as `Connection refused`, never a library's wrapper text, or the SSH server's own words (NETCAT-17); `No answer within {n} seconds` for timed out; `No reply within {n} seconds` for open or filtered (singular "second" in both when {n} is 1); and empty for open. The Host is resolved once per block, within Timeout. Over TCP the resolved addresses are tried in the resolver's order until one connects, and the port reports the last attempt's state and reason; over UDP the first address is used (NETCAT-17). A Host that does not resolve, or does not resolve within Timeout, is not a port state: it fails the block (NETCAT-11, NETCAT-17).

**Outputs and Failure Handling**: Success, a Boolean, and Results, a Table, are always outputs (NETCAT-7). Results holds one row per port, in the Port range order, with the columns `port`, `protocol` (`tcp` or `udp`), `state`, `reason` and `time_ms`, plus `reply` in Talk. `time_ms` counts the milliseconds from the start of the port's connection attempt to its verdict in Check, and to the moment Netcat stops reading the reply in Talk: the server closing its side, the silence Timeout running out, or Max reply size (NETCAT-15). Reply, a String holding the reply as text, is an output only for Single in Talk. A reply becomes text as UTF-8, and any bytes that are not valid UTF-8 appear as the Unicode replacement character, U+FFFD (NETCAT-15). Error and Reason are outputs only while On error is Record and continue, the shared output core. The Failure Handling group's On error setting decides what a run-time failure does. Fail the block, the default, makes the failure a block error. Record and continue lets the block finish: Success reads false, Error carries the message, Reason carries the reason shown below, and a pass-rule failure still fills Results and Reply with what was measured. Only the settings in effect are validated and used: a setting whose row the current Ports, Mode, Send or Protocol hides is ignored, so Send left at Nothing with UDP while Mode is Check is valid (NETCAT-14). Host, Port range and File are read with the spaces around them removed, so a value of spaces only counts as blank. Validation refuses a misconfigured block on WAVE before it runs, wherever it runs and whatever On error says:

| Failure | Message |
|---------|---------|
| Host is missing or blank | `Missing required parameter: host` |
| Ports is Single and Port is missing or blank | `Missing required parameter: port` |
| Port is not an integer or is outside 1 to 65535 | `Parameter 'port' must be between 1 and 65535` |
| Ports is Range and Port range is missing or blank | `Missing required parameter: port_range` |
| A Port range entry is neither a port nor a range, an empty entry included; the message shows the entry with its spaces removed | `Parameter 'port_range' has an invalid entry: '{entry}'` |
| A Port range entry has an end outside 1 to 65535 | `Parameter 'port_range' must be between 1 and 65535` |
| A Port range entry runs backward | `Parameter 'port_range' has a range that runs backward: '{entry}'` |
| Ports, Pass when, Protocol, Mode, Send or On error holds a value outside its choices | `Parameter '{name}' must be one of: {values}; Actual value was '{value}'.` |
| Mode is Talk, Send is Nothing and Protocol is UDP | `Parameter 'send' cannot be 'nothing' when protocol is 'udp'` |
| Mode is Talk, Send is File and no file is named | `Missing required parameter: file` |
| Timeout is not a whole number | `Parameter 'timeout' must be an integer` |
| Timeout is zero or negative | `Parameter 'timeout' must be positive` |
| Max reply size (bytes) is not a whole number | `Parameter 'max_reply_bytes' must be an integer` |
| Max reply size (bytes) is zero or negative | `Parameter 'max_reply_bytes' must be positive` |
| CRLF line endings is not true or false | `Parameter 'crlf' must be a boolean` |
| Close after sending is not true or false | `Parameter 'close_after_send' must be a boolean` |

A failure while the block runs follows On error and carries the reason shown:

| Failure | Message | Reason |
|---------|---------|--------|
| Host does not resolve, or does not resolve within Timeout | `Could not resolve host: {host}` | Unresolved |
| The payload file is not there, its Test Data or Artifacts target is missing, or the path names something that is not a file; the message names the value as written | `Payload file not found: {path}` | StartFailed |
| The payload file cannot be read; the message carries the error's own text | `Could not read the payload file: {os error}` | StartFailed |
| Protocol is UDP and the payload is larger than one datagram can carry | `Payload is too large for one UDP datagram: {n} bytes (at most 65507)` | StartFailed |
| Protocol is UDP and a port is the Local Port of one of this run's open SSH tunnels | `SSH tunnel detected on local port {p}: SSH tunnels carry TCP only, so UDP cannot go through one` | Tunnel |
| Single: the port is closed | `Port {port} on {host} is closed: {reason}` | Closed |
| Single: the port timed out | `Port {port} on {host} timed out after {n} seconds` (singular "second" when {n} is 1) | Timeout |
| Range with One succeeds: no port succeeded | `No port on {host} succeeded ({count} checked)` | NoneSucceeded |
| Range with All succeed: some ports did not succeed | `{k} of {count} ports on {host} did not succeed: {list}`, where the list names each such port as `{port} {state}`, joined by `, `, the first 10 followed by ` (+{m} more)` when there are more | NotAllSucceeded |

**Console Output**: The run console receives these lines; payload and reply bytes are never logged:

| When | Level | Line |
|------|-------|------|
| The block starts, Check, Single | INFO | `Checking {host} port {port} over {TCP or UDP}` |
| The block starts, Check, Range | INFO | `Checking {count} ports on {host} over {TCP or UDP}` |
| The block starts, Talk, Single | INFO | `Talking to {host} port {port} over {TCP or UDP}` |
| The block starts, Talk, Range | INFO | `Talking to {count} ports on {host} over {TCP or UDP}` |
| A tunneled port is found | INFO | `SSH tunnel detected on local port {p}: testing {far_host}:{far_port} from {ssh_host}` |
| Single: the port's result | INFO | `Port {port}/{tcp or udp} {state}`, followed by `: {reason}` when the reason is not empty |
| Range: each port's result | DEBUG | The same line as a Single result |
| Range: the summary | INFO | `{open} open, {closed} closed, {timed_out} timed out, {open_or_filtered} open or filtered of {count}`, with every zero count left out |
| Talk: the traffic of each port | INFO for Single, DEBUG for Range | `Sent {n} bytes, received {m} bytes` |
| A reply reached Max reply size (bytes) | WARN | `Reply from port {port} stopped at {n} bytes` |
| A failure recorded under Record and continue | WARN | `Netcat failed ({reason}): {message}` |

**SSH Tunnels**: On the WAVE server Netcat is aware of the run's SSH tunnels (NETCAT-9). When Host is `127.0.0.1` or `localhost` and a port is the Local Port of one of this run's open SSH tunnels, Netcat does not use the local listener: it opens a channel to the tunnel's far host and port through the tunnel's own SSH connection, so the state is the far port's real state, and in Talk the payload and the reply flow through that channel. The console receives one INFO line per tunneled port naming the far host and port and the SSH host they are reached from. SSH tunnels carry TCP only, so UDP to a tunnel's Local Port fails the block at run time with reason Tunnel. That refusal comes after the start line and before any port is dialed, names the first such port in Port range order, and leaves Results empty (NETCAT-18). The check semantics are set out under REQ-3.3.4.3 Port connectivity tests.

**Remote Execution**: Inside a Remote Execution body Netcat runs on the remote host and carries the R badge (NETCAT-1), with the same core as on the WAVE server (NETCAT-11), so the ports are dialed from the remote host. Validation still runs on WAVE before the request leaves. A `testdata://` or `artifact://` payload is read on WAVE and sent along; a plain path is read on the remote host, a relative one from the SSH user's home folder; Browse is hidden (NETCAT-10). A failure to read a WAVE-side payload file on WAVE follows On error exactly as it does on the WAVE server: Record and continue records it with reason StartFailed and empty outputs (NETCAT-19). No tunnel lookup happens there, because loopback inside a body is the remote host itself (NETCAT-9).

**Stopping**: Stop ends the block at once, whatever it is doing: resolving the host, connecting, sending, reading a reply or opening an SSH tunnel channel (NETCAT-6). A tunnel channel that is still opening is never waited on, and none is left open.

**Sheet Layout**: The top rows stand ungrouped in the order Host, Ports, Port or Port range, Pass when, Protocol, Mode, Send, Text or File, and Timeout (NETCAT-12). Below them come Advanced, collapsed and shown only in Talk, holding CRLF line endings, Close after sending (TCP only, hidden when Protocol is UDP) and Max reply size (bytes); Failure Handling, collapsed, holding the shared On error row; and Secrets, collapsed. Every two- and three-value choice is a select, and every row that is not a select carries the expression toggle. The block carries the Cable glyph and sits in the Network palette category.

##### REQ-3.3.4.3 Port connectivity tests (nc -vz)

A port connectivity test is the Netcat block in Check mode (REQ-3.4.1.9): for each port it reports whether the port accepts a connection from where the block runs, and the block passes or fails on a stated rule.

**What Each State Means**: A check ends every port in one of four states (NETCAT-8), and the block's documentation page explains each of them:

| State | Meaning |
|-------|---------|
| `open` | Something on the port answered: over TCP the connection was made, over UDP a reply came back, through an SSH tunnel the far port accepted the channel |
| `closed` | The attempt ended in an error before Timeout ran out. Usually the port refused it: a TCP reset, an ICMP port-unreachable message, or the SSH server refusing the tunnel channel. Another network error, such as `No route to host`, also reports `closed`. The reason carries the system's or the SSH server's own words |
| `timed out` | Over TCP or through an SSH tunnel channel, neither an acceptance nor a refusal came back within Timeout |
| `open or filtered` | UDP only: neither a reply nor a refusal came back within Timeout. UDP has no handshake, so that silence has two explanations nobody on the sending side can tell apart: a service received the datagram and chose not to answer, or something on the way, such as a firewall, dropped it. The state names both rather than guess |

**Pass Rule**: A port succeeds when its state is `open` or `open or filtered`; `open or filtered`, the state nmap calls `open|filtered`, counts as a pass (NETCAT-8). A `closed` or `timed out` port does not succeed. Single passes when its one port succeeds (NETCAT-3). Range passes under Pass when: One succeeds, the default and nc's own rule, when any port succeeds; All succeed when every port does. A failed check fails the block with reason Closed or Timeout for Single, and NoneSucceeded or NotAllSucceeded for Range, with the messages listed under REQ-3.4.1.9 Network tools; under Record and continue, Results still carries every port's measured state (NETCAT-7).

**UDP Truthfulness**: A UDP check sends one datagram holding the single byte `X`, as `nc -u -z` does, so that a listening service has something to answer and a closed port has something to refuse (NETCAT-13). A UDP port is never reported `open` on silence: `open` needs a reply and `closed` needs an ICMP port-unreachable refusal or another network error, and anything else is `open or filtered`, with the reason `No reply within {n} seconds`, singular "second" when {n} is 1 (NETCAT-8). Because a UDP port never answers a datagram that was not sent, Talk over UDP always sends Text or File, never Nothing (NETCAT-4). A UDP check aimed at an SSH tunnel's Local Port fails the block at run time rather than report a state, because SSH tunnels carry TCP only (NETCAT-9).

**Ports Behind an SSH Tunnel**: On the WAVE server, a check of `127.0.0.1` or `localhost` on a port that is the Local Port of one of this run's open SSH tunnels tests the tunnel's far port, not the local listener (NETCAT-9). The local listener belongs to the tunnel, so reaching it shows only that the tunnel is listening. Netcat instead opens a channel to the tunnel's far host and port over the tunnel's own SSH connection, so the state is the far port's real state: `open` when the channel opens, `closed` when the SSH server refuses it, with the SSH server's own words as the reason, and `timed out` when the channel does not open within Timeout. One INFO line per tunneled port names the far host and port that were really tested and the SSH host they were reached from. Inside a Remote Execution body no tunnel lookup happens, because loopback there is the remote host.

##### REQ-3.3.4.4 Ping tests for reachability

A reachability test is the Ping block (REQ-3.4.1.9): it sends echo requests to one host and reports which of them came back, and the block passes or fails on a stated rule.

**What Counts as an Answer**: A host has answered a request when an echo reply carrying the sequence number that was sent comes back while the block is still running; a reply that arrives after the block has ended does not count. A request is unanswered when no reply comes back within the time the block had, or when an ICMP error arrives instead of a reply. Both are loss. An ICMP error is recorded with its own detail in the Replies status column, naming the error and the address that sent it, and it is still loss (PING-15), so a router answering `time exceeded` on behalf of a host that was never reached never reads as the target being reachable.

**Loss**: Loss percent is the requests that went unanswered as a share of the requests that were sent. When Timeout cuts the block short, the share is taken against the requests the block actually sent, not against the Count that was asked for, so a block that sent 2 of 4 requests and had one of them answered reports 2 sent, 1 received and 50 percent loss.

**Pass Rule**: Pass when decides the verdict against those counts (PING-3). Any reply, the default and ping's own rule, passes when one or more requests were answered. All replies passes only when nothing was lost: every request that was sent was answered. At least N replies passes when the answered count reaches Minimum replies. At least X% replies passes when the answered share reaches Minimum rate (%). A failed rule fails the block with reason NoReply, NotAllReplied, TooFewReplies or RateTooLow, with the messages listed under REQ-3.4.1.9 Network tools; under Record and continue, Summary and Replies still carry what was measured (PING-9).

**Where Reachability Is Measured**: Reachability is a statement about the machine the block ran on, not about WAVE. A block that runs on the WAVE server reports the path from the WAVE server to the target. Inside a Remote Execution body the same block reports the path from the execution host to the target, and the two can differ, which is the point of running it there (PING-1).
