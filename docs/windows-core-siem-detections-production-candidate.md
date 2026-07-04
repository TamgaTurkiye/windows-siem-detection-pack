# Internal SOC Detection Pack
# Windows Core SIEM Detections: Production Candidate v1.0

---

## 0. Kapsam ve Güvenlik Bildirimi

Bu belge, şirket içi SOC ve detection engineering kullanımı için hazırlanmış **savunma odaklı Windows SIEM algılama kuralları paketidir.** Açık kaynak olarak paylaşılabilir; ancak şirket ortamı, hesap adları, IP aralıkları, asset isimleri, allowlist'ler, change-ticket örnekleri ve gerçek loglar dokümandan çıkarılmalıdır.

**Bu paket production candidate — yani üretime alınmaya aday bir detection backlog'udur.** Şirket ortamında doğrudan kopyala-yapıştır etkinleştirilecek "plug-and-play" bir çözüm değildir. Her kural, kurumun log toplama mimarisi, SIEM field mapping'i, asset sınıflandırması, IAM/change-management süreci ve SOC triage akışı doğrulandıktan sonra alert moduna alınmalıdır.

Her kuralın gerçek bir ortamda etkinleştirilmeden önce şu adımlardan geçmesi gerekir:

- Log kaynağı doğrulaması
- SIEM alan eşleme (field mapping) doğrulaması
- Ortam bazlı baseline ölçümü (en az 7–14 gün)
- Eşik değeri (threshold) ayarlaması
- Yanlış pozitif analizi ve istisna listesi oluşturma
- SOC triage iş akışına entegrasyon
- Sessiz/denetim (audit) modunda test
- SOC ekip onayı

**Güvenlik sınırları:**
- Bu belgede saldırı adımı, istismar kodu, kimlik bilgisi kötüye kullanımı, zararlı yazılım çalıştırma, atlatma/kaçınma talimatı veya hedefe yönelik saldırı rehberliği bulunmaz.
- Tüm içerik savunma odaklıdır.
- Laboratuvar örnekleri zararsız ve sentetiktir.
- Alan adı, Event ID davranışı veya Sigma söz dizimi konusunda belirsizlik varsa "doğrulama gerekir" (needs validation) olarak işaretlenmiştir.


**Şirket içi üretim varsayımları:**
- Kural sahibi: Detection Engineering / SOC Content Owner.
- Operasyon sahibi: SOC L1/L2 triage ekibi.
- Log sahibi: Windows Platform / Infrastructure ekibi.
- Değişiklik onayı: Change Management / CAB süreci.
- İlk deployment modu: varsayılan olarak `hunting` veya `audit`; yalnızca düşük gürültülü ve yüksek değerli kurallar filtrelendikten sonra `alert`.
- Şirket içi değerler, host isimleri, kullanıcı adları, IP blokları, servis hesapları, allowlist'ler ve gerçek log örnekleri açık kaynak sürümünde yer almamalıdır.
- Kural başarısı yalnızca "çalışıyor" diye ölçülmez; signal-to-noise oranı, triage süresi, yanlış pozitif oranı ve müdahale etkisi birlikte ölçülür.

---

## 1. Bu Paketi Nasıl Kullanmalı

| Kullanım Amacı | Açıklama |
|---|---|
| **Öğrenme** | SIEM kural yazımı, Sigma formatı ve Windows güvenlik loglarını öğrenmek için |
| **Algılama Mühendisliği Pratiği** | Sigma correlation, threshold ve katmanlı algılama mantığı uygulamak için |
| **SOC Backlog Oluşturma** | Yeni algılama kuralları için iş listesi (backlog) oluşturmak için |
| **SIEM Ayarlama** | Mevcut kuralları karşılaştırma ve ayarlama referansı olarak |
| **Açık Kaynak Paylaşım** | Şirket içi hassas bilgiler çıkarıldıktan sonra savunma topluluğuyla paylaşılabilecek başlangıç paketi olarak |
| **Doğrudan Üretime Alma** | ❌ **Bu amaçla kullanılmamalıdır.** Owner ataması, field mapping, baseline, audit-mode testi, change-management entegrasyonu ve SOC onayı olmadan alert moduna alınmamalıdır. |

---

## 2. Dağıtım Hazırlık Kontrol Listesi

Herhangi bir kuralı üretim ortamında etkinleştirmeden önce aşağıdaki kontrol listesini tamamlayın:

