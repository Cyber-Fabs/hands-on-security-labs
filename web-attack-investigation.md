# 🔍 Web Attack Investigation — Apache Log Analysis

![Kali](https://img.shields.io/badge/Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![Apache](https://img.shields.io/badge/Apache%20Logs-D22128?style=flat-square&logo=apache&logoColor=white)
![SOC](https://img.shields.io/badge/SOC-Log%20Analysis-1F6FEB?style=flat-square)
![NIST](https://img.shields.io/badge/NIST-Detection%20%26%20Analysis-000000?style=flat-square)
![Outcome](https://img.shields.io/badge/Attack-Attempted%20(Unsuccessful)-2EA043?style=flat-square)

**I investigated a web attack in Apache logs from start to finish** — following the full incident investigation lifecycle on my own Kali Linux server. I preserved the evidence, traced the attacker's activity, identified the attack phases, determined root cause, and documented the findings.

| | |
|---|---|
| **Evidence** | Apache `access.log` (14 requests) |
| **Attack** | Reconnaissance scan → SQL injection attempt |
| **Tool used** | `curl` (automated, not a browser) |
| **Target** | `/products?id=` endpoint |
| **Outcome** | ⚠️ Attempted — did **not** succeed (all 404s) |
| **Method** | 9-step investigation lifecycle |

> ⚠️ Conducted entirely on my own local server (`127.0.0.1`) with self-generated traffic. Ethical, isolated lab.

---

## 🔄 The Investigation Lifecycle

```
Preserve → Identify → Collect → Correlate → Analyze →
Root Cause → Remediate → Verify → Document
```

---

## 1️⃣ Preserve

Protected the original evidence before touching it — copied the log to a working file and recorded its hash to prove integrity (chain of custody).

```bash
sudo cp /var/log/apache2/access.log ./evidence.log
sha256sum evidence.log | tee evidence_hash.txt
```
```
SHA-256: e2d48952e2b4e10c67484d0e2b8a74ceda726da38d93b3041a09992ee9863933
```

---

## 2️⃣ Identify

Sized up the evidence — how much, and over what time window.

```bash
wc -l evidence.log        # → 14 requests
head -1 evidence.log      # → first: 11:09:45 (normal)
tail -1 evidence.log      # → last:  20:32:34 (attack)
```
```
14 requests · 19 Aug 2026 · 11:09 → 20:32 (~9.5 hrs)
Starts normal, ends with a SQL injection probe.
```

---

## 3️⃣ Collect

Aggregated the key data points — who, and what outcomes.

```bash
awk '{print $1}' evidence.log | sort | uniq -c | sort -rn   # source IPs
awk '{print $9}' evidence.log | sort | uniq -c | sort -rn   # status codes
```
```
Source IPs:      14  from a single source
Status codes:    13 × 404   (Not Found)
                  1 × 200   (OK)
```

> 🚩 **13 of 14 requests failed (404).** Normal users don't generate floods of 404s — that ratio is the signature of **scanning**.

---

## 4️⃣ Correlate

Pulled the full timeline of the suspicious source into one view.

```bash
grep "127.0.0.1" evidence.log
```

This surfaced the complete sequence of activity in time order — setting up the phase analysis below.

---

## 5️⃣ Analyze

Read the timeline as a story. Three distinct phases emerged:

```
📖 PHASE 1 — Normal          11:09   GET /              200
   → legitimate homepage visit (the baseline)

📖 PHASE 2 — Reconnaissance  20:26   10 requests, ALL 404
   /admin  /administrator  /wp-login.php  /phpmyadmin
   /.env   /backup.zip     /config.php    /db  /test.php ...
   → scanning for sensitive pages — all in the SAME SECOND

📖 PHASE 3 — Exploitation    20:32   GET /products?id=1'   404
   → SQL injection probe (the ' tests if input breaks the query)
```

**The escalation:** `Normal → Recon → Exploitation` — the anatomy of a real web attack, visible line by line.

---

## 6️⃣ Determine Root Cause

The attacker targeted the `/products` endpoint's `id` parameter with a SQL injection probe (`id=1'`) — testing whether user input flows unsanitised into a database query.

```bash
grep "products" evidence.log | awk '{print $9}'
# → 404 404
```

> **Both attempts returned 404 — the endpoint didn't exist, so the injection did not succeed.** Reported honestly: the attack was **attempted**, not successful. No vulnerable application was present and no data was exposed.

---

## 7️⃣ Remediate

Recommended defences — tied directly to the attack observed:

```
Immediate:
  → Block the source IP at the firewall / WAF
  → Alert on rapid-404 bursts (10 requests/second = automated scan)

Root-cause fixes (for any real /products endpoint):
  → Prepared statements (parameterised queries) — stops SQLi at source
  → Input validation (id must be numeric; reject id=1' )
  → Least-privilege database account

Defence in depth:
  → Web Application Firewall to block scanner/injection signatures
  → Rate limiting per source IP
```

---

## 8️⃣ Verify

Confirmed the conclusion by extracting the attacker's user-agent.

```bash
grep "products" evidence.log | awk -F'"' '{print $6}'
# → curl/8.18.0
```

> **`curl` is a command-line tool, not a browser.** Three independent indicators agree: the 13:1 404 ratio, the same-second timestamps, and the curl user-agent — all confirming deliberate, automated activity, not a real user.

---

## 9️⃣ Document — Incident Summary

```
WHO:    Automated client (curl/8.18.0) from a single source
WHAT:   Two-phase attack — reconnaissance scan for sensitive
        pages, then a SQL injection attempt on /products?id=1'
WHEN:   19 Aug 2026 — recon 20:26, injection 20:32
WHERE:  Apache web server; exploitation targeted /products
WHY:    Probing for an unsanitised input (SQL injection)
IMPACT: None — all requests returned 404. Attempted but
        UNSUCCESSFUL. No data exposed, no compromise.
ACTION: Block IP · alert on scan patterns · enforce prepared
        statements + input validation on any real endpoint
```

---

## 🎯 Key Takeaways

- **A flood of 404s from one source = scanning**, not browsing — the single clearest tell in web logs.
- **Same-second timestamps reveal automation** — no human requests 10 pages in one second.
- **User-agent fingerprints the tool** — `curl`/`sqlmap` ≠ a real browser.
- **Attempted ≠ successful.** Checking the status code (404 vs 200) is what separates a real breach from a failed probe — and reporting that honestly is what makes an analyst credible.
- **This is the defender's view of the SQL injection I performed offensively** in my SQLi lab — same attack, both sides.

---

## 🧰 Skills Demonstrated

```
✅ Evidence preservation & integrity hashing (chain of custody)
✅ Log triage with grep, awk, sort, uniq
✅ Identifying attack phases (recon → exploitation)
✅ Recognising automated scanning (404 ratio, timestamps)
✅ Fingerprinting attacker tools via user-agent
✅ Root-cause analysis & honest impact assessment
✅ Mapping findings to remediation
✅ Writing a clear incident summary (5 W's)
✅ Full 9-step incident investigation lifecycle
```

---

## 📎 Evidence

| Screenshot | Shows |
|-----------|-------|
| `01_apache_log_discovery.png` | Locating the Apache log; a request appearing in it in real time |
| `02_preserve_identify_collect.png` | Preserve (hash), Identify (count/scope), Collect (IPs + status codes) |
| `03_analyze_verify.png` | Isolating the injection attempts, confirming 404 outcome, and the curl user-agent |

---

**References:** [NIST SP 800-61 Incident Handling](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final) · [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) · [Apache Log Files](https://httpd.apache.org/docs/current/logs.html)
