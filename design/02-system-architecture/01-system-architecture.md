## 2. System Architecture

### 2.1 Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                          User Interfaces                         │
├────────────────────────────────┬─────────────────────────────────┤
│            Web GUI             │         REST API Client         │
│       (Test Engineering)       │         (External Tools)        │
└───────────────┬────────────────┴────────────────┬────────────────┘
                │                                 │
                ↓                                 ↓
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                          │
│                    (FastAPI + WebSocket)                        │
└─────────────────────────────────────────────────────────────────┘
         │
         ↓
┌────────────────────────────────────────────────────────────────┐
│                      Business Logic Layer                      │
├─────────────┬──────────────┬─────────────┬─────────────────────┤
│  Workflow   │   Execution  │  Validation │    Reporting        │
│  Manager    │    Engine    │   Engine    │    Engine           │
└─────────────┴──────────────┴─────────────┴─────────────────────┘
         │
         ↓
┌───────────────────────────────────────────────────────────────┐
│                        Plugin System                          │
├────────────┬────────────┬────────────┬────────────┬───────────┤
│    Core    │    K8s     │  Network   │    Data    │   Script  │
│  Plugins   │  Plugins   │  Plugins   │  Plugins   │  Plugins  │
└────────────┴────────────┴────────────┴────────────┴───────────┘
         │
         ↓
┌────────────────────────────────────────────────────────────────┐
│                      Storage Layer                             │
├─────────────────────┬─────────────────────┬────────────────────┤
│         File        │        Config       │     Artifacts      │
│       Storage       │        Store        │       Store        │
└─────────────────────┴─────────────────────┴────────────────────┘
```

### 2.2 Component Interaction Flow

```
User → GUI → API Gateway → Business Logic → Plugin System → External Systems
                  ↓                ↓              ↓
             WebSocket      Storage Layer      K8s/SSH/DB
              Updates         (YAML/JSON)      Connections
```

### 2.3 Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Linux Host System                        │
├─────────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────────────────┐     │
│  │            WAVE Application (Single Process)           │     │
│  │  ┌──────────────────┐                                  │     │
│  │  │  FastAPI Server  │                                  │     │
│  │  │   (Port 8000)    │                                  │     │
│  │  └──────────────────┘                                  │     │
│  └────────────────────────────────────────────────────────┘     │
│                               │                                 │
│  ┌────────────────────────────┼────────────────────────────┐    │
│  │                     System Resources                    │    │
│  │  • Data folder named in the config file                 │    │
│  │  • Network Access (K8s API, SSH, HTTP/S)                │    │
│  │  • Process Management                                   │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                               │
               ┌───────────────┼────────────────┐
               │               │                │
        ┌──────▼──────┐ ┌──────▼──────┐ ┌───────▼──────┐
        │ Kubernetes  │ │   Node SSH  │ │  OpenSearch  │
        │   Cluster   │ │   Access    │ │   Cluster    │
        └─────────────┘ └─────────────┘ └──────────────┘
```

### 2.4 WAVE Host Independence

#### 2.4.1 Test Outcomes and the WAVE Host

##### REQ-4.6.8 Test outcomes never depend on hidden WAVE host state

WAVE owns everything that can decide a test's result. The account WAVE runs under, the WAVE host's environment variables and its system files never change an outcome silently, so a test that passes on one WAVE host passes on another against the same systems under test. Where WAVE still uses something of the WAVE host, an admin turns it on in Settings, Settings shows what it is, and the run log says when a block used it.

**Where the WAVE host could reach a run, and what WAVE uses instead**:

| Concern | What WAVE uses | What it never reads |
|---------|----------------|---------------------|
| SSH host keys | WAVE's trusted SSH hosts list (REQ-4.5.14) | the account's known hosts file |
| SSH login and jump hosts | the block's own rows and WAVE's session chain (REQ-4.5.15) | the account's SSH config file, default keys and SSH agent |
| Server certificates | the block's CA, or the Additional Certificates plus the public authorities chosen in Settings (REQ-4.5.16) | environment variables that point the TLS library at other certificates or configuration |
| Web proxy | the Proxy setting, off by default (REQ-4.7.14), or the HTTP block's own choice (REQ-3.4.3.9) | the WAVE host's proxy variables, unless an admin picks them as the Proxy setting's source |
| Helm | a clean environment and the block's own registry credentials (REQ-3.5.3.6) | helm's settings, registry logins and Docker credentials on the WAVE host |
| File types | WAVE's own file-type table (REQ-3.4.3.10) | the WAVE host's file-type table |
| SQL drivers | the block's rows, WAVE's own SQLite and the program name WAVE (REQ-3.4.1.19) | Oracle's network configuration, the WAVE host's time zone and its SQLite library |
| Relative paths | the home folder of the account the block runs as, or the block's Working Directory (REQ-3.4.3.12) | the folder WAVE was started from |
| Script working folder | the temp folder Python reports where the block runs (REQ-3.4.3.11) | a fixed temp path |
| Data folder | the config file setting written at installation (REQ-4.7.13) | the folder WAVE was started from |
| Config file | the file in WAVE's own installation folder, or the one an environment variable names by absolute path (REQ-4.7.15) | the folder WAVE was started from |
| Text | UTF-8 (REQ-4.6.9) | the WAVE host's locale |

