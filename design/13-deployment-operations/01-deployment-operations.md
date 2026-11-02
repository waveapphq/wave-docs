## 13. Deployment and Operations

### 13.1 Deployment Process

The deployment process includes the following steps:

**Prerequisites Check**:
- Verify Python 3.12+ installation
- Verify Node.js 22 LTS installation
- Verify Git 2.0+ availability

**Installation Steps**:
1. Install backend dependencies and package
2. Build frontend application bundle
3. Create the configuration file from the example shipped with WAVE and write the data folder location into it
4. Make the installation's own secret key with the documented key command and write it into the configuration file
5. Start the application server

### 13.2 Configuration Management

**Resolution order**:
Settings resolve from three sources, each overriding the one before it: built-in
defaults, the configuration file, and the environment variables that name a
setting. Defaults are secure, so an installation that sets nothing refuses
browser origins rather than accepting all of them.

**What a deployment must supply**:
Two settings have no default. The secret encryption key is made once per
installation with the documented key command, since WAVE ships none, and every
stored secret and credential is sealed under it. Editing it in the
configuration by hand leaves the stored secrets unreadable. Startup fails loudly when the key is missing, malformed, or unable to
decrypt existing data, rather than running in a state where runs stall on
secrets that cannot be opened; when the key is missing, the message names the
key command.

The data folder is the other. Its location is written into the configuration
file when WAVE is installed, and a relative value starts at the folder that
holds that file, never at the folder WAVE was started from. Startup refuses to
proceed without it, naming the setting to add, and logs the absolute path in
use; Settings shows the same path read-only (REQ-4.7.13).

Beyond these two, a deployment chooses the browser origins the API accepts, where
execution artifacts are written, log verbosity, and whether the
administrative surface exists at all. Anything the deployment does not state
falls back to a default.

**Configuration file**:
One file per WAVE, in a structured text format, holding the settings a
deployment pins rather than passes at launch - the data folder among them - and
the settings an admin changes in Settings, which WAVE writes back into it
(REQ-4.7.15, REQ-4.7.16). WAVE finds it in its own installation folder, or at the
absolute path an environment variable names; no environment name selects among
several files. The environment-variable override of the data folder stays for
deployments that place it at launch.

The concrete inventory - literal variable names, the settings each one reaches,
coercion rules, and defaults - is operational reference material rather than
architecture, and is maintained with the code it describes in the backend and
configuration READMEs.

---