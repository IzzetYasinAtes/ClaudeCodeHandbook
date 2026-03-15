# Context Window Yönetimi

**Context window** (bağlam penceresi) yönetimi, Claude Code kullanımında edinmeniz gereken **en kritik beceri**dir. Context window dolmaya başladığında performans düşer, talimatlar göz ardı edilir ve hatalar artar. Bu bölümde token yönetimini, performans eğrisini ve pratik stratejileri öğreneceksiniz.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code nasıl çalışır | [Claude Code Nasıl Çalışır?](../06-claude-code-tanitim/02-claude-code-nasil-calisir.md) |
| CLAUDE.md dosyası | [CLAUDE.md Dosyası](./01-claude-md-dosyasi.md) |

---

## Context Window Nedir?

Context window, Claude'un bir oturumda aynı anda "görebildiği" bilgi miktarıdır. Claude Code 200.000 token (~150.000 kelime) kapasiteye sahiptir. Bu kapasite sabit olup, oturum boyunca şunlarla dolar:

```mermaid
flowchart TD
    subgraph window ["200K Token Context Window"]
        direction TB
        SYS["Sistem Talimatları\n~5K token"]
        MEM["CLAUDE.md + Auto Memory\nDeğişken (~2-5K)"]
        CONV["Konuşma Geçmişi\n(Her mesajla büyür)"]
        TOOL["Araç Çıktıları\n(Dosya okuma, komut sonuçları)"]
        FREE["Kullanılabilir Alan\n(Giderek azalır)"]
    end

    SYS --> MEM --> CONV --> TOOL --> FREE

    style SYS fill:#95A5A6,color:#fff
    style MEM fill:#3498DB,color:#fff
    style CONV fill:#F39C12,color:#fff
    style TOOL fill:#E67E22,color:#fff
    style FREE fill:#27AE60,color:#fff
```

---

## %80 Kuralı

Context window %80'e ulaştığında oturumu yeniden başlatmanız önerilir. Bu, performans düşüşünü önlemenin en basit kuralıdır.

```mermaid
flowchart LR
    subgraph gauge ["Context Window Doluluk"]
        direction TB
        G1["0-25%: ✅ Optimal\nEn iyi performans"]
        G2["25-50%: ✅ İyi\nNormal çalışma"]
        G3["50-73%: ⚠️ Dikkat\nPerformans azalmaya başlar"]
        G4["73-80%: 🔶 Uyarı\n/compact kullanın"]
        G5["80%+: 🔴 Kritik\nYeni oturum başlatın"]
    end

    G1 --> G2 --> G3 --> G4 --> G5

    style G1 fill:#27AE60,color:#fff
    style G2 fill:#2ECC71,color:#fff
    style G3 fill:#F39C12,color:#fff
    style G4 fill:#E67E22,color:#fff
    style G5 fill:#E74C3C,color:#fff
```

---

## Performans Degradasyon Eğrisi

