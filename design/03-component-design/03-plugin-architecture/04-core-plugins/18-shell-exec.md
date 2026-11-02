##### REQ-3.4.1.18 Shell command execution

The Shell Exec block runs one command line on the WAVE server in a shell the author chooses and returns what the command printed and its exit code.

**Purpose and Locus**: Shell Exec runs one command line on the WAVE server (SHRUN-17). Command is required. Shell names the shell that runs it, `/bin/sh` by default, and may carry flags. The shell is handed the whole of Command as one line to run, so pipes, redirects, globs and `&&` work as they do at a prompt. Shell Exec is not remotable: inside a Remote Execution body it carries the L badge and runs on WAVE, under the same mixed-locus rule as every other non-remotable block.

**Shared Local-Process Behavior**: Shell Exec shares the Shell Script block's local-process behavior, described under REQ-3.4.1.1 Shell script execution, in everything that is not about a script (SHRUN-18). What that subsection states for these areas holds for Shell Exec unchanged:

| Area | Shell Exec |
|------|-----|
| Input Variables | A map from a variable name to a WAVE value or expression, exported into the command's environment, with the same name and conversion rules as the Shell Script block's Input Variables: an empty entry is empty text, and an entry whose value is null fails the block before the command starts and names the entry |
| Inherited environment | The WAVE server's environment as the server has it, with nothing removed and nothing built up in its place; `WAVE_TESTDATA_DIR` and `WAVE_ARTIFACTS_DIR` added |
| Standard input | Empty, so a command never waits for input |
| Working Directory | empty by default, meaning the temp folder Python reports on the machine the block runs on, with Browse and the same path rules; Browse stays visible inside a Remote Execution body |
| Timeout | 300 seconds by default, must be positive |
| Start line | The same INFO line as the block starts, naming the shell as written and never echoing the command: `Running the command under {shell} on the WAVE server` |
| Log Output | The same checkbox and the same live-line and secret rules |
| Stop on pattern | The same checkbox, Pass pattern and Fail pattern rows and outcomes; a pass match logs the same INFO line, `Output matched the pass pattern: {line}`, the line cut to 200 characters and omitted when the matching stream is marked secret; the command gets the same pattern stop, terminated first, then a short grace, then the group is killed |
| Failure Handling and outputs | Stdout, Stderr, Exit Code and Success always; Error and Reason while On error is Record and continue; the same Exit Code rules, 137 included; each captured stream keeps its last 16 MiB |
| Process lifecycle | The command and everything it starts run in one process group the block owns, under the same watchdog: ended whole at once on a timeout, a Stop and a shutdown, and terminated, given a short grace, then killed on a pattern match; anything the command left running is ended when it ends on its own, and a Stop reaches a Shell Exec block inside a Parallel branch because it cancels every lane |

**What Shell Exec Lacks**: Shell Exec has no Script Type, no Arguments and no variable exports: the command line carries its own arguments, and there is no script to set variables. Its Reason is therefore one of NonZeroExit, Timeout, FailPattern and StartFailed, never OutputVariable.

**Glyph and Palette**: Shell Exec carries the Terminal glyph, the one glyph every Exec block in the product uses, while every Script block carries FileTerminal; the category color is what keeps the families apart, so Shell Exec and K8s Exec read as the same kind of block in different categories (SHRUN-19). Its display name is "Shell Exec" and it sits in the System category, where the palette order is Shell Script, Shell Exec, Python, File I/O, Log. Its documentation page carries one version, 1.0.0, whose changelog names every setting (FIO-11).

**Error Handling**: Shell Exec's messages name the command where the Shell Script block's name the script. The failures the two blocks share (Timeout not positive, the pattern rules, variable names, the working directory, a fail-pattern match) carry the Shell Script block's messages. A setting given the wrong shape, or a value outside the choices a setting offers, is refused before the command starts, as it is for the Shell Script block. The ones specific to Shell Exec are:

| Failure | Message | Reason |
|---------|---------|--------|
| Command is empty | `Missing required parameter: command` | Validation |
| Shell is blank | `Parameter 'shell' cannot be empty` | Validation |
| The shell is not installed on the server | `Shell not found on the WAVE server: {shell}` | StartFailed |
| The operating system refuses to start the command | `Could not start the command: {os error}` | StartFailed |
| The shared path resolver refuses the working directory | The resolver's own message, passed through unchanged, for example `artifact:// scheme requires artifacts directory, but none configured in context` | StartFailed |
| The working directory is not there | `Working directory not found: {path}` | StartFailed |
| The command exits with a non-zero code | `Command exited with code {code}: {line}`, where the line is the last line of Stderr that is not blank, trimmed and cut to 200 characters; just `Command exited with code {code}` when Stderr is empty or marked secret | NonZeroExit |
| Timeout runs out | `Command timed out after {n} seconds` | Timeout |

The run console also receives two WARN lines of its own:

| Situation | Console line |
|-----------|--------------|
| A failure recorded under Record and continue | `Command failed ({reason}): {message}` |
| A captured stream dropped earlier output | `Stdout truncated: {n} bytes dropped` or `Stderr truncated: {n} bytes dropped` |