**What stays with the WAVE host on purpose**: Scripts (Shell Script, Shell Exec and Python) inherit the WAVE host's environment unchanged, proxy variables included, because the tools a script calls must find what they would find in a terminal on that machine (SHRUN-2). A block running on a remote host inside a Remote Execution body runs with that host's own environment and proxy settings, because giving a check that host's vantage point is the reason to run it there; WAVE's certificate trust still travels with it (REQ-4.5.16). The machine name and account a database driver reports stay as they are, because they are true facts a database administrator uses to trace a connection.

##### REQ-4.6.9 UTF-8 text everywhere

WAVE reads and writes all text as UTF-8, whatever the WAVE host's locale: its own storage, the run's log files, block import and export, report files, and the output of the programs it runs whenever it reads that output as text. A test file, a log or an export is therefore the same bytes on every WAVE host, and a name or value outside ASCII never turns into an encoding error on a host with a different locale. The build enforces the rule: code that opens a text file without naming its encoding fails the build's lint check, together with a check that also covers the calls the lint rule cannot see.

**File names and program arguments**: File names and the arguments WAVE passes to the programs it starts are UTF-8 too. A host whose file-name encoding is not UTF-8 would store the same name as different bytes than every other WAVE host, so WAVE refuses to start there, in the way it refuses an unusable config file, and its message names the one setting that turns UTF-8 on. A host with a UTF-8 locale or no locale at all is never refused, and neither is macOS or Windows.

**Remote hosts and blocks**: On a remote host WAVE's own worker runs in UTF-8 whatever that host's locale. A Python block's printed output is UTF-8 on every host. Output read from an SSH session is decoded as UTF-8; a byte that is not UTF-8 shows as the replacement character and never fails the block, as with a local program's output. What WAVE sends over an SSH session is never altered that way: a script or command that cannot be written as UTF-8 fails its block before anything is sent. Archive member names in a Kubernetes copy are UTF-8.

**Downloads and incoming text**: A file WAVE hands out under a name taken from the user's data keeps the letters, digits and marks of every language in that name: the export of a Test called `Grüße 測試` is saved as `Grüße-測試.yaml`, and its Test Data archive as `testdata-Grüße-測試.zip`. The name travels in the encoded form of the download header, with a plain stand-in beside it for a client that reads only that form. Text that cannot be written as UTF-8 (half of a character pair) never enters WAVE: a request, an imported file, an MCP recipe or a message sent over the live connection that holds such text is refused whole before anything is stored or started, and the answer says where the text is. An upload whose form names a charset other than UTF-8 is refused the same way.

#### 2.4.2 Network Trust and Routing

##### REQ-4.5.16 Certificate trust

Every place WAVE verifies a server certificate follows one rule: the HTTP block, the SQL block (PostgreSQL, MySQL / MariaDB, Oracle, SQL Server), the OpenSearch block, every Kubernetes block, and Helm for both its cluster and its chart repositories. The editor helpers that connect from the WAVE host, SQL Get Tables and OpenSearch Get fields, follow it too.

**The rule**: With Verify Server Certificate on, a CA Certificate set on the block, or on its session for the Kubernetes and Helm blocks, is the only certificate the block trusts, as curl's own CA option behaves. With it blank, the block trusts the Additional Certificates in Settings plus the public certificate authorities chosen there. Any trusted certificate completes a chain, whether it is a root or an intermediate: a certificate an admin adds is a complete trust point on its own and the check stops there, as curl does, so there is no separate chain setting and the certificate added decides how far the check goes. Environment variables of the WAVE host that name certificate files or folders, log TLS secrets or change the TLS library's configuration never change trust.

