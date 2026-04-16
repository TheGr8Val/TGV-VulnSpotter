# Contributing to TGV-VulnSpotter

Thanks for wanting to contribute. This repo grows through community submissions — security practitioners sharing what they know.

---

## Before You Submit

- The vulnerability must be **original or properly attributed** (cite the CVE, writeup, or source if inspired by a real incident).
- Every challenge **must have a corresponding solution file**. No orphaned challenges.
- The vulnerability must map to a **CWE**. If you can map it to a MITRE ATT&CK technique, do it.
- The code/config in the challenge should look **plausible** — not a textbook strawman. The flaw should require thought to spot.
- Avoid duplicating an existing challenge ID or concept unless it's a meaningfully different angle.

---

## Fork & PR Workflow

1. **Fork** this repository.
2. Create a branch named `challenge/[category]-[short-title]`  
   Example: `challenge/web-ssrf-metadata`
3. Add your challenge file to `challenges/[category]/challenge-XXX.md`
4. Add the solution file to `solutions/[category]/solution-XXX.md`
5. Update the challenge table in `README.md` with your new entry.
6. Open a Pull Request with:
   - A brief description of the vulnerability
   - Why you think it's worth including (real-world relevance, subtlety, teaching value)

---

## Challenge Template

Copy this into your new `challenge-XXX.md`:

```markdown
---
id: VS-XXX
category: web | cloud | network | code-review
difficulty: Apprentice | Hunter | Archmage
tags: []
cwe: CWE-XXX
mitre: TXXXX.XXX (if applicable)
author: your-handle
language: EN/ES
---

# Challenge VS-XXX — [Short Title]

## 🧩 Scenario / Escenario
[1-2 sentence context describing the environment or use case.]

[ES] [Traducción del escenario]

## 🔍 Find the vulnerability / Encuentra la vulnerabilidad

\`\`\`[language]
[Code snippet, config, or pseudo-architecture with the vulnerability embedded]
\`\`\`

## 💡 Hint / Pista (optional)
[A subtle nudge. Don't give it away.]

[ES] [Pista en español]

## 📊 Difficulty / Dificultad
- Apprentice — Common, well-documented vuln
- Hunter — Requires chaining or context
- Archmage — Subtle, logic-based, or multi-vector

---
🔒 Solution in: solutions/[category]/solution-XXX.md
```

---

## Solution Template

Copy this into your new `solution-XXX.md`:

```markdown
---
id: VS-XXX
challenge: challenges/[category]/challenge-XXX.md
---

# Solution VS-XXX — [Short Title]

## ✅ Vulnerability / Vulnerabilidad
[Name + CWE]

## 🔎 Explanation / Explicación
[Clear technical explanation of what the vulnerability is and why it exists.]

[ES] [Explicación en español]

## 💥 Impact / Impacto
[What an attacker could do with this.]

## 🛡️ Remediation / Remediación
[How to fix it — include corrected code or config if applicable.]

## 📚 References / Referencias
- [OWASP / CVE / CWE link]
- [MITRE link if applicable]
```

---

## Quality Bar

| Requirement | Required |
|-------------|----------|
| CWE mapping | Yes |
| Corresponding solution file | Yes |
| Plausible real-world scenario | Yes |
| Original or attributed | Yes |
| MITRE ATT&CK mapping | Encouraged |
| Bilingual (EN + ES blocks) | Encouraged |

---

## Code of Conduct

Be direct. Be technical. Be respectful.

- No gatekeeping. Security is for everyone.
- Feedback on PRs should be constructive — explain *why*, not just *what*.
- Discrimination of any kind will result in an immediate ban.

This project follows the [Contributor Covenant](https://www.contributor-covenant.org/) v2.1.

---

Questions? Open an issue or reach out via [@thegr8val](https://github.com/thegr8val).