Context window doluluk oranı ile Claude Code performansı arasında doğrudan bir ilişki vardır:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '14px'}}}%%
xychart-beta
    title "Context Window Doluluk vs Performans"
    x-axis "Doluluk (%)" [0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
    y-axis "Performans (%)" 0 --> 100
    line "Performans" [95, 95, 93, 90, 85, 78, 70, 60, 45, 30, 15]
```

### Performans Aşamaları

| Doluluk Aralığı | Performans | Belirtiler | Eylem |
|-----------------|-----------|------------|-------|
| **0-25%** | ⭐ Pik performans | Talimatlar eksiksiz takip edilir, yaratıcı çözümler | Normal çalışma |
| **25-50%** | ✅ İyi | Hafif sapma olabilir ama genel kalite yüksek | Normal çalışma |
| **50-73%** | ⚠️ Azalan | CLAUDE.md kuralları ara sıra göz ardı edilebilir | `/compact` kullanmayı düşünün |
| **73-80%** | 🔶 Düşük | Talimat takibi belirgin şekilde azalır, tekrarlar artar | `/compact` kullanın |
| **80%+** | 🔴 Kritik | Kurallar sıkça göz ardı edilir, hatalar artar, döngüler oluşur | Yeni oturum başlatın |

---

## Token Sayacını İzleme

Claude Code, **status bar** (durum çubuğu) üzerinde token kullanımını gösterir:

```
> claude
╭──────────────────────────────────────────╮
│ Claude Code                    ◼ 45K/200K│
╰──────────────────────────────────────────╯
```

### Token Sayacı Okuma Rehberi

```mermaid
flowchart LR
    COUNTER["Token Sayacı:\n45K / 200K"] --> PERCENT["Doluluk:\n%22.5"]
    PERCENT --> STATUS{"Durum?"}
    STATUS -->|"< %50"| GREEN["✅ Güvenli\nDevam edin"]
    STATUS -->|"%50-73"| YELLOW["⚠️ Dikkat\nKarmaşık görevleri\nbitirin"]
    STATUS -->|"%73-80"| ORANGE["🔶 Uyarı\n/compact çalıştırın"]
    STATUS -->|"> %80"| RED["🔴 Yeniden\nbaşlatın"]

    style GREEN fill:#27AE60,color:#fff
    style YELLOW fill:#F39C12,color:#fff
    style ORANGE fill:#E67E22,color:#fff
    style RED fill:#E74C3C,color:#fff
```

---

## /compact Komutu

`/compact` komutu, konuşma geçmişini özetleyerek context window'da yer açar:

```bash
# Temel kullanım
> /compact

# Özel odak noktasıyla compact
> /compact auth modülündeki değişikliklere odaklan
```

### /compact Nasıl Çalışır?

```mermaid
flowchart TD
    BEFORE["Mevcut Bağlam\n150K / 200K (%75)"]
    COMPACT["/compact komutu"]
    PROCESS["Konuşma geçmişi\nözetlenir"]
    AFTER["Sıkıştırılmış Bağlam\n~40K / 200K (%20)"]

    BEFORE --> COMPACT --> PROCESS --> AFTER

    subgraph korunan ["Korunan Bilgiler"]
        K1["CLAUDE.md kuralları"]
        K2["Önemli kararlar"]
        K3["Son yapılan değişiklikler"]
        K4["Aktif görev bağlamı"]
    end

    subgraph silinen ["Özetlenen / Silinen"]
        S1["Eski dosya okumaları"]
        S2["Tamamlanmış ara adımlar"]
        S3["Tekrarlı araç çıktıları"]
        S4["Ayrıntılı hata geçmişi"]
    end

    PROCESS --> korunan
    PROCESS --> silinen

    style BEFORE fill:#E74C3C,color:#fff
    style AFTER fill:#27AE60,color:#fff
```

### /compact Ne Zaman Kullanılmalı?

| Durum | /compact Önerisi |
|-------|------------------|
| Token sayacı %73+ | ✅ Hemen kullanın |
| Uzun oturumda ara verirken | ✅ Bağlamı temizleyin |
| Farklı bir göreve geçerken | ✅ Önceki bağlamı özetletin |
| Kısa, odaklı bir oturumdaysanız | ❌ Gerek yok |
| Kritik bir görevin ortasındaysanız | ⚠️ Dikkatli olun, görev bağlamı kaybolabilir |

---

## /context Komutu

`/context` komutu, context window kullanımını renkli bir ızgara olarak görselleştirir:

```bash
> /context
```

### Örnek Çıktı

```
Context usage: 67,340 / 200,000 tokens (33.7%)

████████████████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░

  System:       ██ 5,200 (2.6%)
  CLAUDE.md:    ██ 3,400 (1.7%)
  Conversation: ████████████ 35,200 (17.6%)
  Tool outputs: ████████ 23,540 (11.8%)
  Available:    ░░░░░░░░░░░░░░░░░░░░░░░░ 132,660 (66.3%)
```

```mermaid
flowchart LR
    subgraph grid ["Context Kullanım Izgarası"]
        direction TB
        SYS["🟦 System: %2.6"]
        CMD["🟩 CLAUDE.md: %1.7"]
        CONV["🟨 Konuşma: %17.6"]
        TOOL["🟧 Araç çıktıları: %11.8"]
        FREE["⬜ Boş: %66.3"]
    end

    CONTEXT_CMD["/context\nkomutu"] --> grid
```

---

## Görev Parçalama (Task Chunking) Stratejileri

Büyük görevleri tek bir oturumda yapmaya çalışmak context window'u hızla doldurur. Bunun yerine görevleri parçalayın:

```mermaid
flowchart TD
    subgraph kotu ["❌ Kötü: Tek Oturumda Her Şey"]
        direction TB
        B1["Oturum 1:\n1. Proje yapısını oluştur\n2. 10 dosya yaz\n3. Testleri yaz\n4. CI/CD kur\n5. Dokümantasyon"]
        B1 --> OVERFLOW["💥 Context window doldu!\nPerformans çöktü"]
    end

    subgraph iyi ["✅ İyi: Parçalanmış Görevler"]
        direction TB
        G1["Oturum 1:\nProje yapısını oluştur"]
        G2["Oturum 2:\nAuth modülünü yaz"]
        G3["Oturum 3:\nProduct modülünü yaz"]
        G4["Oturum 4:\nTestleri yaz"]
        G5["Oturum 5:\nCI/CD ve docs"]
    end

    style OVERFLOW fill:#E74C3C,color:#fff
    style G1 fill:#27AE60,color:#fff
    style G2 fill:#27AE60,color:#fff
    style G3 fill:#27AE60,color:#fff
    style G4 fill:#27AE60,color:#fff
    style G5 fill:#27AE60,color:#fff
```

### Parçalama Stratejileri

| Strateji | Açıklama | Örnek |
|----------|----------|-------|
| **Modül bazlı** | Her modülü ayrı oturumda | Auth → Products → Orders |
| **Katman bazlı** | Her katmanı ayrı oturumda | Database → Service → API → UI |
| **İşlem bazlı** | Her işlem türünü ayrı oturumda | Oluştur → Test → Refactor |
| **Dosya bazlı** | Büyük dosyaları ayrı oturumda | Her 5-10 dosya için yeni oturum |

---

## Pratik Örnek 1: Context Window İzleme Akışı

```bash
# 1. Oturumu başlatın
$ claude

# 2. Token durumunu kontrol edin
> /context

# 3. Görevinizi verin
> Auth modülü için rate limiting middleware yaz

# 4. Görev uzadığında tekrar kontrol edin
> /context

# 5. %73+ ise compact yapın
> /compact auth göreviyle ilgili bağlamı koru

# 6. %80+ ise yeni oturum başlatın
> /exit
$ claude
> Auth modülünde kaldığım yerden devam et, rate limiting middleware'i tamamla
```

---

## Pratik Örnek 2: Verimli Token Kullanımı

Token tüketimini azaltmak için yapılabilecekler:

```bash
# ❌ Kötü: Tüm dosyaları okutma
> Projedeki tüm dosyaları oku ve analiz et

# ✅ İyi: Spesifik dosya iste
> src/auth/middleware.ts dosyasını oku

# ❌ Kötü: Belirsiz görev
> Bu projeyi iyileştir

# ✅ İyi: Odaklı görev
> src/auth/middleware.ts'deki rate limiting fonksiyonuna
> Redis cache desteği ekle

# ❌ Kötü: Çıktıyı gereksiz yere büyütme
> Tüm test sonuçlarını göster

# ✅ İyi: Özetli çıktı
> Testleri çalıştır, sadece başarısız olanları göster
```

---

## Pratik Örnek 3: Büyük Proje Stratejisi

50+ dosyalık bir projeyi refactor etmeniz gerekiyorsa:

```bash
# Oturum 1: Analiz ve plan
$ claude
> Bu projenin yapısını analiz et ve refactoring planı oluştur.
> Planı REFACTORING-PLAN.md dosyasına kaydet.

# Oturum 2: İlk modül
$ claude
> REFACTORING-PLAN.md'yi oku. Adım 1'i uygula: auth modülü refactoring.

# Oturum 3: İkinci modül
$ claude
> REFACTORING-PLAN.md'yi oku. Adım 2'yi uygula: product modülü refactoring.

# Her oturum temiz bağlamla başlar!
```

Bu yaklaşımın avantajı, her oturumun temiz bir context window ile başlamasıdır. Plan dosyası, oturumlar arasında bağlam aktarımını sağlar.

---

## Context Window Yönetim Kontrol Listesi

```mermaid
flowchart TD
    START["Oturum Başlangıcı"] --> Q1{"Görev ne kadar\nbüyük?"}

    Q1 -->|"Küçük\n(1-3 dosya)"| SINGLE["Tek oturumda tamamla"]
    Q1 -->|"Orta\n(5-10 dosya)"| MONITOR["Token sayacını izle\n%73'te /compact yap"]
    Q1 -->|"Büyük\n(10+ dosya)"| CHUNK["Görevleri parçala\nHer parça ayrı oturum"]

    SINGLE --> DONE["✅ Tamamlandı"]
    MONITOR --> CHECK{"%80+ mi?"}
    CHECK -->|"Hayır"| CONTINUE["Devam et"]
    CHECK -->|"Evet"| NEW["Yeni oturum başlat\n--continue ile"]
    CHUNK --> PLAN["Plan dosyası oluştur\nOturumlar arası bağlam"]

    style DONE fill:#27AE60,color:#fff
    style NEW fill:#E74C3C,color:#fff
```

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Context Window** | Claude'un aynı anda görebildiği 200K token'lık bilgi penceresi |
| **%80 Kuralı** | %80 dolulukta yeni oturum başlatın |
| **Performans eğrisi** | 0-25% optimal → 73%+ ciddi düşüş |
| **/compact** | Konuşma geçmişini özetleyerek yer açar |
| **/context** | Token kullanımını görselleştirir |
| **Task chunking** | Büyük görevleri parçalayarak context overflow'u önleme |

---

## Sonraki Adım

Context window dolduysa veya farklı bir göreve geçecekseniz oturumu yönetmeniz gerekir. Oturum yönetimi mekanizmalarını inceleyelim:

→ [Oturum Yönetimi](./06-oturum-yonetimi.md)