**Public certificate authorities**: Settings > Network > Certificates offers two sources. WAVE's built-in list, the default, is Mozilla's public list bundled with WAVE: dated, with a known count, and the same on every WAVE host, so a WAVE host whose system store is old or modified cannot change a result. This WAVE host's store follows the operating system's own certificate bundle and its updates, for the case where the built-in list of an older WAVE falls behind; WAVE reads that bundle from the operating system's standard locations itself, and choosing it is refused at Save when the WAVE host has none. Settings shows the built-in list's date and count and lists its authorities read-only, or, for the host store, shows the store, its certificate count and when it last changed.

**Additional Certificates**: An admin adds the certificates of internal authorities, roots or intermediates, by pasting PEM text or choosing a PEM or DER file. Only certificate authority certificates are accepted, a pasted private key is refused, an expired certificate is marked but allowed, and a certificate already in the list is shown as already added. Each certificate is identified by its SHA-256 fingerprint. The list lives in WAVE's data folder as shared work and is part of backup and restore; the choice of public source is this WAVE's own setting, kept in its config file (REQ-4.7.17).

**Remote hosts**: A block inside a Remote Execution body verifies on the remote host by the same rule. WAVE's built-in list and the Additional Certificates travel with the run; when the source is this WAVE host's store, the remote host uses its own store plus the Additional Certificates.

**Run log**: Each block run that verifies a server certificate with a blank CA writes one DEBUG line naming the source used and, when there are any, how many additional certificates it trusted. A block whose CA is set writes none.

**TLS off**: A PostgreSQL connection with its TLS row off is a plain connection: WAVE asks the server for no TLS at all rather than trying it unverified.

##### REQ-4.7.14 Proxy for web requests from the WAVE host

Settings > Network > Proxy turns on one proxy for the web requests (HTTP and HTTPS) WAVE makes from the WAVE host. It is off by default, so a WAVE host's proxy variables decide nothing until an admin chooses to use them. Proxy is a parent with two pages: Proxy Setup holds the setting, and Proxy Blocks shows which blocks use it.

**Source**: With "Use a proxy" on, the source is either "WAVE host's settings" or "Custom", and both are offered on every WAVE host. Only one source is ever used: with Custom, or with the proxy off, WAVE's web requests ignore the WAVE host's own proxy variables. Turning the proxy off or changing the source discards the Custom values when the page is saved: a Save keeps only what the page shows (REQ-4.7.16).

**WAVE host's settings**: The source is the WAVE host's HTTPS_PROXY, HTTP_PROXY and NO_PROXY variables, read each time a request is made and shown read-only on the page with any password inside a value masked. No other variable of the WAVE host takes part. A value written without a protocol is read as an http address, and a value that names no port means port 80. Saving this source with the proxy on is refused when the WAVE host sets neither HTTPS_PROXY nor HTTP_PROXY, so the choice never turns silently into a direct connection. If the WAVE host sets neither variable when a block runs, which can happen after a restart, the request connects directly and the run log says so with a warning, once per block run. A value that names a proxy WAVE cannot use, one reached over TLS or a SOCKS proxy, is refused at Save too; at run time it fails only a block whose request would use that value, with the same message.

**Custom**: The custom proxy is an address, which is an IPv4 address, an IPv6 address or a host name, and a port. No protocol is entered: WAVE reaches the proxy over a plain connection, and the proxy still carries HTTPS traffic. Both are required while the source is Custom and the proxy is on. The address holds at most 253 characters and the port is a whole number from 1 to 65535. A login is optional: a username may stand without a password, a password needs a username, and saving an empty username also removes the stored password. The username and the password hold at most 255 characters each, and the username holds no colon. The password is sealed at rest like every WAVE credential and is never shown: once saved the page shows it masked, and replacing or removing it goes through the clear-and-edit confirm used for every protected value.

**One endpoint per protocol**: The proxy in force is one endpoint for https targets and one for http targets, each with its own address, port and login. Custom fills both from its one address and one login. The WAVE host's settings fill each from its own variable. Each of the WAVE host's two proxy values keeps its own login. A protocol with no endpoint connects directly: a WAVE host that sets only HTTPS_PROXY proxies https targets and connects directly to http targets, and the reverse, with no fall-back from one to the other.

**One rule for every web request**: The proxy carries the web requests WAVE itself makes from the WAVE host: the HTTP block on its WAVE setting proxy choice (REQ-3.4.3.9), every Kubernetes block over REST and over the exec stream, the OpenSearch block and the editor's field lookup, which connects the way the block does, and Helm for both cluster and chart traffic. Kubernetes and OpenSearch connections pass through the proxy as a tunnel it opens to the server, so TLS and its certificate checks stay end to end with the server; an https request of the HTTP block is tunneled the same way and keeps the block's own TLS settings. Connections that are not web requests, such as SSH, SQL, Netcat and Ping, never use it.

