# Yasal Uyumluluk

Claude Code'u kurumsal ortamda kullanırken legal agreements (yasal sözleşmeler), compliance certifications (uyumluluk sertifikaları), GDPR ve KVKK gibi düzenlemelere dikkat etmek gerekmektedir. Bu rehber, hukuki çerçeveyi ve uyumluluk gereksinimlerini kapsar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Veri güvenliği ve ZDR | [Veri Güvenliği ve ZDR](./08-veri-guvenligi-ve-zdr.md) |
| Güvenlik en iyi uygulamalar | [Güvenlik En İyi Uygulamalar](../10-izinler-ve-guvenlik/05-guvenlik-en-iyi-uygulamalar.md) |

---

## Hukuki Çerçeve Genel Bakış

Claude Code kullanımı birden fazla hukuki alan ile ilişkilidir:

```mermaid
flowchart TD
    CC["Claude Code\nKurumsal Kullanım"] --> CONTRACTS["📝 Sözleşmeler\nKullanım şartları, SLA"]
    CC --> PRIVACY["🔒 Gizlilik\nGDPR, KVKK"]
    CC --> IP["⚖️ Fikri Mülkiyet\nÜretilen kodun hakları"]
    CC --> SECURITY["🛡️ Güvenlik\nSOC 2, ISO 27001"]
    CC --> INDUSTRY["🏭 Sektörel\nFinans, sağlık, kamu"]

    style CC fill:#2C3E50,color:#fff
    style CONTRACTS fill:#3498DB,color:#fff
    style PRIVACY fill:#E74C3C,color:#fff
    style IP fill:#F39C12,color:#fff
    style SECURITY fill:#27AE60,color:#fff
    style INDUSTRY fill:#9B59B6,color:#fff
```

---

## Anthropic Sözleşme Yapısı

### Sözleşme Türleri

```mermaid
flowchart LR
    subgraph agreements ["Sözleşme Türleri"]
        TOS["Terms of Service\n(Kullanım Şartları)\nBireysel kullanıcılar"]
        API_TOS["API Terms\n(API Şartları)\nAPI kullanıcıları"]
        ENTERPRISE["Enterprise Agreement\n(Kurumsal Sözleşme)\nBüyük organizasyonlar"]
    end

    TOS --> CLAUDE_AI["claude.ai"]
    API_TOS --> API_USAGE["Claude Code\n(API üzerinden)"]
    ENTERPRISE --> CUSTOM["Özelleştirilmiş\nkurumsal kullanım"]

    style TOS fill:#3498DB,color:#fff
    style API_TOS fill:#F39C12,color:#fff
    style ENTERPRISE fill:#8E44AD,color:#fff
```

### Claude Code İçin Geçerli Sözleşme

Claude Code, API Terms kapsamında çalışır. Kurumsal müşteriler için Enterprise Agreement uygulanabilir:

| Sözleşme | Kapsam | Taraf |
|----------|--------|-------|
| API Terms of Service | API kullanımı, Claude Code dahil | Bireysel / küçük takımlar |
| Enterprise Agreement | Özelleştirilmiş kurumsal koşullar | Büyük organizasyonlar |
| ZDR Addendum | Zero Data Retention ek sözleşmesi | ZDR talep edenler |
| BAA | Business Associate Agreement (HIPAA) | Sağlık sektörü |

---

## Güvenlik Sertifikaları

Anthropic'in sahip olduğu güvenlik sertifikaları:

```mermaid
flowchart TD
    subgraph certs ["Güvenlik Sertifikaları"]
        SOC2["SOC 2 Type II\nGüvenlik, erişilebilirlik,\ngizlilik kontrolleri"]
        CSA["CSA STAR\nCloud güvenlik\ndeğerlendirmesi"]
        PENTEST["Penetration Testing\nDüzenli güvenlik\ntestleri"]
    end

    subgraph practices ["Güvenlik Uygulamaları"]
        ENCRYPT["Şifreleme\nTransit + At-rest\nTLS 1.3, AES-256"]
        ACCESS["Erişim Kontrolü\nRole-based access\nMFA zorunlu"]
        MONITOR["İzleme\n7/24 güvenlik\nmonitoring"]
        INCIDENT["Olay Müdahale\nIncident response\nplanı"]
    end

    certs --> practices

    style SOC2 fill:#27AE60,color:#fff
    style CSA fill:#3498DB,color:#fff
    style PENTEST fill:#F39C12,color:#fff
```

