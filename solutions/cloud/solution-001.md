---
id: VS-002
challenge: challenges/cloud/challenge-001.md
---

# Solution VS-002 — The Open Bucket

## ✅ Vulnerability / Vulnerabilidad

**AWS S3 Public Access Misconfiguration — CWE-732 (Incorrect Permission Assignment for Critical Resource)**

A legacy bucket policy statement grants `s3:GetObject` to `Principal: "*"` (the entire internet) on all objects in the bucket, not just the intended static assets subdirectory.

---

## 🔎 Explanation / Explicación

The policy contains three statements. Two are correct. One is not.

**Statement 1 — `AllowCloudFrontRead`** (correct)
Allows CloudFront to read objects, scoped to a specific distribution ARN and limited to the `static/*` prefix. This is the right pattern for serving public CDN assets.

**Statement 3 — `DenyDelete`** (correct)
Denies deletion to all principals. Protective guard, no issues.

**Statement 2 — `LegacyPublicAccess`** (the problem)
```json
{
  "Sid": "LegacyPublicAccess",
  "Effect": "Allow",
  "Principal": "*",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::marketing-assets-prod/*"
}
```

- `Principal: "*"` — anyone on the internet, unauthenticated
- `Resource: "arn:aws:s3:::marketing-assets-prod/*"` — the wildcard `/*` covers **every object in the bucket**, not just `static/*`

This means financial exports, HR documents, and any other file uploaded to the bucket — regardless of prefix — is publicly readable by anyone who knows (or guesses) the object key.

S3 object keys are often predictable: `reports/2025-Q4-financials.csv`, `hr/onboarding/new-hire-john.pdf`. Enumeration can be performed via timing attacks, exposed presigned URLs, or application errors that leak key names.

[ES] La política contiene tres declaraciones. Dos son correctas. Una no lo es.

La declaración `LegacyPublicAccess` otorga `s3:GetObject` a `Principal: "*"` (cualquier persona en internet) sobre todos los objetos del bucket con el wildcard `/*`. Esto expone públicamente cualquier archivo subido al bucket — no solo los assets estáticos en `static/*` — incluyendo exportaciones financieras y documentos de RRHH.

Las claves de objetos S3 suelen ser predecibles: `reports/2025-Q4-financials.csv`, `hr/onboarding/new-hire-john.pdf`. Un atacante puede enumerar o adivinar claves y descargar los archivos directamente.

---

## 💥 Impact / Impacto

- Unauthorized public access to all objects in `marketing-assets-prod/`
- Exposure of sensitive files uploaded after the original policy was written (financial reports, PII, internal documents)
- No authentication required — a direct `GET` request to the object URL returns the file
- Potential regulatory violations (GDPR, HIPAA, SOC 2) depending on data classification

---

## 🛡️ Remediation / Remediación

**Step 1:** Remove the `LegacyPublicAccess` statement entirely.

**Step 2:** Ensure the CloudFront statement covers the correct scope for public assets. If `static/*` is the only content that should be publicly accessible, that scoping is correct.

**Step 3:** Enable S3 Block Public Access at the bucket level as a defense-in-depth control. This prevents any future policy or ACL from accidentally re-enabling public access.

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

**Step 4:** Audit access logs for the bucket to determine if sensitive objects were accessed before remediation.

---

## 📚 References / Referencias

- [CWE-732: Incorrect Permission Assignment for Critical Resource](https://cwe.mitre.org/data/definitions/732.html)
- [AWS: Blocking public access to S3 storage](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [MITRE ATT&CK T1530: Data from Cloud Storage](https://attack.mitre.org/techniques/T1530/)
- [OWASP Cloud Security: Storage Misconfiguration](https://owasp.org/www-project-cloud-security/)
