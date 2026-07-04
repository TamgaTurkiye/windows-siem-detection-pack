# Windows SIEM Detection Pack

Production-minded Windows SIEM detection starter pack for SOC, blue team, and detection engineering teams.

This repository provides Windows-focused detection candidates, Sigma-style rules, SOC triage notes, validation guidance, and deployment readiness checks for common defensive monitoring use cases.

> This project is not a plug-and-play SIEM rule set.
> Each organization must validate logs, field mappings, thresholds, false-positive patterns, privileged group inventories, service account allowlists, and SOC workflows before enabling production alerts.

---

## Scope

This pack currently focuses on Windows-based SIEM detections for:

* Multiple failed Windows logon attempts
* New Windows user account creation
* User added to privileged or security-enabled groups
* Suspicious PowerShell activity using a layered hunting/alerting model
* New Windows service installation

Covered log sources include:

* Windows Security Event Log
* Windows System Event Log
* PowerShell Operational / Script Block Logging

---

## Repository Structure

```text
docs/      Main documentation, production gate, validation matrix, and SOC runbooks
sigma/     Sigma-style detection candidates and correlation drafts
samples/   Space reserved for synthetic / lab-safe sample logs
issues/    Contribution and improvement issue templates
```

---

## Important Usage Boundary

These detections are designed as **production-minded candidates**, not universal production-ready alerts.

Before enabling alert mode, each rule should go through:

1. Log source validation
2. Windows audit policy validation
3. SIEM field mapping validation
4. 7–14 day baseline measurement
5. Threshold tuning
6. False-positive review
7. Service account / admin account allowlisting
8. Privileged group inventory validation
9. Audit or hunting mode testing
10. SOC review and approval

Recommended deployment flow:

```text
Hunting / Audit Mode → Baseline → Tuning → Review → Controlled Alerting
```

---

## Included Detection Areas

### 1. Multiple Failed Logons

Detects repeated failed Windows logon attempts using Event ID `4625`.

Includes separate detection logic for:

* Multiple failures against the same account
* Multiple failures from the same source against different accounts

This helps distinguish brute-force-style behavior from password-spraying-style behavior.

---

### 2. New Windows User Account Created

Detects new Windows account creation using Event ID `4720`.

Designed to help identify unexpected account creation, especially when correlated with:

* Account enablement
* Privileged group membership changes
* Activity outside approved change windows
* Non-standard administrative accounts

---

### 3. Privileged Group Membership Added

Detects users being added to security-enabled local, global, or universal groups.

Relevant Event IDs:

* `4732` — Member added to a security-enabled local group
* `4728` — Member added to a security-enabled global group
* `4756` — Member added to a security-enabled universal group

Production use should prefer a validated privileged group inventory and, where possible, SID/RID-based matching instead of relying only on localized group names.

---

### 4. Suspicious PowerShell Activity

Provides a layered PowerShell detection model.

The pack separates:

* Low-confidence indicators for hunting
* Context enrichment
* Higher-confidence alert candidates

PowerShell detections should not rely on simple keyword matching alone. Script Block Logging must be enabled where applicable, and organizations should consider privacy and sensitive log content handling.

---

### 5. New Windows Service Installed

Detects new Windows service installation using:

* `4697` from Windows Security logs
* `7045` from Windows System / Service Control Manager logs

Both sources should be validated locally. SIEM ingestion and parsing behavior may vary by environment.

---

## SIEM Platform Notes

Sigma is used as a common open detection format, but each SIEM platform may require different field names, query syntax, and correlation logic.

Target platforms may include:

* Microsoft Sentinel / KQL
* Splunk SPL
* Elastic
* Wazuh
* QRadar
* Other SIEM or log analytics platforms

Sigma correlation support depends on the selected backend, backend version, conversion tooling, and target SIEM capability. Always validate converted rules before use.

---

## Security and Privacy Boundary

Do not submit or commit:

* Real IP addresses
* Real hostnames
* Real usernames
* Real domain names
* Real customer data
* Real incident logs
* Access tokens
* API keys
* Passwords
* Secrets
* Malware samples
* Offensive payloads
* Exploitation steps
* Evasion or bypass instructions

Only synthetic, anonymized, or lab-safe examples should be used.

---

## Contribution Guidelines

Allowed contributions include:

* Source verification
* Field mapping improvements
* Sigma syntax improvements
* False-positive notes
* SOC runbook improvements
* Platform mapping notes
* Synthetic sample logs
* Defensive lab validation notes
* Documentation improvements

Not allowed:

* Exploit steps
* Credential theft or abuse
* Malware execution guidance
* Real victim logs
* Real credentials, tokens, or secrets
* Offensive payloads
* Evasion or bypass techniques
* Target-specific attack guidance

---

## Status

Current status: **v1.0 production-minded candidate pack**

This repository is intended to evolve through defensive review, validation, and community contribution.

Future improvement areas may include:

* Windows 4688 process creation enrichment
* Sysmon enrichment
* Microsoft Defender for Endpoint enrichment
* Microsoft Sentinel rule translations
* Splunk SPL translations
* Elastic detection rule validation
* Wazuh mapping notes
* Linux authentication detection pack
* Cloud IAM detection pack
* Synthetic log test corpus

---

## License

This project is released under the MIT License.

---

# Türkçe Açıklama

Bu repo, SOC, blue team ve detection engineering ekipleri için hazırlanmış Windows odaklı SIEM algılama kuralları başlangıç paketidir.

Bu paket doğrudan üretime alınacak hazır alarm kuralları sunmaz. Her kurum; log kaynaklarını, Windows denetim politikalarını, SIEM alan eşlemelerini, eşik değerlerini, yanlış pozitif kaynaklarını, ayrıcalıklı grup envanterini, servis hesabı istisnalarını ve SOC iş akışını kendi ortamına göre doğrulamalıdır.

Önerilen kullanım akışı:

```text
Hunting / Audit Mode → Baseline → Tuning → Review → Controlled Alerting
```

## Kapsam

Bu paket şu Windows odaklı algılama başlıklarını içerir:

* Çoklu başarısız Windows oturum açma denemeleri
* Yeni Windows kullanıcı hesabı oluşturma
* Ayrıcalıklı veya security-enabled gruba kullanıcı eklenmesi
* Şüpheli PowerShell etkinliği için katmanlı hunting/alert modeli
* Yeni Windows servis kurulumu

## Önemli Not

Bu repo savunma amaçlıdır. Gerçek IP, hostname, kullanıcı adı, domain adı, müşteri verisi, olay müdahale kaydı, access token, API key, parola, secret, zararlı yazılım örneği veya saldırı adımı paylaşılmamalıdır.

Katkılar yalnızca güvenli, savunma odaklı, anonimleştirilmiş veya sentetik örnekler üzerinden yapılmalıdır.