### Sertifika Durumları

| Sertifika | Durum | Kapsam |
|-----------|-------|--------|
| SOC 2 Type II | ✅ Aktif | Güvenlik, erişilebilirlik, işlem bütünlüğü, gizlilik |
| CSA STAR | ✅ Aktif | Cloud güvenlik değerlendirmesi |
| Penetration Testing | ✅ Düzenli | Uygulama ve altyapı güvenliği |
| ISO 27001 | Bilgi için iletişime geçin | Bilgi güvenliği yönetim sistemi |
| HIPAA | BAA ile | Sağlık verileri (ABD) |

---

## GDPR Uyumluluğu

GDPR (General Data Protection Regulation / Genel Veri Koruma Tüzüğü), Avrupa Birliği'nin kişisel veri koruma düzenlemesidir.

### GDPR Kapsamında Claude Code

```mermaid
flowchart TD
    GDPR["GDPR Gereksinimleri"] --> LAWFUL["Yasal Dayanak\nVeri işleme için hukuki temel"]
    GDPR --> PURPOSE["Amaç Sınırlaması\nBelirlenen amaçla sınırlı"]
    GDPR --> MINIMIZE["Veri Minimizasyonu\nGerekli minimum veri"]
    GDPR --> RIGHTS["Veri Sahibi Hakları\nSilme, erişim, düzeltme"]
    GDPR --> TRANSFER["Veri Transferi\nAB dışına aktarım kuralları"]
    GDPR --> DPA["DPA\nVeri İşleme Sözleşmesi"]

    style GDPR fill:#003399,color:#fff
    style DPA fill:#E74C3C,color:#fff
```

### GDPR Kontrol Listesi

| Gereksinim | Claude Code'da Karşılığı |
|------------|--------------------------|
| Yasal dayanak | Meşru menfaat veya sözleşme yükümlülüğü |
| Veri minimizasyonu | Yalnızca gerekli dosyaları Claude Code'a gönderin |
| Veri işleme sözleşmesi (DPA) | Anthropic DPA mevcut (Enterprise) |
| Veri transferi | SCCs (Standard Contractual Clauses) uygulanır |
| Veri sahibi hakları | ZDR ile veri saklanmadığında uyumluluk kolaylaşır |
| Veri koruma etki değerlendirmesi (DPIA) | Kişisel veri işleniyorsa DPIA yapılmalı |

### DPA (Data Processing Agreement)

Anthropic, kurumsal müşterilerine DPA (Veri İşleme Sözleşmesi) sunmaktadır:

```
Anthropic DPA kapsamı:
- Veri işleme amaçları ve sınırları
- Alt işleyiciler (sub-processors) listesi
- Veri güvenliği önlemleri
- İhlal bildirim prosedürleri
- Veri sahibi hakları desteği
- Uluslararası veri transferi mekanizmaları
```

---

## KVKK Uyumluluğu

KVKK (Kişisel Verilerin Korunması Kanunu), Türkiye'nin kişisel veri koruma düzenlemesidir ve GDPR ile büyük ölçüde benzerdir.

### KVKK Kapsamında Dikkat Edilmesi Gerekenler

```mermaid
flowchart TD
    KVKK["KVKK Gereksinimleri"] --> EXPLICIT["Açık Rıza\nKişisel veri işleme için"]
    KVKK --> REGISTER["VERBİS Kaydı\nVeri sorumlusu sicili"]
    KVKK --> CROSS_BORDER["Yurt Dışı Aktarım\nYeterli koruma veya taahhüt"]
    KVKK --> SECURITY_KVKK["Güvenlik Önlemleri\nTeknik ve idari tedbirler"]
    KVKK --> NOTIFY["İhlal Bildirimi\n72 saat içinde KVK Kurulu'na"]

    style KVKK fill:#E30A17,color:#fff
```

### KVKK Kontrol Listesi

