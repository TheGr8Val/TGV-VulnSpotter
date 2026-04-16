---
id: VS-003
challenge: challenges/network/challenge-001.md
---

# Solution VS-003 — East-West Exposure

## ✅ Vulnerability / Vulnerabilidad

**Overly Permissive Internal Firewall Rule Enabling Lateral Movement — CWE-284 (Improper Access Control)**

Rule R006 grants unrestricted TCP access from the DMZ to the entire APP zone on all ports. Combined with the fact that APP has direct access to DATA, this creates an unobstructed east-west path from a compromised DMZ host to the database tier.

---

## 🔎 Explanation / Explicación

The intended traffic flows are:

```
Internet → DMZ (R001, R002)
DMZ → APP on 8080/TCP only (R003)
APP → DATA on 5432/TCP (R004)
APP → DATA on 6379/TCP (R005)
```

This is a reasonable three-tier architecture. But R006 breaks it:

```
R006 | DMZ | APP (10.0.2.0/24) | 0-65535/TCP | ALLOW | DMZ to APP unrestricted (temp - migration)
```

R006 allows **all TCP ports** from the DMZ zone to the entire APP subnet. R003 already covers the legitimate need (port 8080). R006 was added as a temporary migration convenience and never removed.

**The attack chain from a compromised DMZ host:**

1. Attacker gains foothold in DMZ (e.g., via RCE in the reverse proxy — see VS-001).
2. R006 allows the attacker to reach any port on any host in APP (10.0.2.0/24).
3. From APP, R004 and R005 grant access to the DATA zone on database ports.
4. Attacker can now interact directly with PostgreSQL (5432) or Redis (6379) in DATA.

Additionally, R007 allows SSH from `10.0.2.15/32` to DATA. If the attacker can pivot to or spoof traffic from that IP in APP, they reach SSH in the DATA zone as well.

The security team reviewed north-south traffic (internet → DMZ) and found it clean. East-west traffic within the production network was not evaluated — a common gap.

[ES] La regla R006 permite todo el tráfico TCP desde la zona DMZ hacia toda la subred APP en todos los puertos. Dado que APP tiene acceso directo a DATA a través de R004 y R005, esto crea un camino este-oeste sin obstáculos desde un host DMZ comprometido hasta la capa de base de datos.

Un atacante con acceso inicial en DMZ puede usar R006 para alcanzar cualquier puerto en APP, y desde allí usar R004/R005 para interactuar directamente con PostgreSQL y Redis en DATA.

---

## 💥 Impact / Impacto

From a single compromised host in the DMZ, an attacker can:

- Reach all services on all APP hosts on any TCP port (SSH, internal APIs, debug endpoints, metrics, admin interfaces)
- Query or modify the PostgreSQL database directly
- Read or overwrite Redis cache (session tokens, application data)
- Potentially reach SSH in DATA (R007) if they can pivot through or spoof `10.0.2.15`
- Move laterally across the entire production environment — the three-tier segmentation provides no real isolation

---

## 🛡️ Remediation / Remediación

**Step 1:** Delete R006. It was a temporary migration rule. "Temp" rules that persist are one of the most consistent sources of security debt in production environments.

**Step 2:** Verify R003 (DMZ → APP on 8080/TCP) covers all legitimate application traffic. If additional ports are needed, add scoped rules with specific source IPs and destination ports.

**Step 3:** Re-evaluate R007. SSH access from a broad app-tier host to DATA is a high-privilege path. Consider replacing it with a dedicated bastion host with MFA, and restrict the source to that bastion's IP only.

**Step 4:** Implement host-based firewall rules (`iptables`, `nftables`, or security groups if cloud-hosted) as a secondary control. Network perimeter rules alone are insufficient if lateral movement is possible within the same zone.

**Corrected ruleset:**

```
R001 | ANY          | DMZ               | 443/TCP      | ALLOW  | Inbound HTTPS to reverse proxy
R002 | ANY          | DMZ               | 80/TCP       | ALLOW  | Inbound HTTP (redirect only)
R003 | DMZ          | APP (10.0.2.0/24) | 8080/TCP     | ALLOW  | Proxy to app servers
R004 | APP          | DATA (10.0.3.0/24)| 5432/TCP     | ALLOW  | App to PostgreSQL
R005 | APP          | DATA (10.0.3.0/24)| 6379/TCP     | ALLOW  | App to Redis cache
R006 | 10.0.1.10/32 | 10.0.2.15/32     | 22/TCP       | ALLOW  | Bastion SSH to ops host only
R007 | 10.0.2.15/32 | DATA (10.0.3.0/24)| 22/TCP      | ALLOW  | Ops bastion to DATA zone SSH
R008 | ANY          | ANY               | ANY          | DENY   | Default deny all
```

---

## 📚 References / Referencias

- [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)
- [MITRE ATT&CK T1021: Remote Services (Lateral Movement)](https://attack.mitre.org/techniques/T1021/)
- [NIST SP 800-125B: Secure Virtual Network Configuration](https://csrc.nist.gov/publications/detail/sp/800-125b/final)
- [CISA: Network Segmentation Best Practices](https://www.cisa.gov/sites/default/files/publications/layering-network-security-segmentation_infographic_508.pdf)
