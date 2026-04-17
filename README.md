<p align="center">
  <img src="assets/Post de Instagram - Uncover vulnerabilities, master the cyber world!.png" alt="Uncover vulnerabilities, master the cyber world!" width="480"/>
</p>

<h1 align="center">TGV-VulnSpotter</h1>

<p align="center">
  <strong>Sharpen your eye. Hunt the flaw.</strong><br>
  <em>Afila tu ojo. Caza el error.</em>
</p>

<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/Challenges-4-ef4444?style=for-the-badge"/></a>
  <a href="#"><img src="https://img.shields.io/badge/Categories-4-f97316?style=for-the-badge"/></a>
  <a href="#"><img src="https://img.shields.io/badge/Bilingual-EN%20%7C%20ES-f97316?style=for-the-badge"/></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge"/></a>
  <a href="https://buymeacoffee.com/thegr8val"><img src="https://img.shields.io/badge/Buy%20Me%20A%20Coffee-%E2%98%95-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black"/></a>
</p>

---

## 🧠 What is this? / ¿Qué es esto?

**TGV-VulnSpotter** is a community-driven collection of hands-on cybersecurity challenges where you practice spotting vulnerabilities in real-world-style code snippets, cloud configurations, network rulesets, and architecture patterns.

No CTF scoring system. No leaderboard. Just you, the code, and your eye for flaws.

[ES] **TGV-VulnSpotter** es una colección de desafíos donde entrenas tu capacidad para detectar vulnerabilidades en fragmentos de código, configuraciones cloud, reglas de red y patrones de arquitectura. Sin sistema de puntuación CTF. Solo tú, el código y tu ojo para los errores.

---

## 🎯 Difficulty System / Sistema de Dificultad

```
  🟢  Apprentice  ──  Common, well-documented vulnerability.
                      Good entry point for those building their foundation.

  🟡  Hunter      ──  Requires chaining multiple observations or
                      understanding contextual behavior.

  🔴  Archmage    ──  Subtle, logic-based, or multi-vector.
                      These are the ones that slip through code review.
```

---

## 📂 Challenge Categories / Categorías

```
  ┌──────────────────┬─────────────────────────────────────────────────┐
  │  🌐 web          │  Server-side flaws — injection, auth bypass,     │
  │                  │  SSRF, template injection, broken logic          │
  ├──────────────────┼─────────────────────────────────────────────────┤
  │  ☁️  cloud        │  Misconfigurations in AWS/Azure/GCP —            │
  │                  │  IAM, storage, network policies, metadata        │
  ├──────────────────┼─────────────────────────────────────────────────┤
  │  🔌 network      │  Firewall rules, ACLs, trust relationships,      │
  │                  │  lateral movement opportunities                  │
  ├──────────────────┼─────────────────────────────────────────────────┤
  │  🔎 code-review  │  Source code audits — insecure patterns,         │
  │                  │  dangerous sinks, business logic flaws           │
  └──────────────────┴─────────────────────────────────────────────────┘
```

---

## 📋 Challenge Index / Índice de Desafíos

| ID | Category | Difficulty | Title | CWE |
|----|----------|:----------:|-------|-----|
| [VS-001](challenges/web/challenge-001.md) | 🌐 Web | 🟡 Hunter | Flask Template Trap | CWE-94 |
| [VS-002](challenges/cloud/challenge-001.md) | ☁️ Cloud | 🟢 Apprentice | The Open Bucket | CWE-732 |
| [VS-003](challenges/network/challenge-001.md) | 🔌 Network | 🟡 Hunter | East-West Exposure | CWE-284 |
| [VS-004](challenges/code-review/challenge-001.md) | 🔎 Code Review | 🟡 Hunter | Shell by Design | CWE-78 |

---

## 🔍 How to Use / Cómo Usar

```
  1.  🗂️  Pick a challenge from the index above.
  2.  📖  Read the scenario and code snippet carefully.
  3.  🕵️  Identify the vulnerability:
          ├── What type is it? (CWE category)
          ├── Where exactly is the flaw? (line / config key / rule)
          ├── How would an attacker exploit it?
          └── What is the blast radius?
  4.  ✅  Check your answer in the linked solution file.
```

---

## 📊 Challenge Stats / Estadísticas

```
  By Category                    By Difficulty
  ───────────────────────────    ──────────────────────────────
  🌐 Web          ████░░  1      🟢 Apprentice  ████░░░░  1
  ☁️  Cloud        ████░░  1      🟡 Hunter      ████████  3
  🔌 Network      ████░░  1      🔴 Archmage    ░░░░░░░░  0
  🔎 Code Review  ████░░  1
```

---

## 🗺️ Roadmap / Lo que viene

| Feature | Status |
|---------|:------:|
| 📦 More challenges across all categories | 🔄 Ongoing |
| 🌐 Web — SSRF via cloud metadata endpoint | 🔜 Planned |
| ☁️ Cloud — IAM privilege escalation path | 🔜 Planned |
| 🔎 Code Review — JWT algorithm confusion | 🔜 Planned |
| 💬 Community answer submissions per challenge | 🔜 Planned |
| 🏆 Scoring system (points per difficulty tier) | 🔜 Planned |
| 🏅 Honor Wall — top contributors & spotters | 🔜 Planned |
| 🌐 Dedicated web interface | 💭 Considering |

---

## 🤝 Contributing / Contribuir

Want to submit a challenge? Read [CONTRIBUTING.md](CONTRIBUTING.md).

Every submission must include: a plausible real-world scenario, a CWE mapping, and a corresponding solution file.

[ES] ¿Quieres enviar un desafío? Lee [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 📄 License

[MIT](LICENSE) — thegr8val

---

<p align="center">
  <strong>TGV Toolkit</strong><br><br>
  <a href="https://github.com/TheGr8Val/TGV-Grimoire">
    <img src="https://img.shields.io/badge/TGV-Grimoire-6366f1?style=for-the-badge"/>
  </a>
  <a href="https://github.com/TheGr8Val/TGV-KQLDojo">
    <img src="https://img.shields.io/badge/TGV-KQLDojo-0078D4?style=for-the-badge"/>
  </a>
  <a href="https://github.com/TheGr8Val/TGV-ReportForge">
    <img src="https://img.shields.io/badge/TGV-ReportForge-00bcd4?style=for-the-badge"/>
  </a>
  <a href="https://github.com/TheGr8Val/TGV-CareerCompass">
    <img src="https://img.shields.io/badge/TGV-CareerCompass-22c55e?style=for-the-badge"/>
  </a>
  <br><br>
  <a href="https://buymeacoffee.com/thegr8val">
    <img src="https://img.shields.io/badge/Buy%20Me%20A%20Coffee-%E2%98%95%20Support%20thegr8val-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black"/>
  </a>
  <br><br>
  <em>Built by <strong>thegr8val</strong> — Community tooling for the next generation of security practitioners.</em>
</p>
