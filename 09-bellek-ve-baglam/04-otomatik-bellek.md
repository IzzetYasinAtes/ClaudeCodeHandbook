# Otomatik Bellek (Auto Memory)

Claude Code, sizin düzeltmelerinizden ve tercihlerinizden öğrendiklerini otomatik olarak kaydeden bir **auto memory** (otomatik bellek) mekanizmasına sahiptir. Bu dosya, CLAUDE.md'nin tamamlayıcısıdır — siz CLAUDE.md'yi yazarsınız, Claude otomatik belleği yazar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| CLAUDE.md dosyası | [CLAUDE.md Dosyası](./01-claude-md-dosyasi.md) |
| Kurallar dizini | [Kurallar Dizini](./03-kurallar-dizini.md) |

---

## Otomatik Bellek Nedir?

Claude Code, çalışma sırasında yaptığınız düzeltmeleri ve verdiğiniz geri bildirimleri fark eder. Bu öğrenmeleri `~/.claude/user_memory.md` dosyasına kaydeder ve sonraki oturumlarda bu bilgileri kullanır.

```mermaid
flowchart TD
    subgraph oturum ["Çalışma Oturumu"]
        U1["Kullanıcı: 'Hayır, burada\nsingle quote kullan'"]
        U2["Kullanıcı: 'Test dosyalarını\n__tests__/ altına koy'"]
        U3["Kullanıcı: 'Commit mesajlarını\nİngilizce yaz'"]
    end

    subgraph ogrenme ["Claude'un Öğrenmesi"]
        L1["Düzeltmeyi algıla"]
        L2["Kalıbı çıkar"]
        L3["Belleğe kaydet"]
    end

    subgraph bellek ["~/.claude/user_memory.md"]
        M1["- Single quote tercih eder"]
        M2["- Test dosyaları __tests__/ altında"]
        M3["- Commit mesajları İngilizce"]
    end

    U1 --> L1
    U2 --> L1
    U3 --> L1
    L1 --> L2 --> L3
    L3 --> M1
    L3 --> M2
    L3 --> M3

    style bellek fill:#27AE60,color:#fff
```

---

## CLAUDE.md vs Otomatik Bellek

İki bellek mekanizması birbirini tamamlar:

```mermaid
flowchart LR
    subgraph claude_md ["CLAUDE.md (Siz Yazarsınız)"]
        direction TB
        C1["Proje kuralları"]
        C2["Mimari kararlar"]
        C3["Build komutları"]
        C4["Kodlama standartları"]
        C5["Bilinçli, planlı talimatlar"]
    end

    subgraph auto_mem ["Otomatik Bellek (Claude Yazar)"]
        direction TB
        A1["Kişisel tercihler"]
        A2["Düzeltme kalıpları"]
        A3["İş akışı alışkanlıkları"]
        A4["Stil tercihleri"]
        A5["Organik öğrenmeler"]
    end

    SESSION["Oturum\nBaşlangıcı"] --> claude_md
    SESSION --> auto_mem
    claude_md --> CONTEXT["Birleşik\nBağlam"]
    auto_mem --> CONTEXT

    style claude_md fill:#4A90D9,color:#fff
    style auto_mem fill:#E67E22,color:#fff
    style CONTEXT fill:#27AE60,color:#fff
```

### Detaylı Karşılaştırma

| Özellik | CLAUDE.md | Otomatik Bellek |
|---------|-----------|-----------------|
| **Kim yazar** | Siz (geliştirici) | Claude Code |
| **Konum** | Proje kökü / user dizini | `~/.claude/user_memory.md` |
| **Kapsam** | Proje bazlı veya kullanıcı bazlı | Kullanıcı bazlı (tüm projeler) |
| **İçerik türü** | Bilinçli kurallar ve talimatlar | Öğrenilmiş tercihler ve kalıplar |
| **Boyut limiti** | Önerilen: 300 satır altı | İlk 200 satır ile sınırlı |
| **Versiyon kontrolü** | Evet (git ile) | Hayır (yerel dosya) |
| **Yükleme zamanı** | Oturum başlangıcı | Oturum başlangıcı |
| **Düzenlenebilir** | Evet | Evet (elle düzenleyebilirsiniz) |