| # | Kontrol Maddesi | Durum |
|---|---|---|
| 1 | Gerekli loglar toplanıyor mu? (Security, System, PowerShell Operational) | ☐ |
| 2 | Denetim politikaları (Audit Policy) etkinleştirildi mi? (Logon/Logoff, Account Management, System Integrity, Object Access vb.) | ☐ |
| 3 | PowerShell Script Block Logging etkinleştirildi mi? (Grup İlkesi ile) | ☐ |
| 4 | SIEM'deki alan eşlemeleri (field mapping) doğrulandı mı? (TargetUserName, SourceNetworkAddress, ServiceFileName vb.) | ☐ |
| 5 | Kurumun ayrıcalıklı grup envanteri tanımlandı mı? (SID/RID veya grup adı listesi) | ☐ |
| 6 | Bilinen yönetici, provisioning ve servis hesapları belgelendi mi? | ☐ |
| 7 | Yazılım dağıtım araçları istisna listesine alındı mı? (SCCM, Intune, WSUS vb.) | ☐ |
| 8 | Yüksek değerli varlıklar (high-value assets) etiketlendi mi? (DC'ler, Tier-0 sistemler) | ☐ |
| 9 | Baseline ölçümü yapıldı mı? (En az 7–14 gün sessiz modda) | ☐ |
| 10 | Eşik değerleri ortama göre ayarlandı mı? | ☐ |
| 11 | SOC runbook'u hazırlandı mı? | ☐ |
| 12 | Uyarı önem dereceleri triage iş akışına eşlendi mi? | ☐ |
| 13 | Yanlış pozitif örnekleri belgelendi mi? | ☐ |
| 14 | Gizlilik / log içeriği endişeleri (örn. Script Block içinde hassas veri) gözden geçirildi mi? | ☐ |
| 15 | Değişiklik yönetimi (change management) entegrasyonu mevcut mu? | ☐ |
| 16 | Kural önce audit/sessiz modda test edildi mi? | ☐ |
| 17 | Kural için owner, reviewer ve rollback sorumlusu atandı mı? | ☐ |
| 18 | Uyarı üretirse iş etkisi ve otomatik aksiyon riski değerlendirildi mi? | ☐ |
| 19 | Açık kaynak sürümünde şirket içi IP, hostname, kullanıcı, servis hesabı ve gerçek log bulunmadığı doğrulandı mı? | ☐ |

---

## 3. Doğrulama Matrisi

| Algılama | Gerekli Log Kaynağı | Gerekli Event ID | Gerekli Denetim Politikası | Kritik Alanlar | İsteğe Bağlı Zenginleştirme | Beklenen FP Seviyesi | Önerilen İlk Önem | Önerilen İlk Mod | Doğrudan Üretime Alınmamasının Ana Nedeni | Üretime Hazır Hale Getiren |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 — Çoklu Başarısız Giriş | Security | 4625 | Audit Logon (Success+Failure) | TargetUserName, LogonType, SourceNetworkAddress | 4624 (başarılı giriş), 4740 (hesap kilitleme), GeoIP | Yüksek | Medium | Hunting / Audit | Eşik değeri ortama özgüdür; NAT/VPN/proxy yanlış pozitif üretir | Baseline sonrası eşik ayarı, servis hesabı istisnaları, IP allowlist |
| 2 — Yeni Hesap Oluşturma | Security | 4720 | Audit User Account Management | SubjectUserName, TargetUserName, TargetSid | 4722, 4732/4728/4756, değişiklik yönetimi | Orta | Medium | Audit | Provisioning araçları çok sayıda meşru olay üretir | Provisioning allowlist, mesai dışı yükseltme, DC bağlamı |
| 3 — Ayrıcalıklı Gruba Ekleme | Security | 4732, 4728, 4756 | Audit Security Group Management | TargetUserName, MemberSid, SubjectUserName | Grup SID/RID eşlemesi, değişiklik yönetimi | Düşük-Orta | High | Alert (filtrelenmiş) | Yerelleştirilmiş grup adları ve yeniden adlandırma sorunları | SID tabanlı eşleşme, ayrıcalıklı grup envanteri |
| 4 — Şüpheli PowerShell | PowerShell Operational | 4104 (Script Block) | Script Block Logging (GPO) | ScriptBlockText, Path, UserName | Sysmon, EDR, Process Creation 4688, AMSI | Çok Yüksek | Low (Katman 1) | Hunting | Anahtar kelime eşleşmesi tek başına çok gürültülü | Katmanlı model, baseline, parent process bağlamı |
| 5 — Yeni Servis Kurulumu | Security + System | 4697, 7045 | Audit Security System Extension | ServiceName, ServiceFileName, SubjectUserName | Değişiklik yönetimi, dosya hash, imza durumu | Orta-Yüksek | Medium | Audit | Yazılım dağıtımı çok sayıda meşru olay üretir | Yazılım dağıtım allowlist, yol bazlı filtreleme, DC bağlamı |

---

## 4. Algılama 1 — Kısa Sürede Çoklu Başarısız Oturum Açma

### 4.1 Kural Kartı

| Alan | Açıklama |
|---|---|
| **Kural Başlığı** | Kısa Sürede Çoklu Başarısız Oturum Açma |
| **Düz Dille Açıklama** | Aynı hedef hesaba veya aynı kaynak IP'den birden fazla hesaba kısa sürede yapılan çok sayıda başarısız oturum açma denemesini tespit eder. İki ayrı algılama varyantı içerir: (A) aynı hesaba yönelik kaba kuvvet ve (B) aynı kaynaktan farklı hesaplara yönelik parola püskürtme sinyali. |
| **Ana Log Kaynağı** | Windows Security Event Log |
| **İlgili Event ID** | 4625 — An account failed to log on |
| **Destekleyici Event ID'ler** | 4624 (başarılı giriş — korelasyon), 4740 (hesap kilitleme — bağlam) |
| **Önerilen Önem Derecesi** | **Medium** — Başarılı giriş korelasyonu veya yüksek değerli varlık bağlamında **High** |

**Logon Type Önemi:**

| LogonType | Anlamı | Algılama Bağlamı |
|---|---|---|
| 2 | Etkileşimli (Interactive) | Fiziksel veya konsol girişi |
| 3 | Ağ (Network) | SMB, paylaşım erişimi — en yaygın |
| 7 | Kilit açma (Unlock) | Ekran kilidi — genellikle düşük risk |
| 10 | Uzak Masaüstü (RemoteInteractive) | RDP — yüksek ilgi |
| 11 | Önbellek (CachedInteractive) | Çevrimdışı DC — farklı bağlam |

**Alan Eşleme Notları:**
- `TargetUserName`: Başarısız girişin hedefi olan hesap adı.
- `TargetDomainName`: Hedef hesabın etki alanı. Gruplama için `TargetUserName + TargetDomainName` birlikte kullanılmalıdır.
- `SourceNetworkAddress` veya `IpAddress`: Kaynak IP adresi. **Bu alanın adı SIEM platformuna göre değişebilir — doğrulama gerekir.**
- `WorkstationName`: Kaynak iş istasyonu adı. **Sınırlama:** İstemci tarafından bildirilen bir değerdir; güvenilir bir tanımlayıcı olarak kullanılmamalıdır.

**NAT / VPN / Proxy Uyarısı:** Paylaşılan IP adreslerinden gelen trafik tek bir kaynak gibi görünür. VPN çıkış noktaları, NAT gateway'leri ve proxy sunucuları nedeniyle kaynak IP bazlı gruplama yanlış pozitif üretebilir. Bu IP aralıkları belgelenmeli ve eşik değerleri buna göre ayarlanmalıdır.

**Servis Hesabı Döngüleri:** Parolası değişmiş ancak eski parola ile kimlik doğrulamaya devam eden servis hesapları sürekli 4625 üretir. Bu hesaplar tespit edilip istisna listesine alınmalıdır.

**Baseline Gereksinimi:** Eşik değeri belirlemeden önce ortamın normal başarısız giriş oranını en az 7–14 gün sessiz modda ölçün.

| Alan | Açıklama |
|---|---|
| **Neden Şüpheli Olabilir?** | Normal kullanıcılar kısa sürede çok sayıda başarısız giriş denemesi yapmaz. Yüksek sayıda deneme, otomatik araç kullanımına veya kaba kuvvet/parola püskürtme saldırısına işaret edebilir. Başarısız denemelerden sonra başarılı giriş olması kritik öneme sahiptir — saldırının başarılı olduğuna işaret edebilir. |
| **Yaygın Yanlış Pozitifler** | Parolası değişmiş servis hesapları; önbelleğe alınmış eski kimlik bilgileri; VPN/proxy arkasından çoklu kullanıcı; parola sıfırlama sürecindeki kullanıcılar; yanlış yapılandırılmış uygulamalar; ağ taraması yapan meşru araçlar; NAT gateway'leri; mobil cihaz senkronizasyon sorunları. |
| **MITRE ATT&CK** | T1110.001 — Brute Force: Password Guessing; T1110.003 — Brute Force: Password Spraying (Tactic: Credential Access) |
| **"Kesin Kanıt Değildir" Notu** | Bu uyarı tek başına saldırı kanıtı değildir. Yanlış yapılandırma, kullanıcı hatası veya altyapı sorunları da bu uyarıyı tetikleyebilir. Bağlam ile değerlendirilmelidir. |

### 4.2 Sigma Kuralları

#### 4.2.A — Temel Kural (Base Rule)

```yaml
# Internal SOC Detection Pack — Algılama 1 — Temel Kural
# Enterprise-ready candidate — yerel doğrulama gerektirir
title: Windows Başarısız Oturum Açma Olayı
name: internal_failed_logon
id: a1b2c3d4-e5f6-7890-abcd-ef1234567801  # Üretimde gerçek UUID ile değiştirin
status: test
description: >
  Windows 4625 başarısız oturum açma olaylarını seçer. Tek başına uyarı
  üretmez; korelasyon kuralları tarafından referans alınır.
references:
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625
author: Internal Detection Engineering
date: 2026/07/04
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4625
  # Makine hesaplarını ($) hariç tut — yanlış pozitif azaltır
  filter_machine_accounts:
    TargetUserName|endswith: '$'
  # Anonim oturum açma denemelerini hariç tut
  filter_anonymous:
    TargetUserName: 'ANONYMOUS LOGON'
  condition: selection and not filter_machine_accounts and not filter_anonymous
# NOT: SubjectUserName|endswith: '$' filtresi de değerlendirilebilir.
# SIEM'inizdeki alan adlarını doğrulayın.
# SourceNetworkAddress veya IpAddress alanı SIEM'e göre farklı olabilir.
```

#### 4.2.B — Korelasyon: Aynı Hesaba Kaba Kuvvet (Brute-Force)

```yaml
# Internal SOC Detection Pack — Algılama 1B — Korelasyon: Kaba Kuvvet
title: Aynı Hesaba Kısa Sürede Çoklu Başarısız Giriş (Kaba Kuvvet Sinyali)
id: a1b2c3d4-e5f6-7890-abcd-ef1234567802  # Üretimde gerçek UUID ile değiştirin
status: test
description: >
  Aynı kullanıcı hesabına kısa sürede çok sayıda başarısız giriş denemesi
  yapılmasını tespit eder. Kaba kuvvet saldırısına işaret edebilir.
  Eşik değeri ortam baseline'ına göre ayarlanmalıdır.
references:
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625
  - https://attack.mitre.org/techniques/T1110/001/
  - https://sigmahq.io/docs/meta/correlations.html
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.credential_access
  - attack.t1110.001
correlation:
  type: event_count
  rules:
    - internal_failed_logon
  group-by:
    - TargetUserName
    - TargetDomainName
  timespan: 5m
  condition:
    gte: 10  # ⚠️ Örnek eşik — ortam baseline'ına göre ayarlayın
falsepositives:
  - Parolası değişmiş servis hesapları
  - Önbelleğe alınmış eski kimlik bilgileri ile bağlanan istemciler
  - Parola sıfırlama sürecindeki kullanıcılar
  - Yanlış yapılandırılmış uygulamalar
level: medium
# NOT: 'timespan' ve eşik değeri ortama özgüdür.
# Yüksek değerli hesaplar (Domain Admin vb.) için daha düşük eşik düşünülebilir.
# 4624 ile korelasyon (başarılı giriş sonrası) ayrı bir temporal kural olarak
# oluşturulabilir — bkz. aşağıdaki not.
```

#### 4.2.C — Korelasyon: Aynı Kaynaktan Farklı Hesaplara (Parola Püskürtme)

```yaml
# Internal SOC Detection Pack — Algılama 1C — Korelasyon: Parola Püskürtme
title: Aynı Kaynaktan Farklı Hesaplara Başarısız Giriş (Parola Püskürtme Sinyali)
id: a1b2c3d4-e5f6-7890-abcd-ef1234567803  # Üretimde gerçek UUID ile değiştirin
status: test
description: >
  Aynı kaynak IP adresinden kısa sürede birden fazla farklı hesaba başarısız
  giriş denemesi yapılmasını tespit eder. Parola püskürtme saldırısına
  işaret edebilir.
references:
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625
  - https://attack.mitre.org/techniques/T1110/003/
  - https://sigmahq.io/docs/meta/correlations.html
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.credential_access
  - attack.t1110.003
correlation:
  type: value_count
  rules:
    - internal_failed_logon
  group-by:
    - SourceNetworkAddress
    # NOT: Bu alan adı SIEM'e göre farklı olabilir.
    # Elastic: source.ip, Splunk: src_ip, Sentinel: IpAddress
    # Doğrulama gerekir.
  timespan: 10m
  condition:
    gte: 5  # ⚠️ Örnek eşik — 10 dakikada 5+ farklı hesap
    field: TargetUserName
falsepositives:
  - NAT gateway veya proxy arkasından gelen meşru kullanıcılar
  - VPN çıkış noktaları
  - Ağ taraması yapan meşru güvenlik araçları
  - Paylaşılan iş istasyonları
level: medium
# NOT: VPN/NAT IP aralıkları belgelenip eşik yükseltilmelidir.
# SourceNetworkAddress alanı bazı SIEM platformlarında
# IpAddress, src_ip veya source.ip olarak eşlenebilir — doğrulama gerekir.
```

**Ek Korelasyon Notu — Başarısız Sonrası Başarılı Giriş:**
Birden fazla 4625 olayının ardından aynı hesap için 4624 (başarılı giriş) olayı gözlenirse, saldırının başarılı olmuş olabileceğine dair güçlü bir sinyal oluşur. Bu korelasyon `temporal` veya `temporal_ordered` tipi ile kurulabilir. Ancak `temporal_ordered` desteği sınırlı backend'lerde mevcuttur; `temporal` tipi daha geniş desteklenir ve pratikte benzer sonuçları verir.

**Ek Korelasyon Notu — 4740 Hesap Kilitleme:**
4625 olaylarıyla birlikte 4740 (hesap kilitleme) olayının gözlenmesi, eşik aşımının gerçekleştiğini doğrular. Mevcut olduğunda bağlam zenginleştirmesi olarak kullanılabilir.

### 4.3 Ayarlama Notları

- Bilinen servis hesaplarını `TargetUserName` istisna listesine ekleyin.
- VPN/NAT gateway IP aralıkları için ayrı, daha yüksek eşikli kurallar oluşturun.
- `LogonType` bazında ayrı kurallar düşünün: RDP (10) için daha düşük eşik, ağ (3) için daha yüksek.
- Baseline ölçümü olmadan eşik değeri belirlemeyin.
- Yüksek değerli hesaplar (Domain Admin, Enterprise Admin) için ayrı, düşük eşikli kurallar oluşturun.
- `SubStatus` alanı başarısızlık nedenini içerir (0xC000006A = yanlış parola, 0xC0000064 = olmayan hesap vb.) ve filtreleme için kullanılabilir.

### 4.4 SOC Runbook

**İlk 5 Soru:**
1. Hedef hesap kim? Servis hesabı mı, kullanıcı hesabı mı, yönetici hesabı mı?
2. Kaynak IP adresi nedir? Kurum içi mi, dış mı? VPN/NAT arkasında mı?
3. Hangi LogonType kullanılmış? (3=ağ, 10=RDP, 2=etkileşimli)
4. Başarısız denemelerin ardından aynı hesap için başarılı giriş (4624) var mı?
5. Hesap kilitlenmiş mi? (4740 olayı)

**Toplanacak Kanıtlar:**
- 4625 olaylarının tam listesi (zaman, kaynak IP, LogonType, SubStatus)
- Aynı zaman diliminde aynı hesap için 4624 olayları
- 4740 hesap kilitleme olayları
- Kaynak IP'nin GeoIP bilgisi ve itibar skoru (threat intelligence)
- Hedef hesabın türü ve ayrıcalık düzeyi

**Korelasyon Kontrolleri:**
- 4624 ile başarılı giriş korelasyonu
- 4740 hesap kilitleme korelasyonu
- Aynı kaynak IP'den farklı hesaplara deneme (parola püskürtme kontrolü)
- Başarılı giriş sonrası olağandışı etkinlik (yeni process, dosya erişimi)

**Yükseltme (Escalate) Koşulları:**
- Başarısız denemeler sonrası başarılı giriş tespit edildi
- Hedef hesap ayrıcalıklı (Domain Admin, Enterprise Admin vb.)
- Kaynak IP dış/bilinmeyen bir adres
- Çok sayıda farklı hesaba deneme (parola püskürtme deseni)
- Mesai dışı saatte yoğun deneme

**Yanlış Pozitif Olarak Kapatma Koşulları:**
- Servis hesabının eski parolası ile döngüsel deneme — hesap sahibi ile doğrulandı
- Bilinen VPN/NAT gateway IP aralığından meşru kullanıcı trafiği
- Parola sıfırlama süreci sırasındaki beklenen hatalar
- İstisna listesine eklenmesi uygun

**Güvenli Müdahale Eylemleri (yalnızca savunma odaklı):**
- Hesabı geçici olarak kilitleyerek koruma altına alma (iş etkisi değerlendirilmeli)
- Parola sıfırlama talebi açma
- Kaynak IP'yi geçici olarak engelleme (dış IP ise)
- İlgili kullanıcıyı bilgilendirme

---

## 5. Algılama 2 — Yeni Kullanıcı Hesabı Oluşturulması

### 5.1 Kural Kartı

| Alan | Açıklama |
|---|---|
| **Kural Başlığı** | Yeni Kullanıcı Hesabı Oluşturulması |
| **Düz Dille Açıklama** | Windows ortamında yeni bir yerel veya etki alanı kullanıcı hesabının oluşturulmasını tespit eder. Kim tarafından, ne zaman ve hangi bağlamda oluşturulduğunu değerlendirmek için tasarlanmıştır. |
| **Ana Log Kaynağı** | Windows Security Event Log |
| **İlgili Event ID** | 4720 — A user account was created |
| **Destekleyici Event ID'ler** | 4722 (hesap etkinleştirildi), 4732/4728/4756 (grup üyeliği eklendi) |
| **Önerilen Önem Derecesi** | **Medium** — Koşullara göre yükseltilebilir |

**Alan Detayları:**
- `SubjectUserName` / `SubjectLogonId`: Hesabı oluşturan kişi/süreç.
- `TargetUserName` / `SamAccountName`: Oluşturulan yeni hesabın adı. **Not:** Bu alanların SIEM'deki eşlemesi doğrulanmalıdır.
- `TargetSid`: Oluşturulan hesabın SID'i — yerel vs etki alanı ayrımı için kullanılabilir.

**Etki Alanı vs Yerel Hesap Bağlamı:**
- Domain Controller üzerinde oluşturulan hesaplar etki alanı hesaplarıdır.
- İş istasyonu veya member server üzerinde oluşturulanlar yerel hesaplardır.
- Yerel hesap oluşturma, merkezî yönetim dışında yapıldığında daha şüphelidir.

**Önem Derecesi Yükseltme Koşulları:**

| Koşul | Yükseltme |
|---|---|
| Mesai dışı veya değişiklik penceresi dışında oluşturuldu | → High |
| Standart yönetici hesabı dışında biri tarafından oluşturuldu | → High |
| Kurumsal adlandırma kurallarına uymayan ad | → High |
| Oluşturulduktan kısa süre sonra ayrıcalıklı gruba eklendi (4732/4728/4756) | → Critical |
| Domain Controller veya Tier-0 varlık üzerinde oluşturuldu | → High |
| Oluşturulduktan hemen sonra etkinleştirildi (4722) ve giriş yapıldı (4624) | → High |

| Alan | Açıklama |
|---|---|
| **Yaygın Yanlış Pozitifler** | BT yöneticilerinin rutin hesap yönetimi; otomasyon/provisioning araçları (SCCM, Azure AD Connect, Intune, HR onboarding sistemleri); test ortamı hesapları; IAM (Identity and Access Management) süreçleri. |
| **MITRE ATT&CK** | T1136.001 — Create Account: Local Account; T1136.002 — Create Account: Domain Account (Tactic: Persistence) |
| **"Kesin Kanıt Değildir" Notu** | Hesap oluşturma meşru BT operasyonlarının doğal bir parçasıdır. Değişiklik yönetimi, HR onboarding ve IAM süreçleri bağlamında değerlendirilmelidir. |

### 5.2 Sigma Kuralı

```yaml
# Internal SOC Detection Pack — Algılama 2 — Yeni Hesap Oluşturma
title: Yeni Windows Kullanıcı Hesabı Oluşturulması
id: a1b2c3d4-e5f6-7890-abcd-ef1234567804  # Üretimde gerçek UUID ile değiştirin
status: test
description: >
  Windows ortamında yeni bir kullanıcı hesabı oluşturulduğunu tespit eder.
  Provisioning araçları ve yetkili yöneticiler istisna listesiyle filtrelenmelidir.
references:
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4720
  - https://attack.mitre.org/techniques/T1136/
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.persistence
  - attack.t1136.001
  - attack.t1136.002
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4720
  # === İstisna Listesi — Ortama Göre Düzenleyin ===
  # filter_provisioning:
  #   SubjectUserName:
  #     - 'svc_provisioning'      # Ortama özgü — doğrulayın
  #     - 'svc_azuread_connect'   # Ortama özgü — doğrulayın
  #     - 'SYSTEM'                # Bazı otomasyon araçları SYSTEM olarak çalışır
  # filter_known_admins:
  #   SubjectUserName:
  #     - 'admin_john.doe'        # Ortama özgü — doğrulayın
  condition: selection
  # Filtrelerle: selection and not filter_provisioning and not filter_known_admins
falsepositives:
  - BT yöneticilerinin rutin hesap oluşturma işlemleri
  - Otomatik provisioning araçları (SCCM, Azure AD Connect, Intune)
  - HR onboarding süreçleri ve IAM sistemleri
  - Test ortamı hesapları
level: medium
# NOT: SubjectUserName ve TargetUserName alan adları SIEM'e göre
# farklı eşlenebilir. Microsoft dokümantasyonuna göre doğrulayın.
# SamAccountName alanı da bazı SIEM'lerde mevcuttur.
# Domain Controller vs iş istasyonu ayrımı için ComputerName alanı
# kullanılabilir.
```

**Korelasyon Notu:** 4720 → 4732/4728/4756 zinciri (hesap oluşturma → ayrıcalıklı gruba ekleme) `temporal` korelasyon kuralı ile izlenebilir. Bu korelasyon Algılama 3 ile birlikte değerlendirilmelidir.

### 5.3 Ayarlama Notları

- Provisioning servis hesaplarını istisna listesine ekleyin ve listeyi düzenli güncelleyin.
- Mesai dışı oluşturmaları yükseltmek için zaman bazlı koşul ekleyin (SIEM'e özgü).
- Domain Controller'lar üzerindeki oluşturmaları ayrı, daha yüksek öncelikli bir kuralda izleyin.
- Oluşturulan hesap adının kurumsal adlandırma kurallarına uyup uymadığını kontrol eden regex tabanlı zenginleştirme düşünülebilir (SIEM desteğine bağlı).
- 4720 → 4722 → 4624 zinciri (oluştur → etkinleştir → giriş yap) korelasyonu yüksek değerli bir sinyaldir.

### 5.4 SOC Runbook

**İlk 5 Soru:**
1. Hesabı kim oluşturdu? (SubjectUserName) Yetkili bir yönetici mi, provisioning aracı mı?
2. Oluşturulan hesap adı kurumsal adlandırma kurallarına uyuyor mu?
3. İşlem ne zaman yapıldı? Mesai saatleri içinde mi, değişiklik penceresi içinde mi?
4. Bu oluşturma için bir değişiklik yönetimi talebi veya HR onboarding kaydı var mı?
5. Oluşturulduktan sonra ayrıcalıklı bir gruba eklenmiş mi? (4732/4728/4756)

**Toplanacak Kanıtlar:**
- 4720 olayının tam detayları (SubjectUserName, TargetUserName, zaman, makine)
- Aynı zaman diliminde 4722 (etkinleştirme) ve 4732/4728/4756 (grup ekleme) olayları
- Oluşturan hesabın normal davranış profili
- Değişiklik yönetimi / HR kaydı

**Yükseltme Koşulları:**
- Bilinmeyen/yetkisiz hesap tarafından oluşturuldu
- Oluşturulduktan kısa süre sonra ayrıcalıklı gruba eklendi
- Domain Controller veya Tier-0 varlık üzerinde
- Mesai dışı ve değişiklik yönetimi kaydı yok

**Yanlış Pozitif Olarak Kapatma Koşulları:**
- Bilinen provisioning aracı tarafından beklenen iş akışı dahilinde oluşturuldu
- Değişiklik yönetimi talebi ile eşleşiyor
- İstisna listesine eklenmesi uygun

---

## 6. Algılama 3 — Ayrıcalıklı Gruba Kullanıcı Eklenmesi

### 6.1 Kural Kartı

| Alan | Açıklama |
|---|---|
| **Kural Başlığı** | Güvenlik Etkin Ayrıcalıklı Gruba Kullanıcı Eklenmesi |
| **Düz Dille Açıklama** | Bir kullanıcının güvenlik etkin yerel, global veya evrensel (universal) bir ayrıcalıklı gruba eklenmesini tespit eder. |
| **Ana Log Kaynağı** | Windows Security Event Log |

**Üç Event ID — Farkları:**

| Event ID | Açıklama | Kapsam |
|---|---|---|
| **4732** | A member was added to a security-enabled **local** group | Yerel gruplar (ör. yerel Administrators) |
| **4728** | A member was added to a security-enabled **global** group | Etki alanı global grupları (ör. Domain Admins) |
| **4756** | A member was added to a security-enabled **universal** group | Evrensel gruplar (ör. Enterprise Admins, Schema Admins) |

**Grup Adı Eşleşme Sorunu:**
- ❗ **Yerelleştirilmiş (lokalize) Windows kurulumlarında grup adları farklıdır.** Örneğin Türkçe Windows'ta "Administrators" yerine "Yöneticiler" kullanılır.
- ❗ **Gruplar yeniden adlandırılabilir.** Kurumlar varsayılan grup adlarını değiştirebilir.
- ❗ **SIEM normalizasyonu** grup adını farklı biçimde saklayabilir.
- ✅ **Önerilen yaklaşım:** Mümkünse grup **SID/RID** değerlerini kullanın. Örneğin:

| Grup | Bilinen SID / RID |
|---|---|
| Administrators (Yerleşik/Builtin) | S-1-5-32-544 |
| Domain Admins | *-512 (etki alanı RID) |
| Enterprise Admins | *-519 |
| Schema Admins | *-518 |
| Account Operators | S-1-5-32-548 |
| Backup Operators | S-1-5-32-551 |
| Server Operators | S-1-5-32-549 |

- ✅ **Alternatif:** Kurumun kendi ayrıcalıklı grup envanterini oluşturun ve bu envantere göre eşleştirme yapın.
- ⚠️ **Production notu:** Aşağıdaki Sigma SID/RID örnekleri okunabilirlik içindir. Kendi SIEM'inizde `TargetSid`, `GroupSid`, `MemberSid`, SID suffix/prefix operatörleri ve normalization davranışı test edilmeden alert moduna alınmamalıdır. Production'da tercih edilen model: kurum tarafından onaylı privileged group inventory + SIEM'de doğrulanmış SID/RID eşlemesi.

**İzlenmesi Gereken Gruplar (savunma amaçlı doğrulama listesi):**
- Administrators / yerel Administrators
- Domain Admins
- Enterprise Admins
- Schema Admins
- Account Operators
- Backup Operators
- Server Operators
- Remote Desktop Users (ortamda hassas ise)
- Kurum tarafından oluşturulmuş özel ayrıcalıklı gruplar

**Önem Derecesi Yükseltme Koşulları:**

| Koşul | Yükseltme |
|---|---|
| Olağandışı bir hesap (SubjectUserName) tarafından ekleme | → Critical |
| Onaylı değişiklik penceresi dışında | → High |
| Yeni oluşturulmuş hesap (4720 sonrası kısa sürede) ayrıcalıklı gruba eklendi | → Critical |
| Geçici ekleme/çıkarma deseni (kısa süre sonra 4733/4729/4757 ile kaldırma) | → High (yetki yükseltme gizleme girişimi olabilir) |
| Domain Controller veya Tier-0 sistem üzerinde | → Critical |

| Alan | Açıklama |
|---|---|
| **Önerilen Önem Derecesi** | **High** — Ayrıcalıklı gruplar için. Düşük ayrıcalıklı gruplar için Medium. |
| **Yaygın Yanlış Pozitifler** | Planlı yönetici atama süreçleri; değişiklik yönetimi dahilinde yapılan grup değişiklikleri; otomasyon araçlarının geçici yetki yükseltmeleri; test ortamı işlemleri. |
| **MITRE ATT&CK** | T1098 — Account Manipulation (Tactic: Persistence, Privilege Escalation) |
| **"Kesin Kanıt Değildir" Notu** | Grup üyelik değişiklikleri meşru yönetim süreçlerinin parçası olabilir. Değişiklik yönetimi kayıtları ve iş bağlamıyla birlikte değerlendirilmelidir. |

### 6.2 Sigma Kuralı

```yaml
# Internal SOC Detection Pack — Algılama 3 — Ayrıcalıklı Gruba Ekleme
title: Güvenlik Etkin Ayrıcalıklı Gruba Kullanıcı Eklenmesi
id: a1b2c3d4-e5f6-7890-abcd-ef1234567805  # Üretimde gerçek UUID ile değiştirin
status: test
description: >
  Bir kullanıcının güvenlik etkin yerel (4732), global (4728) veya evrensel
  (4756) bir ayrıcalıklı gruba eklenmesini tespit eder. Yetki yükseltme
  girişimlerine işaret edebilir.
references:
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4732
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4728
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4756
  - https://attack.mitre.org/techniques/T1098/
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.persistence
  - attack.privilege_escalation
  - attack.t1098
logsource:
  product: windows
  service: security
detection:
  selection_event:
    EventID:
      - 4732
      - 4728
      - 4756
  # --- Seçenek A: Grup Adı Tabanlı Eşleşme ---
  # ⚠️ Yerelleştirilmiş Windows'ta farklı olabilir!
  filter_privileged_groups_by_name:
    TargetUserName:
      - 'Administrators'
      - 'Domain Admins'
      - 'Enterprise Admins'
      - 'Schema Admins'
      - 'Account Operators'
      - 'Backup Operators'
      - 'Server Operators'
      # Türkçe Windows için (doğrulama gerekir):
      # - 'Yöneticiler'
      # - 'Etki Alanı Yöneticileri'
      # Kurum özel grupları eklenebilir
  # --- Seçenek B: SID/RID Tabanlı Eşleşme (Tercih Edilen, ancak platforma bağlı) ---
  # ⚠️ Bu blok production-ready kopyala-yapıştır değildir.
  # SIEM'inizde TargetSid/GroupSid alanının varlığı, SID normalization,
  # startswith/endswith operatör desteği ve grup inventory eşleşmesi doğrulanmalıdır.
  # filter_privileged_groups_by_sid:
  #   TargetSid|startswith:
  #     - 'S-1-5-32-544'   # Builtin Administrators
  #     - 'S-1-5-32-548'   # Account Operators
  #     - 'S-1-5-32-549'   # Server Operators
  #     - 'S-1-5-32-551'   # Backup Operators
  #   TargetSid|endswith:
  #     - '-512'            # Domain Admins
  #     - '-518'            # Schema Admins
  #     - '-519'            # Enterprise Admins
  #   # NOT: SID endswith eşleşmesi Sigma'da
  #   # tüm modifiye edilebilir değildir — SIEM'e göre doğrulayın.
  condition: selection_event and filter_privileged_groups_by_name
  # SID tabanlı: selection_event and filter_privileged_groups_by_sid
falsepositives:
  - Planlı yönetici atama süreçleri
  - Değişiklik yönetimi dahilinde yapılan grup değişiklikleri
  - Otomasyon araçlarının geçici yetki yükseltmeleri
  - Test ortamı işlemleri
level: high
# NOT: TargetUserName alanı bu olayda GRUP ADINI taşır, kullanıcı adını değil.
# Eklenen kullanıcı MemberName veya MemberSid alanında bulunur.
# Bu alan eşlemesi SIEM'e göre farklıdır — mutlaka doğrulayın.
# Grup adı yerine SID tabanlı eşleşme daha güvenilirdir ancak
# tüm SIEM'ler SID alanlarını aynı şekilde işlemez.
```

### 6.3 Ayarlama Notları

- **SID tabanlı eşleşmeyi tercih edin.** Grup adları dilin, yeniden adlandırmanın ve SIEM normalizasyonunun etkisindedir.
- Kurum özel ayrıcalıklı grup envanterini oluşturun ve düzenli güncelleyin.
- Düşük ayrıcalıklı gruplar için ayrı, daha düşük öncelikli bir kural oluşturun.
- 4720 → 4732/4728/4756 korelasyonu (hesap oluştur → gruba ekle) kritik bir sinyaldir.
- Geçici ekleme/çıkarma deseni (4732 → 4733, 4728 → 4729, 4756 → 4757) için temporal korelasyon düşünülebilir.
- Değişiklik yönetimi entegrasyonu yanlış pozitif oranını önemli ölçüde düşürür.

### 6.4 SOC Runbook

**İlk 5 Soru:**
1. Değişikliği kim yaptı? (SubjectUserName) Yetkili bir yönetici mi?
2. Hangi kullanıcı (MemberName/MemberSid) hangi gruba (TargetUserName) eklendi?
3. Bir değişiklik yönetimi talebi var mı?
4. Ekleme mesai saatleri ve onaylı değişiklik penceresi içinde mi?
5. Eklenen kullanıcı yeni oluşturulmuş bir hesap mı? (4720 korelasyonu)

**Toplanacak Kanıtlar:**
- 4732/4728/4756 olayının tam detayları
- Aynı zaman diliminde 4720 (hesap oluşturma) olayları
- SubjectUserName'in normal davranış profili
- Değişiklik yönetimi / onay kaydı
- Eklenen kullanıcının mevcut ayrıcalık düzeyi

**Yükseltme Koşulları:**
- Yetkisiz hesap tarafından ekleme
- Yeni oluşturulmuş hesap ayrıcalıklı gruba eklendi
- Domain Controller üzerinde
- Değişiklik yönetimi kaydı yok
- Kısa süre sonra eklenen üye kaldırıldı (geçici yetki yükseltme)

**Yanlış Pozitif Olarak Kapatma Koşulları:**
- Değişiklik yönetimi talebi ile eşleşen, onaylı süreç
- Bilinen otomasyon aracı tarafından beklenen iş akışı dahilinde

---

## 7. Algılama 4 — Şüpheli PowerShell Etkinliği (Katmanlı Model)

### 7.1 Kural Kartı

| Alan | Açıklama |
|---|---|
| **Kural Başlığı** | Şüpheli PowerShell Etkinliği — Katmanlı Algılama Modeli |
| **Düz Dille Açıklama** | PowerShell Script Block Logging üzerinden olağandışı desenleri tespit eder. **Önemli:** PowerShell, Windows sistem yönetiminin temel araçlarından biridir. Bu kuralın amacı meşru kullanımı engellemek değil, olağandışı desenleri görünür kılmaktır. Anahtar kelime eşleşmesi tek başına çok gürültülüdür; bu nedenle katmanlı bir algılama modeli kullanılır. |
| **Ana Log Kaynağı** | Microsoft-Windows-PowerShell/Operational (Event ID 4104 — Script Block Logging) |

**Ön Koşullar:**
- ✅ **Script Block Logging etkinleştirilmiş olmalıdır.** Etkinleştirme: Group Policy → Computer Configuration → Administrative Templates → Windows Components → Windows PowerShell → Turn on PowerShell Script Block Logging.
- ⚠️ **Protected Event Logging:** Script blok içeriği hassas veri içerebilir. Protected Event Logging etkinleştirilerek içerik şifrelenebilir; ancak bu durumda analiz için şifre çözme gerekir.
- ⚠️ **EDR / Sysmon gereksinimi:** Parent/child process bağlamı için EDR veya Sysmon process creation telemetrisi gerekebilir. PowerShell logları tek başına parent process bilgisini sınırlı sağlar.
- ⚠️ **AMSI, Module Logging, Transcription ve 4688 Process Creation** logları algılamayı zenginleştirebilir ancak bu kurallar bunlara bağımlı değildir.

**Katmanlı Algılama Modeli:**

#### Katman 1 — Düşük Güvenilirlikli Göstergeler (Hunting / Telemetri)

Bu katman tek başına uyarı üretmemelidir; hunting sorgusu veya zenginleştirme girişi olarak kullanılmalıdır.

| Gösterge | Açıklama |
|---|---|
| Kodlanmış komut parametresi | `-EncodedCommand`, `-enc` gibi parametrelerin kullanımı |
| Yürütme politikası atlama | `-ExecutionPolicy Bypass`, `-ep bypass` gibi parametreler |
| İndirme deseni | `DownloadString`, `DownloadFile`, `Invoke-WebRequest` gibi cmdlet'ler |
| Olağandışı uzunlukta script blok | Karakter sayısı ortam ortalamasının çok üzerinde (eşik ortama özgü) |
| Karartma (obfuscation) göstergeleri | Aşırı string birleştirme, karakter dizisi ters çevirme, format string manipülasyonu gibi desenler |

#### Katman 2 — Bağlam Zenginleştirme

Script Block Logging ile birlikte veya SIEM korelasyonu ile elde edilebilecek bağlamsal faktörler:

| Faktör | Kaynak | Not |
|---|---|---|
| Olağandışı parent process | EDR / Sysmon / 4688 | PowerShell'in beklenen parent'ları: explorer.exe, cmd.exe, scheduled task engine vb. |
| Olağandışı kullanıcı | 4104 — UserName alanı | PowerShell normalde kullanmayan bir kullanıcı |
| Yönetici olmayan kullanıcı + riskli desen | 4104 + kullanıcı yetki bilgisi | |
| Kullanıcı-yazılabilir yoldan çalıştırma | 4104 — Path alanı | `%TEMP%`, `%APPDATA%`, `Downloads` vb. |
| Ağ bağlantısı (zaman yakınlığı) | Sysmon 3 / EDR / güvenlik duvarı | Script çalıştırma sonrası kısa sürede dış bağlantı |
| Dosya yazma (zaman yakınlığı) | Sysmon 11 / EDR | Script sonrası kullanıcı-yazılabilir dizine dosya oluşturma |
| Süreç oluşturma | 4688 / Sysmon 1 | Script sonrası yeni süreç başlatma |
| İmzasız/bilinmeyen script yolu | EDR | Script dosyasının dijital imza durumu |

#### Katman 3 — Yüksek Güvenilirlikli Uyarı

İki veya daha fazla göstergenin birlikte gözlenmesi daha güçlü bir sinyal oluşturur:

| Kombinasyon | Önerilen Önem |
|---|---|
| Katman 1 göstergesi + olağandışı parent process | High |
| Katman 1 göstergesi + dış ağ bağlantısı (zaman yakınlığı) | High |
| Katman 1 göstergesi + kullanıcı-yazılabilir dizine dosya oluşturma | High |
| Katman 1 göstergesi + yüksek değerli varlık (DC, Tier-0) | High |
| Katman 1 göstergesi + normalde PowerShell kullanmayan kullanıcı/servis hesabı | High |
| Birden fazla Katman 1 göstergesi birlikte | Medium-High |

| Alan | Açıklama |
|---|---|
| **Önerilen İlk Önem** | **Low** (Katman 1 tek başına); **Medium-High** (Katman 3 kombinasyonları) |
| **Önerilen İlk Mod** | **Hunting / Audit** — Baseline ölçümü ve ayarlama sonrası alert moduna geçiş |
| **Yaygın Yanlış Pozitifler** | Sistem yöneticilerinin meşru otomasyon betikleri; SCCM/Intune dağıtım betikleri; DSC yapılandırmaları; güvenlik tarama araçları; kodlanmış parametre kullanan meşru yazılım yükleyicileri; EDR/file metadata ile doğrulanmış güvenilir vendor modülleri; güncelleme betikleri. |
| **MITRE ATT&CK** | T1059.001 — Command and Scripting Interpreter: PowerShell (Tactic: Execution) |
| **"Kesin Kanıt Değildir" Notu** | PowerShell Windows yönetiminin temel aracıdır. Encoded komut kullanımı bile birçok meşru senaryoda normaldir. Bu kurallar mutlaka bağlam ve ek kanıtlarla birlikte değerlendirilmelidir. |

### 7.2 Sigma Kuralları

#### 7.2.A — Katman 1: Şüpheli PowerShell Göstergeleri (Hunting Kuralı)

```yaml
# Internal SOC Detection Pack — Algılama 4A — PowerShell Katman 1
# ⚠️ Bu kural tek başına uyarı üretmek için UYGUN DEĞİLDİR.
# Hunting sorgusu veya korelasyon girişi olarak kullanılmalıdır.
title: PowerShell Şüpheli Gösterge — Katman 1 (Hunting)
name: internal_ps_suspicious_indicator
id: a1b2c3d4-e5f6-7890-abcd-ef1234567806  # Üretimde gerçek UUID ile değiştirin
status: experimental
description: >
  PowerShell Script Block Logging'de şüpheli desenleri tespit eder.
  Tek başına uyarı üretmek için tasarlanmamıştır; katmanlı algılama
  modelinin Katman 1 girişidir. Yüksek yanlış pozitif oranı beklenir.
references:
  - https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging
  - https://attack.mitre.org/techniques/T1059/001/
  - https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/wmf/whats-new/script-logging
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.execution
  - attack.t1059.001
logsource:
  product: windows
  category: ps_script
  # NOT: 'category: ps_script' SigmaHQ standardına göre
  # PowerShell Script Block Logging'e (Event ID 4104) eşlenir.
  # SIEM'inizdeki eşlemeyi doğrulayın.
detection:
  selection_encoded:
    ScriptBlockText|contains:
      - '-EncodedCommand'
      - '-enc '
      # NOT: '-e ' çok kısa; yanlış pozitif oranı yüksek olabilir.
      # Ortamınızda test edin.
  selection_bypass:
    ScriptBlockText|contains:
      - '-ExecutionPolicy Bypass'
      - '-ExecutionPolicy Unrestricted'
      - '-ep bypass'
  selection_download:
    ScriptBlockText|contains:
      - '.DownloadString('
      - '.DownloadFile('
      - 'Invoke-WebRequest '
      - 'Invoke-RestMethod '
      - 'Start-BitsTransfer '
      # NOT: 'iwr', 'wget', 'curl' takma adları çok geniştir;
      # tam cmdlet adları tercih edilmiştir.
  selection_obfuscation:
    ScriptBlockText|contains:
      - '[char]'
      - 'join('
      - '-replace'
      - '-split'
      - '[Convert]::'
      - 'FromBase64String'
    # ⚠️ Bu göstergeler çok geniştir; tek başına kullanılmamalı,
    # diğer göstergelerle korelasyon gerektirir.
  # === Meşru Otomasyon İstisnaları ===
  filter_known_scripts:
    # Ortama özgü — bilinen meşru betiklerin yolları veya hash'leri
    Path|contains:
      - 'C:\Program Files\Microsoft Monitoring Agent'
      - 'C:\Program Files\Microsoft Configuration Manager'
      # Ek meşru yollar ortama göre eklenmelidir
  # İmza / reputation filtresi burada bilerek uygulanmaz.
  # ScriptBlockText içinde imza bloğu görmek imzanın geçerli olduğunu kanıtlamaz.
  # Güvenilirlik kontrolü gerekiyorsa EDR, code-signing metadata, file hash
  # reputation veya software inventory zenginleştirmesi ile yapılmalıdır.
  condition: >
    (selection_encoded or selection_bypass or selection_download or selection_obfuscation)
    and not filter_known_scripts
falsepositives:
  - Sistem yöneticilerinin meşru otomasyon betikleri
  - SCCM, Intune, DSC dağıtım betikleri
  - Güvenlik tarama araçları
  - İmzalı veya bilinen güvenilir modüller; yalnızca EDR/file metadata ile doğrulanırsa
  - Kodlanmış parametre kullanan meşru yazılımlar
level: low
# ⚠️ BU KURAL TEK BAŞINA UYARI ÜRETMEMELİDİR.
# Hunting sorgusu veya Katman 3 korelasyonunun girişi olarak kullanın.
```

#### 7.2.B — Katman 3: Şüpheli PowerShell + Bağlam Korelasyonu

```yaml
# Internal SOC Detection Pack — Algılama 4B — PowerShell Katman 3 (Kavramsal)
# ⚠️ Bu kural kavramsal bir korelasyon taslağıdır.
# Tam uygulama EDR / Sysmon entegrasyonu gerektirir.
title: PowerShell Şüpheli Gösterge + Olağandışı Bağlam (Katman 3)
id: a1b2c3d4-e5f6-7890-abcd-ef1234567807  # Üretimde gerçek UUID ile değiştirin
status: experimental
description: >
  Katman 1 PowerShell göstergelerinin olağandışı bağlam faktörleriyle
  birlikte gözlenmesi durumunda uyarı üretir. Bu kavramsal bir tasarımdır;
  tam uygulama SIEM korelasyon yeteneklerine ve EDR/Sysmon entegrasyonuna
  bağlıdır.
references:
  - https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging
  - https://attack.mitre.org/techniques/T1059/001/
  - https://sigmahq.io/docs/meta/correlations.html
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.execution
  - attack.t1059.001
# --- Kavramsal Korelasyon Mantığı ---
# Bu korelasyon Sigma'nın temporal tipi ile kurulabilir,
# ancak farklı log kaynaklarını (ps_script + sysmon/process_creation)
# birleştirdiği için SIEM'e özgü uygulama gerektirir.
#
# Yaklaşım 1 — Script Block + Kullanıcı-yazılabilir yol:
# Katman 1 kuralı tetiklendi VE Path alanı şüpheli bir dizin içeriyor
# (%TEMP%, %APPDATA%, Downloads, Public vb.)
#
# Yaklaşım 2 — Script Block + Olağandışı kullanıcı:
# Katman 1 kuralı tetiklendi VE çalıştıran kullanıcı bilinen PS
# kullanıcıları listesinde değil
#
# Yaklaşım 3 — Script Block + Ağ bağlantısı (temporal):
# Katman 1 kuralı tetiklendi VE aynı host'ta kısa sürede
# dış ağ bağlantısı gözlendi (Sysmon Event 3 veya EDR)
#
# Aşağıdaki örnek Yaklaşım 1'in basitleştirilmiş halidir.
logsource:
  product: windows
  category: ps_script
detection:
  # Katman 1 göstergelerinden herhangi biri
  selection_encoded:
    ScriptBlockText|contains:
      - '-EncodedCommand'
      - '-enc '
  selection_download:
    ScriptBlockText|contains:
      - '.DownloadString('
      - '.DownloadFile('
      - 'Invoke-WebRequest '
  # Şüpheli yoldan çalıştırma (bağlam)
  selection_suspicious_path:
    Path|contains:
      - '\AppData\'
      - '\Temp\'
      - '\Downloads\'
      - '\Public\'
      - '\ProgramData\'
      # NOT: Bu yollar ortama göre ayarlanmalıdır.
  # Bilinen meşru yolları hariç tut
  filter_known:
    Path|contains:
      - 'C:\Program Files\'
      - 'C:\Program Files (x86)\'
      - 'C:\Windows\System32\'
  condition: >
    (selection_encoded or selection_download)
    and selection_suspicious_path
    and not filter_known
falsepositives:
  - Geçici dizinden çalışan meşru yükleyiciler
  - AppData altında çalışan meşru uygulamalar (VS Code, Teams vb.)
level: medium
# NOT: Bu kural kavramsal bir Katman 3 başlangıcıdır.
# Tam Katman 3 korelasyonu için EDR/Sysmon process creation
# verileriyle temporal korelasyon gerekir.
# SIEM'inizin farklı log kaynaklarını korelasyon yeteneğini doğrulayın.
# Script Block Logging etkinleştirilmiş olmalıdır.
```

### 7.3 Ayarlama Notları

- **Katman 1'i doğrudan alert olarak kullanmayın.** Hunting sorgusu veya korelasyon girişi olarak kullanın.
- Bilinen meşru betiklerin `Path` değerlerini veya hash'lerini istisna listesine ekleyin.
- `selection_obfuscation` göstergeleri çok geniştir; başlangıçta devre dışı bırakıp ortam baseline'ına göre etkinleştirin.
- Normalde PowerShell kullanmayan kullanıcı/makine listesi oluşturun; bu listede yer alan kaynaklardan gelen Katman 1 olaylarını yükseltin.
- Yüksek değerli varlıklarda (DC, veritabanı sunucuları) daha düşük eşikle çalıştırın.
- Encoded komut içeriğinin decode edilmesi (Script Block Logging bunu otomatik yapar) analizi kolaylaştırır.

### 7.4 SOC Runbook

**İlk 5 Soru:**
1. Komutu hangi kullanıcı çalıştırdı? Bu kullanıcı normalde PowerShell kullanır mı?
2. Hangi makinede çalıştırıldı? Bu makine yüksek değerli bir varlık mı?
3. Script bloğun decoded içeriği ne? (Script Block Logging otomatik decode eder)
4. Script hangi yoldan çalıştırıldı? (Path alanı)
5. Parent process ne? (EDR/Sysmon varsa)

**Toplanacak Kanıtlar:**
- 4104 olayının tam Script Block içeriği
- Çalıştıran kullanıcının normal davranış profili
- Parent process bilgisi (EDR/Sysmon)
- Aynı zaman diliminde ağ bağlantıları (Sysmon 3 / güvenlik duvarı)
- Aynı zaman diliminde dosya oluşturma olayları (Sysmon 11)

**Yükseltme Koşulları:**
- Encoded komut + dış ağ bağlantısı birlikte gözlendi
- Normalde PowerShell kullanmayan kullanıcı + indirme deseni
- Yüksek değerli varlık üzerinde herhangi bir Katman 1 göstergesi
- Birden fazla Katman 1 göstergesi birlikte

**Yanlış Pozitif Olarak Kapatma Koşulları:**
- Bilinen otomasyon aracının beklenen betik çalıştırması
- EDR/file metadata ile doğrulanmış bilinen güvenilir script/modül
- İstisna listesine eklenmesi uygun

---

## 8. Algılama 5 — Yeni Windows Servis Kurulumu

### 8.1 Kural Kartı

| Alan | Açıklama |
|---|---|
| **Kural Başlığı** | Yeni Windows Servis Kurulumu Algılama |
| **Düz Dille Açıklama** | Bir Windows sisteminde yeni bir servisin kurulmasını hem Security (4697) hem de System (7045) loglarından tespit eder. |

**İki Event ID — Farkları:**

| Event ID | Log Kaynağı | Açıklama | Denetim Gereksinimi |
|---|---|---|---|
| **4697** | Security | A service was installed in the system | Audit Security System Extension politikası etkinleştirilmeli |
| **7045** | System | Service Control Manager — yeni servis kuruldu | Çoğu Windows ortamında System log içinde görülebilir; merkezi toplama/SIEM ingestion mutlaka doğrulanmalıdır |

**Önemli:** Bazı ortamlarda 4697 denetim politikası etkinleştirilmemiş olabilir. 7045 çoğu Windows ortamında System log / Service Control Manager kaynağında görülebilir; ancak merkezi log toplama, Windows Event Forwarding, agent konfigürasyonu ve SIEM ingestion durumu mutlaka doğrulanmalıdır. **Her iki kaynağı birlikte kullanmanız önerilir.**

**Alan Adı Farklılıkları:**

| Bilgi | 4697 Alanı | 7045 Alanı | Not |
|---|---|---|---|
| Servis adı | ServiceName | ServiceName | Genellikle aynı |
| Servis dosya yolu | ServiceFileName | ImagePath | ⚠️ Farklı alan adı! |
| Servisi kuran hesap | SubjectUserName | AccountName | ⚠️ Farklı alan adı! |
| Servis tipi | ServiceType | ServiceType | |
| Başlatma türü | ServiceStartType | StartType | ⚠️ Farklı alan adı! |

**⚠️ SIEM platformunuzdaki alan eşlemelerini doğrulayın.** Elastic, Splunk, Sentinel ve Wazuh bu alanları farklı şekilde normalleştirip eşleyebilir.

**Şüpheli Servis Kurulum Göstergeleri:**

| Gösterge | Açıklama |
|---|---|
| Kullanıcı-yazılabilir yoldan çalışma | `%TEMP%`, `%APPDATA%`, `Downloads`, `Public`, kullanıcı profil dizini |
| Meşru yazılımı taklit eden ad | Servis adı bilinen yazılıma benziyor ama yol farklı |
| Olağandışı hesap tarafından kurulum | Bilinen yönetici veya dağıtım aracı olmayan hesap |
| Değişiklik yönetimi kaydı yok | Planlı dağıtım dışında |
| Şüpheli giriş/yetki yükseltme sonrası | 4625→4624 veya 4732 sonrası kısa sürede servis kurulumu |
| Yüksek değerli varlık | Domain Controller, Tier-0 sistem |

**Gürültü Azaltma — Bilinen Meşru Kaynaklar:**

| Kaynak | Açıklama |
|---|---|
| Yazılım dağıtım araçları | SCCM, Intune, WSUS, Ansible, Chef, Puppet |
| Güvenlik araçları | EDR ajanları, antivirüs güncellemeleri |
| İzleme araçları | Monitoring agent kurulumları |
| İşletim sistemi | Windows Update, sürücü yüklemeleri |
| Bilinen satıcı yükleyicileri | Kurumda onaylı yazılım listesi |

| Alan | Açıklama |
|---|---|
| **Önerilen Önem Derecesi** | **Medium** — Yol veya hesap şüpheli ise High |
| **Yaygın Yanlış Pozitifler** | Yazılım güncellemeleri; yama yönetimi; güvenlik aracı güncellemeleri; sürücü yüklemeleri; izleme aracı kurulumları; meşru yazılım dağıtımı. |
| **MITRE ATT&CK** | T1543.003 — Create or Modify System Process: Windows Service (Tactic: Persistence, Privilege Escalation) |
| **"Kesin Kanıt Değildir" Notu** | Servis kurulumu yazılım dağıtımının doğal bir parçasıdır. Değişiklik yönetimi kayıtları ve bağlamla birlikte değerlendirilmelidir. |

### 8.2 Sigma Kuralları

#### 8.2.A — Security Log (4697)

```yaml
# Internal SOC Detection Pack — Algılama 5A — Servis Kurulumu (Security Log)
title: Yeni Servis Kurulumu — Security Log (4697)
name: internal_new_service_security
id: a1b2c3d4-e5f6-7890-abcd-ef1234567808  # Üretimde gerçek UUID ile değiştirin
status: test
description: >
  Windows Security logunda yeni servis kurulumunu (4697) tespit eder.
  Audit Security System Extension politikasının etkinleştirilmesi gerekir.
references:
  - https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4697
  - https://attack.mitre.org/techniques/T1543/003/
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.persistence
  - attack.privilege_escalation
  - attack.t1543.003
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4697
  # === Bilinen Yazılım Dağıtım Yollarını Hariç Tut ===
  filter_trusted_paths:
    ServiceFileName|startswith:
      - 'C:\Program Files\'
      - 'C:\Program Files (x86)\'
      - 'C:\Windows\System32\'
      - 'C:\Windows\SysWOW64\'
      # Ortama özgü güvenilir yollar eklenebilir
  # filter_trusted_accounts:
  #   SubjectUserName:
  #     - 'SYSTEM'
  #     - 'svc_sccm'            # Ortama özgü
  #     - 'svc_intune'          # Ortama özgü
  condition: selection and not filter_trusted_paths
  # Filtrelerle: selection and not filter_trusted_paths and not filter_trusted_accounts
falsepositives:
  - Yazılım kurulumları ve güncellemeleri
  - Yama yönetimi araçları
  - Güvenlik aracı güncellemeleri
  - Sürücü yüklemeleri
  - İzleme aracı kurulumları
level: medium
# NOT: Bu kural yalnızca güvenilir yollar DIŞINDAN kurulan servisleri
# raporlar. Güvenilir yol listesi ortama özgüdür ve düzenli
# güncellenmelidir.
# ServiceFileName alan adı SIEM'e göre farklı olabilir
# (ImagePath, BinaryPathName, service.executable vb.) — doğrulama gerekir.
# Audit Security System Extension politikası etkinleştirilmiş olmalıdır.
```

#### 8.2.B — System Log (7045)

```yaml
# Internal SOC Detection Pack — Algılama 5B — Servis Kurulumu (System Log)
title: Yeni Servis Kurulumu — System Log (7045)
name: internal_new_service_system
id: a1b2c3d4-e5f6-7890-abcd-ef1234567809  # Üretimde gerçek UUID ile değiştirin
status: test
description: >
  Windows System logunda Service Control Manager tarafından kaydedilen
  yeni servis kurulumunu (7045) tespit eder. Çoğu Windows ortamında
  System log içinde görülebilir; merkezi toplama ve SIEM ingestion durumu
  mutlaka doğrulanmalıdır.
references:
  - https://attack.mitre.org/techniques/T1543/003/
  # 7045 için resmi Microsoft Security Auditing dokümantasyonu sınırlıdır;
  # System Event Log SCM (Service Control Manager) kapsamındadır.
  # Kaynak doğrulaması gerekebilir.
author: Internal Detection Engineering
date: 2026/07/04
tags:
  - attack.persistence
  - attack.privilege_escalation
  - attack.t1543.003
logsource:
  product: windows
  service: system
detection:
  selection:
    EventID: 7045
  filter_trusted_paths:
    ImagePath|startswith:
      - 'C:\Program Files\'
      - 'C:\Program Files (x86)\'
      - 'C:\Windows\System32\'
      - 'C:\Windows\SysWOW64\'
  # filter_trusted_services:
  #   ServiceName:
  #     - 'wuauserv'          # Windows Update
  #     - 'WinDefend'         # Windows Defender
  #     # Ortama özgü bilinen servisler
  condition: selection and not filter_trusted_paths
falsepositives:
  - Yazılım kurulumları ve güncellemeleri
  - Yama yönetimi araçları
  - Güvenlik aracı güncellemeleri
  - Sürücü yüklemeleri
level: medium
# NOT: 7045'te servis dosya yolu 'ImagePath' olarak adlandırılır;
# 4697'deki 'ServiceFileName'den farklıdır.
# Bazı SIEM'ler 7045'i 4697'den daha iyi ayrıştırır.
# Her iki kaynağı birlikte kullanmanız önerilir.
# 7045 çoğu ortamda System log içinde görülebilir; ancak merkezi toplama,
# agent parsing ve SIEM ingestion doğrulanmalıdır. 4697 audit policy gerektirir.
```

### 8.3 Ayarlama Notları

- **Her iki Event ID'yi (4697 + 7045) birlikte kullanın.** 4697 audit policy gerektirir; 7045 çoğu ortamda System log içinde görülebilir ancak merkezi toplama, agent parsing ve SIEM ingestion doğrulanmalıdır.
- Güvenilir servis yolu listesini oluşturun ve düzenli güncelleyin.
- Yazılım dağıtım araçlarının kurduğu servisleri istisna listesine alın.
- `%TEMP%`, `%APPDATA%`, kullanıcı profil dizinlerinden çalışan servisleri yükseltin.
- Değişiklik yönetimi ve yazılım dağıtım takvimi ile korelasyon kurun.
- Yüksek değerli varlıklarda (DC, veritabanı sunucuları) daha düşük toleransla çalıştırın.
- Servis adının bilinen yazılımı taklit edip etmediğini kontrol eden string benzerlik kuralları düşünülebilir (SIEM desteğine bağlı).

### 8.4 SOC Runbook

**İlk 5 Soru:**
1. Servisi kim kurdu? (SubjectUserName / AccountName) Yetkili bir hesap mı?
2. Servis dosya yolu nedir? Güvenilir bir konum mu? (Program Files, System32 vs %TEMP%, %APPDATA%)
3. Servis adı bilinen bir yazılıma ait mi?
4. Bu kurulum için bir değişiklik yönetimi talebi veya yazılım dağıtım kaydı var mı?
5. Kurulum zamanlaması planlı bir dağıtım penceresiyle örtüşüyor mu?

**Toplanacak Kanıtlar:**
- 4697 ve/veya 7045 olayının tam detayları
- Servis dosyasının hash değeri ve dijital imza durumu (EDR varsa)
- Servisi kuran hesabın normal davranış profili
- Aynı zaman diliminde şüpheli giriş (4625→4624) veya yetki yükseltme (4732) olayları
- Değişiklik yönetimi / yazılım dağıtım kaydı

**Yükseltme Koşulları:**
- Kullanıcı-yazılabilir yoldan çalışan servis
- Bilinmeyen/yetkisiz hesap tarafından kurulum
- Değişiklik yönetimi kaydı yok ve planlı dağıtım dışında
- Şüpheli giriş veya yetki yükseltme sonrası kısa sürede kurulum
- Yüksek değerli varlık üzerinde

**Yanlış Pozitif Olarak Kapatma Koşulları:**
- Yazılım dağıtım aracı tarafından planlı dağıtım dahilinde
- Değişiklik yönetimi talebi ile eşleşen
- Bilinen satıcı yükleyicisi tarafından güvenilir yolda

---

## 9. SIEM Platform Eşleme Notları

Sigma kuralları platforma bağımsız bir formattır ancak her SIEM farklı alan adları, sorgu dili ve korelasyon söz dizimi kullanır. Aşağıdaki tablo temel eşleme farklılıklarını gösterir.

**⚠️ Bu notlar rehber niteliğindedir; her platform için tam doğrulama gerektirir.**

| Sigma Alanı | Microsoft Sentinel (KQL) | Splunk (SPL) | Elastic (ECS) | Wazuh | QRadar |
|---|---|---|---|---|---|
| `TargetUserName` | `TargetUserName` | `TargetUserName` veya `user` | `winlog.event_data.TargetUserName` | `data.win.eventdata.targetUserName` | Doğrulama gerekir |
| `SourceNetworkAddress` | `IpAddress` | `src_ip` veya `SourceNetworkAddress` | `source.ip` | `data.win.eventdata.ipAddress` | Doğrulama gerekir |
| `SubjectUserName` | `SubjectUserName` | `SubjectUserName` | `winlog.event_data.SubjectUserName` | `data.win.eventdata.subjectUserName` | Doğrulama gerekir |
| `ServiceFileName` (4697) | `ServiceFileName` | `ServiceFileName` | `winlog.event_data.ServiceFileName` | Doğrulama gerekir | Doğrulama gerekir |
| `ImagePath` (7045) | `ImagePath` | `ImagePath` | `winlog.event_data.ImagePath` | Doğrulama gerekir | Doğrulama gerekir |
| `ScriptBlockText` | `ScriptBlockText` | `ScriptBlockText` | `winlog.event_data.ScriptBlockText` veya `powershell.file.script_block_text` | Doğrulama gerekir | Doğrulama gerekir |

**Korelasyon Söz Dizimi Farklılıkları:**
- **Sigma Correlation** (`event_count`, `value_count`, `temporal`) → Hedef platforma dönüştürülmeden önce kullanılan `sigma-cli`/`pySigma` backend'i, backend sürümü ve hedef SIEM'in korelasyon kabiliyeti doğrulanmalıdır. Bazı backend'ler eşik/korelasyon mantığını çevirebilir; bazı ortamlarda manuel KQL/SPL/EQL/AQL kuralı yazmak gerekir.
- **Microsoft Sentinel:** KQL `summarize count() by ... | where count_ > threshold` ile karşılanabilir.
- **Splunk:** `| stats count by ... | where count > threshold` ile doğal destek.
- **Elastic:** ES|QL aggregation veya Detection Rules threshold tipi ile.
- **Wazuh:** Kendi korelasyon formatı; Sigma dönüşümü sınırlı olabilir.
- **QRadar:** AQL ve custom rule engine; Sigma dönüşümü sınırlıdır.

**Sigma Dönüşüm Araçları:**
- `sigma-cli` ve `pySigma` backend'leri kullanılarak Sigma kuralları hedef SIEM formatına dönüştürülebilir.
- Her dönüşüm sonrası hedef platformda doğrulama gereklidir.

---


## 10. İç Onay ve Production Gate

Bir kural `alert` moduna alınmadan önce aşağıdaki kurumsal gate tamamlanmalıdır:

| Gate | Beklenen Kanıt | Onay Sahibi |
|---|---|---|
| Log coverage | Son 7–14 gün içinde ilgili Event ID'lerin SIEM'e geldiğini gösteren sorgu çıktısı | Windows Platform / SIEM Owner |
| Field mapping | Kuralda kullanılan her alanın SIEM'deki gerçek alan adıyla eşleştiğini gösteren örnek olay | Detection Engineering |
| Baseline | Normal olay hacmi, saatlik/günlük dağılım, top kullanıcı/IP/host listesi | Detection Engineering |
| False positive review | İlk audit-mode çalıştırmasından FP örnekleri ve önerilen allowlist | SOC L2 / Detection Engineering |
| Change-management alignment | Hesap oluşturma, grup değişikliği ve servis kurulumu için change-ticket eşleşme yöntemi | IT Operations / CAB |
| Runbook readiness | L1/L2'nin kullanacağı triage adımları, escalation kriterleri, kapanış nedenleri | SOC Lead |
| Privacy review | Özellikle PowerShell ScriptBlockText gibi hassas içerik barındırabilecek logların saklama/erişim modeli | Security Governance / Legal |
| Rollback plan | Kural çok gürültülü olursa devre dışı bırakma ve sürüm geri alma yöntemi | SIEM Owner |
| Alert activation | İlk 2 hafta için gözlem penceresi ve günlük review planı | SOC Lead |

**Varsayılan karar:** Bu paketteki kurallar önce `audit` veya `hunting` modunda çalıştırılır. Alert modu yalnızca owner onayı, baseline ve FP analizi sonrası açılır. Otomatik aksiyon üreten playbook'lara bağlanmaz; önce manuel triage ile olgunlaştırılır.

---

## 11. Açık Kaynak / Harici Paylaşım Rehberi

### İzin Verilen Harici Katkılar ✅

| Katkı Türü | Açıklama |
|---|---|
| Kaynak doğrulama | Microsoft dokümantasyonu, MITRE ATT&CK ve SigmaHQ referanslarını doğrulama |
| Alan eşleme iyileştirme | SIEM platformlarındaki alan adı farklılıklarını belgeleme |
| Yanlış pozitif notları | Gerçek ortam deneyimlerine dayalı yanlış pozitif örnekleri ekleme |
| Sigma söz dizimi iyileştirme | Sigma spesifikasyonuna uygunluk düzeltmeleri |
| Platform eşleme notları | Splunk, Elastic, Sentinel, Wazuh, QRadar için eşleme rehberi |
| Sentetik/örnek loglar | Laboratuvar ortamında güvenli şekilde üretilmiş test logları |
| SOC runbook iyileştirme | Triage adımları, korelasyon kontrolleri, yükseltme koşulları |
| Savunma odaklı laboratuvar notları | Kuralları güvenli ortamda test etme adımları |

### Yasaklanan Katkılar ❌

| Yasak Katkı | Neden |
|---|---|
| İstismar (exploit) adımları | Saldırı rehberliği |
| Kimlik bilgisi hırsızlığı veya kötüye kullanımı | Saldırı rehberliği |
| Zararlı yazılım çalıştırma | Zararlı içerik |
| Gerçek kurban logları | Gizlilik ihlali |
| Gerçek kimlik bilgileri / tokenlar / sırlar | Güvenlik ihlali |
| Saldırı payload'ları | Zararlı içerik |
| Atlatma / kaçınma (evasion/bypass) talimatları | Saldırı rehberliği |
| Hedefe yönelik saldırı rehberliği | Zararlı içerik |

---

## 12. Açık Konular / Gelecek İyileştirmeler

| # | Konu | Açıklama | Öncelik |
|---|---|---|---|
| 1 | Windows 4688 Process Creation zenginleştirme | Süreç oluşturma olayları ile korelasyon | Yüksek |
| 2 | Sysmon zenginleştirme | Sysmon Event 1, 3, 7, 11 ile korelasyon | Yüksek |
| 3 | EDR zenginleştirme | EDR telemetrisi ile parent/child process, dosya hash, imza | Orta |
| 4 | MDE (Microsoft Defender for Endpoint) entegrasyonu | DeviceEvents tablosu ile korelasyon | Orta |
| 5 | Linux kimlik doğrulama log eşdeğeri | `/var/log/auth.log`, `pam_unix`, `sshd` tabanlı algılamalar | Gelecek paket |
| 6 | Cloud IAM eşdeğeri | AWS CloudTrail, Azure AD, GCP IAM algılamaları | Gelecek paket |
| 7 | SIEM platform dönüşüm doğrulaması | Sentinel KQL, Splunk SPL, Elastic EQL, Wazuh tam dönüşüm testi | Yüksek |
| 8 | Sentetik log test seti | Her kural için laboratuvar ortamında üretilebilir test logları | Orta |
| 9 | Temporal korelasyon kuralları | 4720→4732, 4625→4624, servis kurulumu zincir korelasyonları | Orta |
| 10 | Baseline ölçüm rehberi | Ortam baseline'ı oluşturma metodolojisi | Orta |

---

## 12. Son Bildirim

Bu algılama kuralları **savunma odaklı başlangıç noktalarıdır.** Kurumsal ortamda etkinleştirilmeden önce:

- Yerel log kaynağı ve alan eşleme doğrulaması,
- En az 7–14 günlük baseline ölçümü,
- Eşik değeri ayarlaması,
- Yanlış pozitif analizi ve istisna listesi oluşturma,
- SOC triage iş akışına entegrasyon,
- Sessiz/denetim modunda test,
- SOC ekip onayı

gereklidir.

**Bu paket "enterprise-ready candidate" niteliğindedir — "plug-and-play" üretime hazır çözüm değildir.**

Hiçbir algılama kuralı tek başına tüm saldırıları tespit edemez. Katmanlı savunma (defense in depth) yaklaşımının bir parçası olarak kullanılmalıdır.

---

## 13. Gözden Geçiren Notları (Reviewer Notes)

### Orijinal Taslaktan Yapılan Değişiklikler

| Değişiklik | Detay |
|---|---|
| **Kural 1 — Korelasyon formatı düzeltildi** | Tek satır `count()` mantığı kaldırıldı. Sigma `event_count` ve `value_count` korelasyon tipleri ile ayrı base rule + iki korelasyon kuralı (brute-force ve password-spray) oluşturuldu. `group-by`, `timespan`, `condition` alanları SigmaHQ spesifikasyonuna uygun olarak kullanıldı. |
| **Kural 1 — 4625 bağlam genişletildi** | LogonType önemi, SourceNetworkAddress/IpAddress farklılıkları, WorkstationName sınırlamaları, NAT/VPN/proxy uyarıları, servis hesabı döngüleri, baseline gereksinimi ve SubStatus alanı eklendi. 4624 ve 4740 korelasyon notları eklendi. |
| **Kural 2 — Üretim odaklı hale getirildi** | SubjectUserName/SubjectLogonId, TargetUserName/SamAccountName detayları, etki alanı vs yerel hesap bağlamı, 4722 ve grup ekleme korelasyonu, değişiklik yönetimi/HR onboarding/IAM bağlamı, istisna listesi rehberi ve önem yükseltme koşulları eklendi. |
| **Kural 3 — 4756 eklendi** | Orijinal taslakta yalnızca 4732 ve 4728 vardı. 4756 (evrensel güvenlik grubu) eklendi. Üç Event ID arasındaki farklar açıklandı. |
| **Kural 3 — Grup eşleşme iyileştirildi** | Yerelleştirilmiş grup adı sorunu, yeniden adlandırma riski ve SIEM normalizasyon farklılıkları belgelendi. SID/RID tabanlı eşleşme önerildi ve bilinen SID değerleri listelendi. |
| **Kural 4 — Katmanlı model oluşturuldu** | Orijinal taslakta düz anahtar kelime listesi vardı. Üç katmanlı algılama modeli (Katman 1: düşük güvenilirlikli göstergeler, Katman 2: bağlam zenginleştirme, Katman 3: yüksek güvenilirlikli kombinasyonlar) oluşturuldu. Katman 1'in tek başına alert üretmemesi gerektiği vurgulandı. |
| **Kural 4 — Gürültü azaltıldı** | `Invoke-WebRequest gördüm = alarm` yaklaşımı kaldırıldı. Bağlam tabanlı (yol, kullanıcı, parent process) filtreleme ve korelasyon eklendi. |
| **Kural 5 — 4697 ve 7045 birlikte ele alındı** | Orijinal taslakta 7045 yalnızca not olarak geçiyordu. Her iki Event ID için ayrı Sigma kuralı oluşturuldu. Alan adı farklılıkları (ServiceFileName vs ImagePath) belgelendi. |
| **Altyapı eklendi** | Dağıtım hazırlık kontrol listesi, doğrulama matrisi, SOC runbook'ları, SIEM platform eşleme notları, GitHub katkı rehberi ve açık konular bölümleri eklendi. |

### Şu An Daha Güçlü Olan Kısımlar

- Kural 1: Gerçek Sigma correlation formatında iki ayrı varyant (brute-force + password-spray)
- Kural 3: 4756 dahil, SID tabanlı eşleşme önerisi ile daha güvenilir
- Kural 4: Katmanlı model ile gürültüden korunmuş, hunting-first yaklaşım
- Kural 5: Her iki log kaynağı ayrı kurallarla kapsanmış
- Genel: SOC runbook'ları, doğrulama matrisi ve kontrol listesi ile operasyonel olgunluk

### Hâlâ Yerel Doğrulama Gerektiren Kısımlar

- Tüm alan adları (field names) hedef SIEM platformunda doğrulanmalı
- Sigma korelasyon kurallarının hedef backend dönüşümü test edilmeli
- SID tabanlı eşleşme SIEM'in SID alanlarını işleme biçimine bağlı
- PowerShell Katman 3 korelasyonu EDR/Sysmon entegrasyonu gerektiriyor
- 7045 alan eşlemesi SIEM'e göre farklı olabilir
- Eşik değerleri ortam baseline'ına göre ayarlanmalı
- İstisna listeleri her kurum için farklı olacak

### En Fazla Gürültü Üretmesi Beklenen Algılamalar

1. **Algılama 4 — PowerShell Katman 1:** En yüksek yanlış pozitif potansiyeli. Tek başına alert olarak kullanılmamalı.
2. **Algılama 5 — Servis Kurulumu:** Yazılım dağıtım dönemlerinde yoğun gürültü. Güvenilir yol istisnaları kritik.
3. **Algılama 1 — Başarısız Giriş:** Servis hesabı döngüleri ve NAT/VPN trafiği nedeniyle baseline olmadan gürültülü.

### Hemen Audit/Hunting Modunda Kullanılabilecek Algılamalar

- **Algılama 4 Katman 1** → Hunting sorgusu olarak
- **Algılama 1** → Baseline ölçümü için sessiz modda
- **Algılama 5** → Ortamdaki servis kurulum profilini anlamak için

### Ayarlama Sonrası Alert Kuralı Olabilecek Algılamalar

- **Algılama 3 — Ayrıcalıklı Gruba Ekleme:** En düşük yanlış pozitif potansiyeli; SID tabanlı eşleşme ile hızla alert moduna geçebilir.
- **Algılama 2 — Yeni Hesap Oluşturma:** Provisioning istisnaları eklendikten sonra alert moduna geçebilir.
- **Algılama 1B — Kaba Kuvvet Korelasyonu:** Baseline sonrası eşik ayarlanınca güvenilir sinyal üretir.
- **Algılama 4B — PowerShell Katman 3:** Bağlam korelasyonu kurulduktan sonra orta-yüksek güvenilirlikte.

---

*Bu belge Internal SOC 30 topluluğu tarafından eğitim ve savunma amaçlı hazırlanmıştır. Tüm katkılar savunma odaklı ve güvenli katkı ilkelerine uygun olmalıdır.*

*Son güncelleme: 2026-07-04*