| Gereksinim | Aksiyon |
|------------|---------|
| Kişisel veri tespiti | Claude Code'a gönderilen veride kişisel veri var mı? |
| Açık rıza | Gerekiyorsa çalışanlardan aydınlatma ve onay |
| Yurt dışı aktarım | ABD'ye veri transferi için yeterli koruma veya taahhütname |
| VERBİS kaydı | Veri sorumlusu olarak kayıt kontrolü |
| Teknik tedbirler | Şifreleme, erişim kontrolü, log tutma |
| İdari tedbirler | Politika, eğitim, gizlilik sözleşmesi |

### Yurt Dışı Veri Transferi

Claude Code kullanıldığında veriler Anthropic sunucularına (ABD) aktarılır. KVKK açısından:

```mermaid
flowchart LR
    TR["🇹🇷 Türkiye\nKaynak veri"] --> TRANSFER{"Yurt dışı\naktarım?"}
    TRANSFER -->|"Doğrudan API"| US["🇺🇸 ABD\nAnthropic sunucuları"]
    TRANSFER -->|"Bedrock"| AWS_REGION["☁️ AWS\nSeçilen bölge"]
    TRANSFER -->|"Vertex AI"| GCP_REGION["☁️ GCP\nSeçilen bölge"]

    US --> MECHANISM["Aktarım Mekanizması:\n- Yeterli koruma kararı\n- Taahhütname\n- Açık rıza"]

    style TR fill:#E30A17,color:#fff
    style US fill:#3C3B6E,color:#fff
    style MECHANISM fill:#F39C12,color:#fff
```

**Öneriler:**

1. **Bedrock/Vertex AI kullanımı** — Veriler seçtiğiniz bölgede kalabilir
2. **ZDR etkinleştirme** — Veri saklanmadığında transfer riski azalır
3. **Hukuk danışmanlığı** — Organizasyonunuza özel değerlendirme yaptırın
4. **Kişisel veri minimizasyonu** — Claude Code'a kişisel veri göndermekten kaçının

---

## Fikri Mülkiyet (IP) Hakları

### Claude Code ile Üretilen Kod

| Soru | Cevap |
|------|-------|
| Üretilen kodun sahibi kim? | **Siz.** API Terms'e göre output'lar müşteriye aittir |
| Anthropic üretilen kodu kullanabilir mi? | Hayır (API kullanımında model eğitimi için kullanılmaz) |
| Lisans kısıtlaması var mı? | Hayır, üretilen kodu dilediğiniz gibi kullanabilirsiniz |
| Üçüncü taraf IP riski var mı? | Olası, ancak düşük risk (genel sorumluluk kullanıcıda) |

### IP Koruma Önerileri

```mermaid
flowchart TD
    IP_PROTECT["Fikri Mülkiyet Koruma"] --> REVIEW["Kod İnceleme\nÜretilen kodu review edin"]
    IP_PROTECT --> LICENSE["Lisans Kontrolü\nAçık kaynak lisans uyumu"]
    IP_PROTECT --> DOCUMENT["Belgeleme\nHangi kodun AI ile üretildiğini belirtin"]
    IP_PROTECT --> POLICY["Politika\nAI kullanım politikası oluşturun"]

    style IP_PROTECT fill:#F39C12,color:#fff
```

---

## Sektörel Uyumluluk

| Sektör | Düzenleme | Claude Code Değerlendirmesi |
|--------|-----------|----------------------------|
| Finans | SPK, BDDK, PCI-DSS | ZDR + Bedrock/Vertex önerilir |
| Sağlık | HIPAA (ABD), sağlık verileri | BAA gerekli, ZDR zorunlu |
| Kamu | Kamu verileri mevzuatı | Hukuk danışmanlığı gerekli |
| Savunma | ITAR, EAR | Detaylı değerlendirme gerekli |
| Eğitim | FERPA (ABD), kişisel veriler | Öğrenci verisi kullanılmamalı |

---

## Pratik Örnek: Uyumluluk Dosyası

Organizasyonunuz için bir AI uyumluluk dosyası oluşturun:

