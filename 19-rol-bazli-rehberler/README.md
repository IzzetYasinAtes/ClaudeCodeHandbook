# Bölüm 19: Birim ve Pozisyon Bazlı Kullanım Rehberleri

Claude Code'un farklı birimlerdeki günlük iş akışlarına nasıl entegre edilebileceğini gösteren 12 pozisyon rehberi. Her rehber, ilgili pozisyona özel kullanım senaryoları, örnek prompt'lar ve en iyi uygulamaları içerir.

---

## İçerik

| Birim | # | Dosya | Pozisyon | Süre |
|-------|---|-------|----------|------|
| **Teknik** | 01 | [Yazılım Geliştirici](./01-teknik-yazilim-gelistirici.md) | Yazılım Geliştirici | ~25 dk |
| **Teknik** | 02 | [QA / Test](./02-teknik-qa-test.md) | QA / Test | ~18 dk |
| **Teknik** | 03 | [Sistem Uzmanı](./03-teknik-sistem-uzmani.md) | Sistem Uzmanı | ~20 dk |
| **Teknik** | 04 | [UI/UX](./04-teknik-ui-ux.md) | UI/UX | ~18 dk |
| **Ürün & Analiz** | 05 | [Ürün Müdürü](./05-urun-urun-muduru.md) | Ürün Müdürü | ~18 dk |
| **Ürün & Analiz** | 06 | [İş Analisti](./06-urun-is-analisti.md) | İş Analisti | ~20 dk |
| **Ürün & Analiz** | 07 | [Proje Yöneticisi](./07-urun-proje-yoneticisi.md) | Proje Yöneticisi | ~18 dk |
| **Ticari** | 08 | [Satış](./08-ticari-satis.md) | Satış | ~15 dk |
| **Ticari** | 09 | [Pazarlama](./09-ticari-pazarlama.md) | Pazarlama | ~15 dk |
| **Operasyon** | 10 | [İK](./10-operasyon-ik.md) | İK | ~15 dk |
| **Operasyon** | 11 | [Finans](./11-operasyon-finans.md) | Finans | ~15 dk |
| **Operasyon** | 12 | [Yönetim](./12-operasyon-yonetim.md) | Yönetim | ~15 dk |

---

## Hangi Birim Nereden Başlamalı?

```mermaid
flowchart TD
    START{"Hangi birimde<br/>çalışıyorsunuz?"}

    START -->|"Teknik"| TECH{"Claude Code<br/>deneyiminiz var mı?"}
    TECH -->|"Evet"| TECH_EXP["Doğrudan<br/>Bölüm 19'a geçin"]
    TECH -->|"Hayır"| TECH_NEW["Bölüm 06'dan<br/>başlayın"]

    START -->|"Ürün & Analiz"| PROD{"Yapay zeka<br/>temellerini biliyor<br/>musunuz?"}
    PROD -->|"Evet"| PROD_EXP["Bölüm 04'ten<br/>başlayın"]
    PROD -->|"Hayır"| PROD_NEW["Bölüm 01'den<br/>başlayın"]

    START -->|"Ticari"| COMM["Bölüm 01'den<br/>başlayın"]

    START -->|"Operasyon"| OPS{"Yapay zeka<br/>temellerini biliyor<br/>musunuz?"}
    OPS -->|"Evet"| OPS_EXP["Bölüm 17'den<br/>başlayın"]
    OPS -->|"Hayır"| OPS_NEW["Bölüm 01'den<br/>başlayın"]
```

---

## Ön Koşullar

Bu bölümü okumadan önce aşağıdaki konulara aşina olmanız önerilir:

| Konu | Bölüm | Gereklilik |
|------|-------|------------|
| Yapay zeka temel kavramları | [Bölüm 01](../01-yapay-zeka-temelleri/README.md) | Tüm birimler |
| AI destekli geliştirme yaklaşımları | [Bölüm 04](../04-ai-destekli-gelistirme/README.md) | Tüm birimler |
| Claude Code nasıl çalışır | [Bölüm 06](../06-claude-code-tanitim/README.md) | Teknik birim |
| Arayüz ve komutlar | [Bölüm 07](../07-arayuz-ve-komutlar/README.md) | Teknik birim |
| Araçlar (Tools) | [Bölüm 08](../08-araclar/README.md) | Teknik, Ürün & Analiz |
| Bellek ve bağlam yönetimi | [Bölüm 09](../09-bellek-ve-baglam/README.md) | Teknik birim |

---

## Önceki Bölüm

← [18 - Kurumsal Kullanım](../18-kurumsal-kullanim/README.md)

## Sonraki Bölüm

→ [20 - Pratik Senaryolar ve Tarifler](../20-pratik-senaryolar/README.md)
