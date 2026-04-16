---
id: VS-004
category: code-review
difficulty: Hunter
tags: [command-injection, Node.js, exec, sanitization, OS-command]
cwe: CWE-78
mitre: T1059.004
author: thegr8val
language: EN/ES
---

# Challenge VS-004 — Shell by Design

## 🧩 Scenario / Escenario

A development platform exposes an internal API that lets authorized services request diagnostic reports on running containers. The endpoint accepts a container name and returns metadata — uptime, resource usage, image info. It was built for an internal monitoring dashboard and only documented in the team wiki.

A junior engineer reviewed it last month and commented: "looks fine, uses a whitelist check."

[ES] Una plataforma de desarrollo expone una API interna que permite a servicios autorizados solicitar reportes de diagnóstico sobre contenedores en ejecución. El endpoint acepta un nombre de contenedor y devuelve metadatos: uptime, uso de recursos, info de imagen. Fue construido para un dashboard de monitoreo interno y solo está documentado en el wiki del equipo.

Un ingeniero junior lo revisó el mes pasado y comentó: "se ve bien, usa una verificación de lista blanca."

---

## 🔍 Find the vulnerability / Encuentra la vulnerabilidad

```javascript
const express = require("express");
const { exec } = require("child_process");
const app = express();

app.use(express.json());

// Allowed container name characters: alphanumeric, hyphens, underscores
const SAFE_CONTAINER_NAME = /^[a-zA-Z0-9_-]+$/;

function getContainerInfo(containerName, callback) {
  if (!SAFE_CONTAINER_NAME.test(containerName)) {
    return callback(new Error("Invalid container name"), null);
  }

  const cmd = `docker inspect --format='{{json .State}}' ${containerName} && docker stats ${containerName} --no-stream --format "{{json .}}"`;

  exec(cmd, { timeout: 5000 }, (err, stdout, stderr) => {
    if (err) {
      return callback(new Error("Container not found or inaccessible"), null);
    }
    callback(null, stdout);
  });
}

app.post("/api/v1/diagnostics", (req, res) => {
  const { container, report_format } = req.body;

  if (!container) {
    return res.status(400).json({ error: "container name required" });
  }

  getContainerInfo(container, (err, data) => {
    if (err) {
      return res.status(404).json({ error: err.message });
    }

    if (report_format === "verbose") {
      const verboseCmd = `echo "=== Diagnostic Report ===" && docker inspect ${container}`;
      exec(verboseCmd, { timeout: 5000 }, (err2, stdout2) => {
        if (err2) return res.status(500).json({ error: "Verbose report failed" });
        return res.json({ report: stdout2 });
      });
    } else {
      return res.json({ report: data });
    }
  });
});

app.listen(3000, "127.0.0.1", () => {
  console.log("Diagnostics service running on port 3000");
});
```

---

## 💡 Hint / Pista (optional)

The whitelist works. But there's more than one place where user input reaches a shell command.

[ES] La lista blanca funciona. Pero hay más de un lugar donde el input del usuario llega a un comando de shell.

---

## 📊 Difficulty / Dificultad

- Apprentice — Common, well-documented vuln
- **Hunter — Requires chaining or context** ← This one
- Archmage — Subtle, logic-based, or multi-vector

Most people find the whitelist, assume the vulnerability is there, and declare the code safe. Read further.

---
🔒 Solution in: [solutions/code-review/solution-001.md](../../solutions/code-review/solution-001.md)
