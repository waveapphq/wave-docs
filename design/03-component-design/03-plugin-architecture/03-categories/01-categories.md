#### 3.3.3 Plugin Categories

Plugins organize into seven categories matching frontend palette grouping and backend metadata.

**System Plugins**
- Shell Script / Shell Exec (a script, or one command line in a chosen shell, on the WAVE server)
- Python script execution
- File I/O operations
- Logging

**Network Plugins**
- HTTP/HTTPS client
- Ping connectivity checks
- Netcat port connectivity

**Remote Plugins**
- SSH Session Open/Exec/Script/Close (persistent session lifecycle with remote scripting)
- SSH Tunnel Open/Close (port forwarding lifecycle)
- SFTP file transfer (upload, download and file operations over an open SSH session)
- Remote Execution (runs the Blocks placed in its body on an execution host reached through an SSH session)

**Database Plugins**
- SQL database operations
- OpenSearch operations

**Kubernetes Plugins**
- K8s Connect (cluster context: authentication, verification, run-token acquisition)
- K8s Get / K8s Events / K8s Logs / K8s Top (kind-agnostic read verbs)
- K8s Apply / K8s Delete / K8s Scale / K8s Rollout (kind-agnostic write verbs)
- K8s Exec / K8s Script / K8s Copy (command, script and file transfer in containers)

**Helm Plugins**
- Helm Install / Helm Upgrade / Helm Rollback / Helm Uninstall / Helm Test / Helm Status (release lifecycle through the machine's own helm, riding the K8s Session)

**Data Plugins**
- CSV parsing
- Excel parsing
- XML parsing
- JSON parsing