---

## Otomatik Bellek Nasıl Çalışır?

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant CC as Claude Code
    participant MEM as user_memory.md

    Note over U,CC: Oturum Sırasında
    U->>CC: "Bu fonksiyona JSDoc ekle"
    CC->>CC: JSDoc ekledi (çift tırnak ile)
    U->>CC: "Hayır, tek tırnak kullan JSDoc'larda"
    CC->>CC: Düzeltmeyi algıladı

    Note over CC,MEM: Öğrenme Kaydı
    CC->>MEM: "Kullanıcı JSDoc'larda tek tırnak tercih ediyor"

    Note over U,CC: Sonraki Oturum
    CC->>MEM: Belleği oku
    MEM-->>CC: Tercihler listesi
    U->>CC: "Bu dosyaya JSDoc ekle"
    CC->>CC: Tek tırnak ile JSDoc ekledi ✅

    Note over U: Düzeltmeye gerek kalmadı!
```

### Öğrenme Tetikleyicileri

Claude Code şu durumlarda öğrenme kaydeder:

| Tetikleyici | Örnek | Kaydedilen Bilgi |
|-------------|-------|-----------------|
| **Düzeltme** | "Hayır, snake_case kullan" | İsimlendirme tercihi |
| **Tercih belirtme** | "Ben her zaman async/await kullanırım" | Kod stili tercihi |
| **Ret** | "Bu yaklaşımı istemiyorum, X yap" | Anti-pattern bilgisi |
| **Onay** | "Evet, tam böyle yap her seferinde" | Pozitif kalıp |

---

## Otomatik Bellek Dosyası Örneği

`~/.claude/user_memory.md` dosyasının tipik içeriği:

```markdown
- Kullanıcı Türkçe yanıt tercih ediyor
- Commit mesajları İngilizce ve Conventional Commits formatında
- TypeScript'te single quote kullanılıyor
- Import sıralaması: React > üçüncü parti > yerel > tipler
- Test dosyaları __tests__/ dizinine konuluyor
- Fonksiyon bileşenleri tercih ediliyor, class bileşen kullanılmıyor
- Hata mesajlarında kullanıcıya yönelik açıklama eklenmeli
- Console.log yerine proper logger kullanılmalı
- 2 boşluk indent tercih ediliyor
- Trailing comma kullanılıyor
- API yanıtlarında her zaman try-catch bloğu kullanılıyor
- Git branch isimlendirmesi: feature/kisa-aciklama formatında
```

---

## 200 Satır Limiti

Otomatik bellek dosyası **ilk 200 satır** ile sınırlıdır. Claude Code yalnızca bu satırları okur:

```mermaid
flowchart TD
    MEM["user_memory.md"] --> READ{"Dosya kaç\nsatır?"}
    READ -->|"≤ 200 satır"| ALL["Tümü okunur ✅"]
    READ -->|"> 200 satır"| PARTIAL["İlk 200 satır okunur ⚠️\nGeri kalanı göz ardı edilir"]

    PARTIAL --> ACTION["Dosyayı düzenleyin:\n- Eski/gereksiz girdileri silin\n- Benzer girdileri birleştirin\n- Önceliklendirin"]

    style ALL fill:#27AE60,color:#fff
    style PARTIAL fill:#F39C12,color:#fff
```

### Limit Yönetimi İpuçları

1. **Periyodik temizlik:** Ayda bir `~/.claude/user_memory.md`'yi gözden geçirin
2. **Eski girdileri silin:** Artık geçerli olmayan tercihleri kaldırın
3. **Birleştirin:** Benzer girdileri tek satırda birleştirin
4. **Projeye özel olanları taşıyın:** Proje bazlı kuralları CLAUDE.md'ye taşıyın

---

## Pratik Örnek 1: Otomatik Belleği İnceleme

```bash
# Otomatik bellek dosyasını görüntüleyin
$ cat ~/.claude/user_memory.md

