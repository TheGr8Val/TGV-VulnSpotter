---
id: VS-003
category: network
difficulty: Hunter
tags: [firewall, lateral-movement, microsegmentation, east-west, misconfiguration]
cwe: CWE-284
mitre: T1021
author: thegr8val
language: EN/ES
---

# Challenge VS-003 — East-West Exposure

## 🧩 Scenario / Escenario

A mid-sized SaaS company segmented their production environment into three network zones: `DMZ`, `APP`, and `DATA`. The firewall ruleset below was written by a contractor during a migration and approved after a brief review. The security team confirmed north-south traffic looked clean and signed off.

What they didn't check was what happens if an attacker already has a foothold in the DMZ.

[ES] Una empresa SaaS de tamaño mediano segmentó su entorno de producción en tres zonas de red: `DMZ`, `APP` y `DATA`. El conjunto de reglas de firewall siguiente fue escrito por un contratista durante una migración y aprobado tras una revisión breve. El equipo de seguridad confirmó que el tráfico norte-sur parecía limpio y dio el visto bueno.

Lo que no verificaron fue qué pasa si un atacante ya tiene un punto de apoyo en la DMZ.

---

## 🔍 Find the vulnerability / Encuentra la vulnerabilidad

```
# Production Firewall Ruleset — fw-prod-01
# Zones: DMZ (10.0.1.0/24), APP (10.0.2.0/24), DATA (10.0.3.0/24)
# Direction: src -> dst
# Format: RULE_ID | SRC | DST | PORT/PROTO | ACTION | COMMENT

R001 | ANY             | DMZ (10.0.1.0/24)  | 443/TCP      | ALLOW  | Inbound HTTPS to reverse proxy
R002 | ANY             | DMZ (10.0.1.0/24)  | 80/TCP       | ALLOW  | Inbound HTTP (redirect only)
R003 | DMZ             | APP (10.0.2.0/24)  | 8080/TCP     | ALLOW  | Proxy to app servers
R004 | APP             | DATA (10.0.3.0/24) | 5432/TCP     | ALLOW  | App to PostgreSQL
R005 | APP             | DATA (10.0.3.0/24) | 6379/TCP     | ALLOW  | App to Redis cache
R006 | DMZ             | APP (10.0.2.0/24)  | 0-65535/TCP  | ALLOW  | DMZ to APP unrestricted (temp - migration)
R007 | 10.0.2.15/32    | DATA (10.0.3.0/24) | 22/TCP       | ALLOW  | Ops bastion to DATA zone SSH
R008 | ANY             | ANY                | ANY          | DENY   | Default deny all
```

---

## 💡 Hint / Pista (optional)

Rules are evaluated top-down. Look at what's explicitly allowed versus what's implied by overly broad rules. Then think about what a post-exploitation tool would do next from DMZ.

[ES] Las reglas se evalúan de arriba a abajo. Observa qué está explícitamente permitido versus lo que está implícito en reglas demasiado amplias. Luego piensa en qué haría una herramienta de post-explotación desde la DMZ.

---

## 📊 Difficulty / Dificultad

- Apprentice — Common, well-documented vuln
- **Hunter — Requires chaining or context** ← This one
- Archmage — Subtle, logic-based, or multi-vector

You need to think like an attacker who has already landed in DMZ. What can you reach? What should be unreachable?

---
🔒 Solution in: [solutions/network/solution-001.md](../../solutions/network/solution-001.md)
