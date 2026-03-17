# Analitik ve Metrikler

Claude Code'un kurumsal ortamda etkin kullanılıp kullanılmadığını ölçmek için analytics (analitik) ve metrics (metrikler) takibi kritik öneme sahiptir. Bu rehber, kullanım verilerinin toplanması, adopsiyon (benimseme) takibi ve engineering velocity (mühendislik hızı) ölçümünü kapsar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Takım kullanımı | [Takım Kullanımı ve Yönetim](./01-takim-kullanimi-ve-yonetim.md) |
| Maliyet yönetimi | [Maliyet Yönetimi](../17-konfigurasyon/05-maliyet-yonetimi.md) |

---

## Analitik Genel Bakış

```mermaid
flowchart TD
    subgraph data ["📊 Veri Kaynakları"]
        USAGE["Kullanım Verileri<br/>Oturum sayısı, süre, token"]
        COST["Maliyet Verileri<br/>Harcama, model dağılımı"]
        ADOPTION["Adopsiyon Verileri<br/>Aktif kullanıcı, sıklık"]
        VELOCITY["Verimlilik Verileri<br/>Commit, PR, döngü süresi"]
    end

    subgraph dashboard ["📈 Analytics Dashboard"]
        OVERVIEW["Genel Bakış"]
        DETAILS["Detay Raporlar"]
        TRENDS["Trendler"]
        ALERTS["Uyarılar"]
    end

    data --> dashboard

```

---

## Kullanım Metrikleri

### Temel Kullanım Metrikleri

| Metrik | Açıklama | Nasıl Ölçülür |
|--------|----------|---------------|
| Aktif kullanıcı sayısı | Belirli dönemde Claude Code kullanan kişi sayısı | DAU/WAU/MAU |
| Oturum sayısı | Toplam başlatılan oturum | Günlük/haftalık/aylık |
| Ortalama oturum süresi | Bir oturumun ortalama süresi | Dakika cinsinden |
| Token tüketimi | Input + output token toplamı | Model bazlı ayrım |
| Araç kullanım dağılımı | Hangi araçlar ne sıklıkla kullanılıyor | Araç bazlı sayaç |
| Turn sayısı | Oturum başına ortalama etkileşim | Mesaj sayısı |

### Kullanım Trendi Analizi

```mermaid
flowchart LR
    subgraph weekly ["Haftalık Trend"]
        direction TB
        W1["Hafta 1<br/>15 oturum/gün"]
        W2["Hafta 2<br/>22 oturum/gün"]
        W3["Hafta 3<br/>28 oturum/gün"]
        W4["Hafta 4<br/>35 oturum/gün"]
    end

    W1 --> W2 --> W3 --> W4

    W4 --> TREND["📈 Artan Trend<br/>Adopsiyon başarılı"]

```

---

## Adopsiyon Takibi

Organizasyondaki Claude Code benimseme oranını ölçmek:

```mermaid
flowchart TD
    subgraph adoption ["Adopsiyon Aşamaları"]
        PILOT["🧪 Pilot<br/>5-10 kişi<br/>Değerlendirme"]
        EARLY["🌱 Erken Adopsiyon<br/>%20 ekip<br/>Öncüler"]
        GROWTH["📈 Büyüme<br/>%50 ekip<br/>Yaygınlaşma"]
        MATURE["🏆 Olgunluk<br/>%80+ ekip<br/>Standart araç"]
    end

    PILOT --> EARLY --> GROWTH --> MATURE

```

### Adopsiyon Metrikleri

| Metrik | Formül | Hedef |
|--------|--------|-------|
| Adopsiyon oranı | Aktif kullanıcı / Toplam geliştirici × 100 | %80+ |
| Haftalık aktif oran | WAU / Toplam lisanslı × 100 | %60+ |
| Retention (elde tutma) | Bu hafta aktif ∩ Geçen hafta aktif / Geçen hafta aktif × 100 | %90+ |
| Derinlik | Ortalama oturum/gün/kullanıcı | 3+ |
| Genişlik | Claude Code kullanan proje sayısı / Toplam proje | %70+ |

---

## Engineering Velocity (Mühendislik Hızı) Ölçümü

Claude Code'un geliştirme verimliliğine etkisini ölçmek:

```mermaid
flowchart TD
    subgraph before ["Claude Code Öncesi"]
        B_CYCLE["Döngü süresi: 5 gün"]
        B_PR["PR/hafta: 3"]
        B_REVIEW["Review süresi: 24 saat"]
        B_BUG["Bug oranı: %8"]
    end

    subgraph after ["Claude Code Sonrası"]
        A_CYCLE["Döngü süresi: 3 gün"]
        A_PR["PR/hafta: 6"]
        A_REVIEW["Review süresi: 12 saat"]
        A_BUG["Bug oranı: %5"]
    end

    before --> COMPARE["📊 Karşılaştırma"]
    after --> COMPARE

    COMPARE --> RESULT["✅ %40 hız artışı<br/>%100 PR artışı<br/>%50 review iyileşme<br/>%37 hata azalma"]

```

