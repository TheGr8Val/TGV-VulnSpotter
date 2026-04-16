---
id: VS-002
category: cloud
difficulty: Apprentice
tags: [S3, misconfiguration, public-access, data-exposure, AWS]
cwe: CWE-732
mitre: T1530
author: thegr8val
language: EN/ES
---

# Challenge VS-002 — The Open Bucket

## 🧩 Scenario / Escenario

A startup's DevOps team set up an S3 bucket to serve static assets for their marketing site — logos, CSS, JS bundles. A junior engineer added a bucket policy to make delivery work without signed URLs. The site launched, everything worked, and the ticket was closed.

Six months later, the same bucket is also being used to store monthly financial exports and HR onboarding documents. Nobody updated the policy.

[ES] El equipo de DevOps de una startup configuró un bucket S3 para servir assets estáticos de su sitio de marketing: logos, CSS, bundles de JS. Un ingeniero junior agregó una política de bucket para que la entrega funcionara sin URLs firmadas. El sitio lanzó, todo funcionó, y el ticket se cerró.

Seis meses después, el mismo bucket también se usa para almacenar exportaciones financieras mensuales y documentos de incorporación de RRHH. Nadie actualizó la política.

---

## 🔍 Find the vulnerability / Encuentra la vulnerabilidad

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontRead",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::marketing-assets-prod/static/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/EXAMPLEID"
        }
      }
    },
    {
      "Sid": "LegacyPublicAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::marketing-assets-prod/*"
    },
    {
      "Sid": "DenyDelete",
      "Effect": "Deny",
      "Principal": "*",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteBucket"
      ],
      "Resource": [
        "arn:aws:s3:::marketing-assets-prod",
        "arn:aws:s3:::marketing-assets-prod/*"
      ]
    }
  ]
}
```

---

## 💡 Hint / Pista (optional)

Pay attention to the resource scope in each statement. One statement is doing exactly what it's supposed to do. Another one is doing much more than intended.

[ES] Presta atención al alcance del recurso en cada declaración. Una está haciendo exactamente lo que debe. Otra está haciendo mucho más de lo previsto.

---

## 📊 Difficulty / Dificultad

- **Apprentice — Common, well-documented vuln** ← This one
- Hunter — Requires chaining or context
- Archmage — Subtle, logic-based, or multi-vector

The flaw is visible once you understand what each statement covers. No trick — just read it carefully.

---
🔒 Solution in: [solutions/cloud/solution-001.md](../../solutions/cloud/solution-001.md)
