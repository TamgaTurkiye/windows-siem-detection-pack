# Windows Core SIEM Detections — Production Candidate v1.0

Bu paket, şirket içi SOC/detection engineering süreçleri için hazırlanmış Windows odaklı SIEM algılama kuralları başlangıç setidir.

## Kapsam

- Çoklu başarısız Windows oturum açma denemeleri
- Yeni Windows kullanıcı hesabı oluşturma
- Ayrıcalıklı/security-enabled gruba üye ekleme
- Şüpheli PowerShell etkinliği için katmanlı hunting/alert modeli
- Yeni Windows servis kurulumu

## Önemli kullanım sınırı

Bu paket plug-and-play değildir. Kopyala-yapıştır production alert olarak kullanılmamalıdır. Her kural önce log coverage, SIEM field mapping, baseline, false-positive analizi, audit-mode testi ve SOC onayından geçmelidir.

## Klasör yapısı

```text
docs/      Ana doküman, production gate ve runbook'lar
sigma/     Ayrı Sigma/YAML taslakları
samples/   Sentetik log örnekleri için ayrılmış alan
issues/    Açık kaynak/harici katkı issue taslakları
```

## Production gate

Alert modu açılmadan önce `docs/windows-core-siem-detections-production-candidate.md` içindeki **İç Onay ve Production Gate** bölümü tamamlanmalıdır.

## Güvenlik sınırı

Gerçek IP, hostname, kullanıcı adı, servis hesabı, access token, secret, müşteri verisi veya gerçek olay logu paylaşmayın.