```markdown
# AI Araç Kullanım Politikası

## Kapsam
Bu politika, [Şirket Adı] çalışanlarının Claude Code 
ve benzeri AI kodlama araçlarını kullanımını düzenler.

## İzin Verilen Kullanımlar
- Açık kaynak ve iç proje kodu geliştirme
- Kod inceleme ve refactoring
- Test yazma ve hata ayıklama
- Dökümantasyon oluşturma

## Yasaklanan Kullanımlar
- Kişisel veri içeren dosyaların AI'ya gönderilmesi
- Müşteri credential'larının paylaşılması
- Sınıflandırılmış/gizli verilerin işlenmesi
- Lisanslı üçüncü taraf kodunun doğrudan kopyalanması

## Teknik Önlemler
- ZDR etkinleştirilmiştir
- Hassas dosya filtreleme hook'u aktiftir
- Tüm oturumlar denetim loglarına kaydedilir
- Managed settings ile güvenlik politikaları uygulanır

## Sorumluluk
- Üretilen tüm kod, teslim öncesi insan tarafından review edilmelidir
- AI ile üretilen kodun sorumluluğu geliştirici ve ekibindedir
```

---

## Uyumluluk Karar Ağacı

```mermaid
flowchart TD
    START["Claude Code\nkullanmak istiyorum"] --> PII{"Kişisel veri\nişlenecek mi?"}

    PII -->|Evet| GDPR_KVKK["GDPR/KVKK\nuyumluluk kontrolü"]
    PII -->|Hayır| SECTOR{"Sektörel\ndüzenleme var mı?"}

    GDPR_KVKK --> DPA_CHECK{"DPA\nimzalandı mı?"}
    DPA_CHECK -->|Evet| ZDR_CHECK{"ZDR\netkin mi?"}
    DPA_CHECK -->|Hayır| GET_DPA["DPA talep edin"]

    SECTOR -->|Evet| SECTOR_CHECK["Sektörel uyumluluk\ndeğerlendirmesi"]
    SECTOR -->|Hayır| READY["✅ Kullanıma hazır"]

    ZDR_CHECK -->|Evet| READY
    ZDR_CHECK -->|Hayır| EVALUATE["ZDR gerekli mi?\ndeğerlendirin"]

    GET_DPA --> DPA_CHECK
    SECTOR_CHECK --> ZDR_CHECK

    style START fill:#3498DB,color:#fff
    style READY fill:#27AE60,color:#fff
    style GET_DPA fill:#E74C3C,color:#fff
```

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| Yasal değerlendirme yapmadan kullanmak | Hukuk ekibinizle değerlendirme yapın |
| KVKK yurt dışı transfer kurallarını görmezden gelmek | Aktarım mekanizması belirleyin |
| DPA imzalamamak | Enterprise plan ile DPA talep edin |
| AI politikası oluşturmamak | Yazılı AI kullanım politikası hazırlayın |
| Üretilen kodu review etmemek | Tüm AI çıktılarını insan gözünden geçirin |

---

## Önemli Linkler

| Kaynak | URL |
|--------|-----|
| Anthropic Privacy Policy | [anthropic.com/privacy](https://www.anthropic.com/privacy) |
| Anthropic Terms of Service | [anthropic.com/terms](https://www.anthropic.com/terms) |
| Anthropic Security | [anthropic.com/security](https://trust.anthropic.com) |
| KVKK Kurumu | [kvkk.gov.tr](https://www.kvkk.gov.tr) |

---

## Özet

| Konu | Anahtar Bilgi |
|------|---------------|
| Sözleşme | API Terms (bireysel), Enterprise Agreement (kurumsal) |
| Sertifikalar | SOC 2 Type II, CSA STAR |
| GDPR | DPA mevcut, SCCs uygulanır |
| KVKK | Yurt dışı transfer değerlendirmesi gerekli |
| IP hakları | Üretilen kodun hakları müşteriye ait |
| Öneriler | ZDR + DPA + AI politikası + hukuk danışmanlığı |

---

## Sonraki Adım

Kurumsal kullanım bölümünü tamamladınız! Şimdi farklı roller için özel rehberlere geçelim:

→ [Bölüm 19: Rol Bazlı Rehberler](../19-rol-bazli-rehberler/README.md)
