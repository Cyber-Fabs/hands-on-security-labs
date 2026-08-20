# 🛡️ Cybersecurity Labs

Hands-on, self-directed cybersecurity labs built and documented from scratch covering both **offensive** (attacking) and **defensive** (investigating) perspectives. Each lab was performed in an isolated Kali Linux environment using only local, non-production systems and self-generated test data.

These labs go beyond coursework: they're independent projects where I build the scenario, execute the technique, and document the full process, the way a working security practitioner would.

---

## 🧪 Labs in this Repository

### 🔴 [SQL Injection Lab](./sql-injection/) — *Offensive*
Built a custom PHP + MariaDB web application, reproduced a SQL injection vulnerability, analysed the root cause at the query level, and remediated it with prepared statements — then verified the fix.

```
Build → Exploit → Root-cause → Fix → Verify
Skills: PHP/MariaDB · SQL injection · prepared statements · least privilege
```

### 🔵 [Web Attack Investigation](./web-attack-investigation/) — *Defensive*
Investigated a web attack in Apache logs following the full 9-step incident lifecycle — preserving evidence, tracing the attacker's activity, identifying attack phases, determining root cause, and documenting the findings.

```
Preserve → Identify → Collect → Correlate → Analyze →
Root Cause → Remediate → Verify → Document
Skills: log analysis · grep/awk · evidence handling · incident documentation
```

---

## 🔗 The Two Labs Together

These labs are deliberately paired — they show the **same attack from both sides**:

```
SQL Injection Lab        →  I PERFORMED the attack      (attacker's view)
Web Attack Investigation →  I INVESTIGATED the attack   (defender's view)
```

Understanding an attack offensively is what makes it recognisable defensively. Together they demonstrate the full picture: how an attack works, and how a SOC analyst detects and responds to it.

---

## 🧰 Tools & Skills

```
Environment   : Kali Linux · Apache · MariaDB · PHP
Offensive     : SQL injection · reconnaissance · payload crafting
Defensive     : log analysis · evidence preservation · incident response
Command-line  : grep · awk · sort · uniq · curl · sha256sum
Frameworks    : incident investigation lifecycle · OWASP · NIST
```

---

## 👤 About

Aspiring SOC & GRC analyst with a background in bioinformatics research, focused on healthcare and pharmaceutical security. These labs are part of an ongoing, hands-on cybersecurity portfolio.


---

> ⚠️ **Ethics:** All labs were performed in isolated, local environments on systems I own, using non-sensitive test data. Offensive techniques are studied strictly to strengthen defensive capability.