# Satır sayısını kontrol edin
$ wc -l ~/.claude/user_memory.md

# Belirli bir tercihi arayın
$ grep -i "typescript" ~/.claude/user_memory.md
```

---

## Pratik Örnek 2: Otomatik Belleği Düzenleme

Otomatik bellek dosyasını elle düzenleyebilirsiniz:

```bash
# Editörle açın
$ code ~/.claude/user_memory.md    # VS Code
$ nano ~/.claude/user_memory.md    # Terminal editör
```

**Düzenleme önerileri:**

```markdown
# ÖNCE (dağınık, tekrarlı)
- Kullanıcı tek tırnak seviyor
- TypeScript'te single quote kullan
- Tırnak işareti olarak ' kullanılmalı
- Her zaman fonksiyon bileşeni yaz
- Class component kullanma
- Functional component tercih ediliyor

# SONRA (temiz, birleştirilmiş)
- Tırnak işareti: single quote (tek tırnak)
- Bileşen tipi: fonksiyon bileşenleri (class bileşen kullanma)
```

---

## Pratik Örnek 3: Bellek Katmanlarının Etkileşimi

Bir oturum başlangıcında tüm bellek katmanları birlikte yüklenir:

```mermaid
flowchart TD
    subgraph katmanlar ["Bellek Katmanları"]
        direction TB
        UM["~/.claude/user_memory.md\n(Otomatik Bellek)"]
        UC["~/.claude/CLAUDE.md\n(Kullanıcı CLAUDE.md)"]
        PC["./CLAUDE.md\n(Proje CLAUDE.md)"]
        PR[".claude/rules/*.md\n(Kural Dosyaları)"]
    end

    SESSION["Oturum Başlangıcı"] --> UM
    SESSION --> UC
    SESSION --> PC
    WORK["Dosya ile çalışma"] --> PR

    UM --> CTX["Birleşik Bağlam"]
    UC --> CTX
    PC --> CTX
    PR -.->|"İstek üzerine"| CTX

    CTX --> CLAUDE["Claude Code\nyönlendirilmiş çalışır"]

    style UM fill:#E67E22,color:#fff
    style UC fill:#3498DB,color:#fff
    style PC fill:#27AE60,color:#fff
    style PR fill:#F39C12,color:#fff
```

---

## Ne Zaman Hangisini Kullanmalı?

| Bilgi Türü | Otomatik Belleğe Bırak | CLAUDE.md'ye Yaz |
|-------------|:----------------------:|:----------------:|
| Kişisel stil tercihleri | ✅ | ❌ |
| Proje build komutları | ❌ | ✅ |
| Dil tercihi (Türkçe/İngilizce) | ✅ | ✅ |
| Mimari kararlar | ❌ | ✅ |
| İsimlendirme kuralları | Claude öğrenebilir | ✅ (güvenilir olması için) |
| Import sıralaması | Claude öğrenebilir | ✅ (güvenilir olması için) |
| Takım genelinde kurallar | ❌ | ✅ |

> **İpucu:** Kritik ve takım genelinde uyulması gereken kuralları her zaman CLAUDE.md'ye yazın. Otomatik belleğe güvenmek, bu kuralların kaybolma veya göz ardı edilme riski taşır.

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Auto Memory** | Claude'un düzeltmelerden ve tercihlerden öğrendiği bilgileri kaydetmesi |
| **Dosya konumu** | `~/.claude/user_memory.md` |
| **200 satır limiti** | Yalnızca ilk 200 satır okunur |
| **Tamamlayıcı** | CLAUDE.md'nin yerine değil, yanına çalışır |
| **Düzenlenebilir** | Elle düzenleme, birleştirme ve temizlik yapılabilir |
| **Kapsam** | Kullanıcı bazlı, tüm projelerde geçerli |

---

## Sonraki Adım

Bellek mekanizmalarını öğrendik. Şimdi en kritik beceri olan context window yönetimini inceleyelim:

→ [Context Window Yönetimi](./05-context-window-yonetimi.md)