### Verimlilik Metrikleri

| Metrik | Ölçüm | Claude Code Öncesi/Sonrası Karşılaştırma |
|--------|-------|------------------------------------------|
| Cycle time (döngü süresi) | Issue'dan deploy'a kadar geçen süre | Azalma beklenir |
| PR throughput | Haftalık merge edilen PR sayısı | Artış beklenir |
| Code review süresi | PR açılmasından onaylanmasına | Azalma beklenir |
| Time to first commit | Bir göreve başlamadan ilk commit'e | Azalma beklenir |
| Bug rate | Üretimdeki hata sayısı / toplam commit | Azalma beklenir |
| Test coverage | Otomatik test kapsamı yüzdesi | Artış beklenir |

---

## Analytics Dashboard Yapılandırma

### Hook ile Metrik Toplama

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "https://analytics.company.com/api/claude-session",
            "timeout": 5000
          }
        ]
      }
    ],
    "SessionStop": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "https://analytics.company.com/api/claude-session-end",
            "timeout": 5000
          }
        ]
      }
    ]
  }
}
```

### OpenTelemetry ile Metrik Gönderme

Detaylı telemetri için OpenTelemetry entegrasyonu kullanılabilir (bir sonraki bölümde detaylı anlatılacak).

```mermaid
flowchart LR
    CC["Claude Code<br/>Oturumları"] --> OTEL["OpenTelemetry<br/>Collector"]
    OTEL --> BACKEND["Analytics Backend<br/>(Datadog, Grafana, vb.)"]
    BACKEND --> DASHBOARD["📊 Dashboard<br/>Görselleştirme"]

```

---

## Raporlama Şablonları

### Haftalık Kullanım Raporu

| Metrik | Bu Hafta | Geçen Hafta | Değişim |
|--------|----------|-------------|---------|
| Aktif kullanıcı | 42 | 38 | +10.5% |
| Toplam oturum | 312 | 278 | +12.2% |
| Ort. oturum süresi | 18 dk | 15 dk | +20.0% |
| Toplam token | 4.2M | 3.8M | +10.5% |
| Toplam harcama | $847 | $723 | +17.1% |
| Oturum başı maliyet | $2.71 | $2.60 | +4.2% |

### Aylık ROI (Return on Investment) Raporu

```mermaid
flowchart TD
    COST["💰 Aylık Maliyet<br/>Lisans: $2,000<br/>API: $3,400<br/>Toplam: $5,400"] --> ROI["📊 ROI Hesabı"]

    SAVINGS["💎 Aylık Kazanım<br/>Zaman tasarrufu: 320 saat<br/>Saat değeri: $75<br/>Toplam: $24,000"] --> ROI

    ROI --> RESULT["✅ Net ROI<br/>$24,000 - $5,400 = $18,600<br/>ROI: %344"]

```

---

## Pratik Örnek: Metrik Toplama Pipeline

```mermaid
sequenceDiagram
    participant DEV as Geliştirici
    participant CC as Claude Code
    participant HOOK as Session Hook
    participant API as Analytics API
    participant DB as Veritabanı
    participant DASH as Dashboard

    DEV->>CC: Oturum başlat
    CC->>HOOK: SessionStart event
    HOOK->>API: POST /session-start
    API->>DB: Oturum kaydı oluştur

    DEV->>CC: Görevler yap (N turn)
    
    DEV->>CC: Oturum kapat
    CC->>HOOK: SessionStop event
    HOOK->>API: POST /session-end (maliyet, turn, süre)
    API->>DB: Oturum kaydını güncelle

    DB->>DASH: Veri sorgulama
    DASH->>DASH: Grafik güncelle
```

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| Sadece kullanım metriklerine bakmak | Verimlilik metriklerini de (PR, cycle time) takip edin |
| ROI hesaplamamak | Maliyet vs tasarruf karşılaştırması yapın |
| Bireysel performansı ölçmek | Takım metrikleri odaklanın, bireysel sıralamadan kaçının |
| Veriyi toplamama | Hook'lar ile otomatik telemetri kurun |

---

## Özet

| Alan | Anahtar Metrikler |
|------|-------------------|
| Kullanım | Aktif kullanıcı, oturum sayısı, token tüketimi |
| Adopsiyon | Adopsiyon oranı, retention, derinlik |
| Verimlilik | Cycle time, PR throughput, bug rate |
| Maliyet | Toplam harcama, oturum başı maliyet, ROI |

---

## Sonraki Adım

OpenTelemetry entegrasyonu ile detaylı izleme ve performans monitoring'i kurmak için:

→ [İzleme ve OpenTelemetry](./03-izleme-ve-opentelemetry.md)