**The route decision**: Whether a request goes through the proxy or connects directly is one decision, made the same way for every transport, for each request and for each hop of a redirect. It compares the request's address, which is its protocol, host and port in a normal form: the host in lower case, an IPv6 host in brackets, and the port left out when it is the protocol's default. The exec stream of a Kubernetes block takes the protocol of the cluster's server URL, https or http.

With Custom, an address list of at most 100 entries decides, in one of two modes. "All addresses except the list", the default, sends every request through the proxy except to the addresses the list matches. "Only the addresses in the list" sends only requests to matched addresses through the proxy and connects directly to everything else; it needs at least one entry. An address that no entry matches takes the mode's default. Each entry is Exact or Regex. An entry holds at most 256 characters. An Exact entry is a protocol, a host and an optional port, kept in the normal form, and it matches when all three equal the request's address; anything after the host and port, a trailing slash included, is refused at Save, so one address cannot be listed in two spellings. A Regex entry is a regular expression, kept as typed; it is matched ignoring case, because the host part of an address is lower case, and it matches when it is found anywhere in the address. An entry equal to an earlier one is refused at Save. Matching one address against one regular expression is limited to 100 milliseconds per match: when the limit is passed the block run fails with a message naming the pattern and the address, and other runs keep going. An expression is checked at Save by the engine that matches it in a run, so what Save accepts is what a run uses.

A stored custom proxy that lacks an address or a port, or a stored entry that would fail the Save rules, as can happen when the config file is edited by hand, fails the block run with the message Save gives for that case; WAVE still starts.

Local addresses, which are localhost, every 127.x.x.x address and ::1, connect directly whatever the list says, unless "Use the proxy for local addresses too" is on; with it on they follow the list like any other address.

With the WAVE host's settings, the WAVE host's NO_PROXY value decides: a lone asterisk matches every host; an entry matches that host and its subdomains, with or without a leading dot; an IP address matches exactly; an address range matches every address inside it; case and any port written in an entry are ignored. Local addresses always connect directly under this source.

**Where the rules are checked**: The refusals named above, for the WAVE host's values, the custom address, port and login, and the list entries, are one set of rules that exists once, on the server. The Proxy Setup page shows them while the values are typed, each under its field or list row, and keeps Save disabled until nothing is invalid; the server applies the same rules again at Save, so the page and the server cannot disagree.

**Helm**: Helm is a separate program that makes its own connections, so WAVE decides the route for the two addresses it knows, the cluster and the chart source (a chart repository, a chart URL or an OCI registry), and hands helm the result. When the chart source goes through the proxy, helm's clean environment carries the proxy variables. The cluster's route is written into the cluster configuration WAVE generates for the call; when the cluster connects directly while the chart source is proxied, the cluster's host is the one NO_PROXY entry helm is given. Helm cannot keep the two routes apart when the cluster connects directly and the chart source's host is the cluster's host, or a name under it, on the same port: the chart source then connects directly too, and the run log says so. Any other host helm contacts during a download takes the chart source's route. A chart source on a local address is never proxied by helm, whatever the local-addresses switch says, and the run log says so (REQ-3.5.3.6).

**Proxy Blocks**: The second page lists every plugin's block in one of three sections, "Blocks that use this proxy", "Blocks that do not use this proxy" and "Not applicable", whether the proxy is on or off. The lists are built from each plugin's own proxy declaration and from nothing else (REQ-3.4.2.10). Remote Execution is a container and is not listed: the blocks inside it state their own.

**What it does not touch**: Blocks running on a remote host are not affected and keep that host's own proxy settings. Scripts keep the WAVE host's environment unchanged, proxy variables included (SHRUN-2). The proxy settings are this WAVE's own settings, kept in its config file (REQ-4.7.17).

**Run log**: A block run on the WAVE host whose requests went through the proxy writes one DEBUG line naming the proxy by its address and port, never its login. A request that connected directly writes no such line, and neither does the editor's field lookup, where no run is open, or a block running on a remote host. The run log names a direct connection in two cases only: the warning above, when the proxy is on with the WAVE host's settings and the WAVE host sets no proxy variable, and a Helm chart source that the route decision sends through the proxy but helm connects directly (REQ-3.5.3.6).

---