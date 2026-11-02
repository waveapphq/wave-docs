#### 3.8.4 Secret Handling

Secret handling over MCP follows one principle: the assistant obeys the same masking policy as every other client, with the user in control of anything that crosses it.

##### REQ-5.2.7.12 Secret write modes

Wherever authoring involves a secret - a secret variable in a test, suite, regression, or the system and global scopes, a credential field in a profile, a secret-valued block setting - the client chooses one of three modes per secret. Provided value: the value is passed in directly and sealed at rest like any other secret write. Server-side file reference: the value is read by the server from a named file at authoring time, so it never appears in the client conversation at all; this is the recommended mode wherever the conversation transcript is itself a concern. Declared empty: the secret is created without a value, and the authoring result ends with explicit manual action items enumerating what must be filled in before the affected definitions can run. A declared-empty secret that a block's output fills at run time is not owed and gets no action item. A secret that an import or restore could not open, or that arrived as a masked placeholder, is stored as null and owed the same way.

##### REQ-5.2.7.13 Secret read policy

MCP responses mask secret values everywhere - discovery listings, recipes read back, execution variables, logs, reports - exactly as the equivalent UI surfaces mask them. When the deployment's system settings permit secret reveal, the assistant may retrieve a value, but only after warning the user and receiving explicit consent in the conversation for that retrieval. When masking policy does not permit reveal, the assistant states plainly that the value cannot be retrieved and presents the masked form. The MCP never widens reveal policy; it can only be as permissive as the system settings already are.
