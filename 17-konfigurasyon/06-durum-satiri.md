# Durum Çubuğu (Status Bar)

Claude Code, terminal penceresinin alt kısmında bir status bar (durum çubuğu) gösterir. Bu çubuk, context window (bağlam penceresi) kullanımı, tahmini maliyet ve model bilgisi gibi kritik verileri gerçek zamanlı sunar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Terminal optimizasyonu | [Terminal Optimizasyonu](./08-terminal-optimizasyonu.md) |
| Context window yönetimi | [Context Window Yönetimi](../09-bellek-ve-baglam/05-context-window-yonetimi.md) |
| Maliyet yönetimi | [Maliyet Yönetimi](./05-maliyet-yonetimi.md) |

---

## Durum Çubuğu Bileşenleri

Claude Code durum çubuğu şu bilgileri gösterir:

```mermaid
flowchart LR
    subgraph statusbar ["Durum Çubuğu"]
        direction LR
        MODEL["🤖 Model\nsonnet-4"]
        CONTEXT["📊 Context\n45K / 200K"]
        COST["💰 Maliyet\n$0.47"]
        MODE["📋 Mod\nagent"]
    end

    style MODEL fill:#3498DB,color:#fff
    style CONTEXT fill:#F39C12,color:#fff
    style COST fill:#E74C3C,color:#fff
    style MODE fill:#27AE60,color:#fff
```

### Bileşen Detayları

| Bileşen | Görüntülenen Bilgi | Anlamı |
|---------|-------------------|--------|
| Model | Aktif model adı | Kullanılan Claude modeli |
| Context | `kullanılan / toplam` token | Context window doluluk oranı |
| Maliyet | Tahmini oturum maliyeti ($) | Şu ana kadar harcanan tutar |
| Mod | `agent`, `plan`, `ask` | Aktif çalışma modu |

---

## Context Window İzleme

Durum çubuğundaki en kritik metrik context window kullanımıdır:

```mermaid
flowchart TD
    subgraph usage ["Context Window Durumları"]
        GREEN["🟢 %0-%60\nNormal kullanım\nDevam edin"]
        YELLOW["🟡 %60-%80\n/compact düşünün\nBağlam büyüyor"]
        RED["🔴 %80-%100\n/compact gerekli!\nOtomatik compact yakın"]
    end

    GREEN --> YELLOW --> RED
    RED --> COMPACT["/compact\nBağlamı özetle"]
    COMPACT --> GREEN

    style GREEN fill:#27AE60,color:#fff
    style YELLOW fill:#F39C12,color:#fff
    style RED fill:#E74C3C,color:#fff
    style COMPACT fill:#3498DB,color:#fff
```

### Context Doluluk Renkleri

| Doluluk | Renk | Anlam | Aksiyon |
|---------|------|-------|---------|
| %0 - %60 | 🟢 Yeşil | Normal | Devam edin |
| %60 - %80 | 🟡 Sarı | Dikkat | `/compact` düşünün |
| %80 - %100 | 🔴 Kırmızı | Kritik | Hemen `/compact` veya yeni oturum |

---

## Maliyet İzleme

Durum çubuğundaki maliyet göstergesi, oturum boyunca kümülatif harcamayı takip eder:

```
Session: $0.47  (Input: 45K tokens, Output: 12K tokens)
```

Daha detaylı maliyet bilgisi için `/cost` komutunu kullanın:

```
> /cost

Session Cost Summary
─────────────────────
Model:           claude-sonnet-4-20250514
Input tokens:    45,230  ($0.14)
Output tokens:   12,840  ($0.19)
Cache read:      38,100  ($0.01)
Cache write:      7,130  ($0.02)
Thinking:         8,400  ($0.13)
─────────────────────
Estimated cost:  $0.49
Duration:        12m 34s
```

---

## Git Durumu İzleme

Claude Code durum çubuğu, proje Git durumunu da yansıtabilir:

```mermaid
flowchart LR
    GIT["Git Bilgisi"] --> BRANCH["🌿 Branch\nmain"]
    GIT --> CHANGES["📝 Değişiklikler\n3 modified, 1 added"]
    GIT --> STATUS["🔄 Durum\nClean / Dirty"]

    style GIT fill:#2C3E50,color:#fff
    style BRANCH fill:#27AE60,color:#fff
    style CHANGES fill:#F39C12,color:#fff
    style STATUS fill:#3498DB,color:#fff
```

---

## Durum Çubuğu Konfigürasyonu

Durum çubuğunun görünümü ve davranışı özelleştirilebilir:

### Temel Ayarlar

```json
{
  "statusBar": {
    "showCost": true,
    "showContext": true,
    "showModel": true,
    "showMode": true
  }
}
```

### Özel Durum Gösterimi

Hook'lar aracılığıyla durum çubuğuna özel bilgiler eklenebilir:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Git branch: $(git branch --show-current), Node: $(node --version)\""
          }
        ]
      }
    ]
  }
}
```

---

## Pratik Örnek: Durum Çubuğu Odaklı İş Akışı

### Adım Adım İzleme

```mermaid
sequenceDiagram
    participant DEV as Geliştirici
    participant SB as Durum Çubuğu
    participant CC as Claude Code

    DEV->>CC: Oturum başlat
    SB->>DEV: 🤖 sonnet-4 | 📊 2K/200K | 💰 $0.00 | agent

    DEV->>CC: "auth modülünü refactor et"
    CC->>CC: Dosyaları oku, analiz et
    SB->>DEV: 🤖 sonnet-4 | 📊 45K/200K | 💰 $0.32 | agent

    DEV->>CC: "testleri de güncelle"
    CC->>CC: Test dosyalarını oku ve düzenle
    SB->>DEV: 🤖 sonnet-4 | 📊 95K/200K | 💰 $0.78 | agent

    Note over SB: ⚠️ Context %47'ye ulaştı

    DEV->>CC: Devam et...
    SB->>DEV: 🤖 sonnet-4 | 📊 140K/200K | 💰 $1.24 | agent

    Note over SB: 🔴 Context %70, compact önerisi

    DEV->>CC: /compact
    SB->>DEV: 🤖 sonnet-4 | 📊 18K/200K | 💰 $1.28 | agent
```

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| Durum çubuğunu görmezden gelmek | Context ve maliyet bilgilerine düzenli bakın |
| Context kırmızıya dönene kadar beklemek | Sarı bölgede `/compact` kullanın |
| Terminal penceresini çok dar yapmak | Tüm göstergelerin görünmesi için yeterli genişlik sağlayın |

---

## Özet

| Bileşen | Bilgi | İzleme Amacı |
|---------|-------|-------------|
| Model | Aktif model | Doğru model kullanıldığından emin olma |
| Context | Token kullanımı | Compact zamanını belirleme |
| Maliyet | Oturum harcaması | Bütçe kontrolü |
| Mod | Çalışma modu | Aktif modun farkında olma |

---

## Sonraki Adım

Claude Code'daki klavye kısayollarını ve özelleştirme seçeneklerini öğrenelim:

→ [Klavye Kısayolları](./07-klavye-kisayollari.md)
