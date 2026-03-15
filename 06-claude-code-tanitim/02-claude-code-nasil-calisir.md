# Claude Code Nasıl Çalışır?

Claude Code, bir **agentic loop** (ajantik döngü) üzerinde çalışır. Kullanıcının verdiği görevi alır, planlar, araçları kullanarak adım adım yürütür ve sonucu doğrular. Bu bölümde bu döngünün detaylarını, araç sistemini ve izin modelini inceleyeceğiz.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code nedir | [Claude Code Nedir?](./01-claude-code-nedir.md) |
| LLM'lerin çalışma mantığı | [Bölüm 02](../02-buyuk-dil-modelleri/README.md) |

---

## Agentic Loop (Ajantik Döngü)

Claude Code'un çalışma prensibi basit ama güçlüdür: **oku → düşün → yap → gözlemle → tekrarla**.

```mermaid
flowchart TD
    START(["Kullanıcı\nistekte bulunur"]) --> GATHER["1. Bağlam Toplama\n(Context Gathering)"]
    GATHER --> PLAN["2. Planlama\n(Planning)"]
    PLAN --> EXEC["3. Yürütme\n(Execution)"]
    EXEC --> OBSERVE["4. Gözlemleme\n(Observation)"]
    OBSERVE --> CHECK{"Görev\ntamamlandı mı?"}
    CHECK -->|"Hayır"| PLAN
    CHECK -->|"Evet"| RESPOND["5. Yanıt\n(Response)"]
    RESPOND --> END(["Kullanıcıya\nsonuç gösterilir"])

    style START fill:#4A90D9,color:#fff
    style END fill:#27AE60,color:#fff
    style CHECK fill:#F39C12,color:#fff
```

### Adım Adım Açıklama

| Adım | Açıklama | Örnek |
|------|----------|-------|
| **1. Bağlam Toplama** | Proje yapısını, mevcut dosyaları ve ilgili kodu okur | `CLAUDE.md`, dizin yapısı, git durumu |
| **2. Planlama** | Görevi alt adımlara böler, strateji belirler | "Önce mevcut kodu okuyayım, sonra testi yazayım" |
| **3. Yürütme** | Araçları kullanarak planı uygular | Dosya okuma, kod yazma, komut çalıştırma |
| **4. Gözlemleme** | Sonuçları değerlendirir, hataları yakalar | Test sonuçları, hata mesajları, çıktılar |
| **5. Yanıt** | Tamamlanan işin özetini kullanıcıya sunar | "3 dosya oluşturdum, 12 test yazdım, hepsi geçti" |

---

## Detaylı Sequence Diagram (Sıralama Diyagramı)

Aşağıdaki diyagram, bir kullanıcı isteğinin tüm yaşam döngüsünü gösterir:

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant CC as Claude Code CLI
    participant LLM as Claude Opus 4.6
    participant T as Araç Sistemi
    participant FS as Dosya Sistemi
    participant SH as Shell
    participant P as İzin Yöneticisi

    U->>CC: "auth modülüne rate limiting ekle"
    CC->>LLM: İstek + proje bağlamı gönderilir

    rect rgb(230, 240, 255)
        Note over LLM: Planlama Aşaması
        LLM->>LLM: Görevi analiz et
        LLM->>LLM: Alt adımları belirle
    end

    rect rgb(230, 255, 230)
        Note over CC,FS: Döngü 1: Bağlam Toplama
        LLM->>T: read_file("src/auth/")
        T->>FS: Dizin listesi oku
        FS-->>T: Dosya listesi
        T-->>LLM: auth/ dizin yapısı

        LLM->>T: read_file("src/auth/middleware.ts")
        T->>FS: Dosya içeriğini oku
        FS-->>T: Dosya içeriği
        T-->>LLM: Mevcut middleware kodu
    end

    rect rgb(255, 243, 230)
        Note over CC,SH: Döngü 2: Uygulama
        LLM->>T: write_file("src/auth/rate-limiter.ts")
        T->>P: Yazma izni kontrol
        P->>U: "rate-limiter.ts oluşturulsun mu?"
        U-->>P: Evet
        P-->>T: İzin verildi
        T->>FS: Dosya oluştur
        FS-->>T: Başarılı

        LLM->>T: edit_file("src/auth/middleware.ts")
        T->>P: Düzenleme izni kontrol
        P-->>T: Önceden izin verilmişti
        T->>FS: Dosya düzenle
        FS-->>T: Başarılı
    end

    rect rgb(255, 230, 230)
        Note over CC,SH: Döngü 3: Doğrulama
        LLM->>T: bash("npm test")
        T->>P: Komut çalıştırma izni
        P-->>T: İzin verildi
        T->>SH: npm test
        SH-->>T: 2 test başarısız

        Note over LLM: Hata algılandı, düzeltme planı
        LLM->>T: edit_file("src/auth/rate-limiter.ts")
        T->>FS: Hatayı düzelt
        FS-->>T: Başarılı

        LLM->>T: bash("npm test")
        T->>SH: npm test
        SH-->>T: Tüm testler geçti ✓
    end

    LLM-->>CC: Özet yanıt
    CC-->>U: "Rate limiting eklendi:\n- rate-limiter.ts oluşturuldu\n- middleware.ts güncellendi\n- Tüm testler geçiyor"
```

---

## Araç Sistemi (Tool System)

Claude Code, 30'dan fazla dahili **tool** (araç) ile donatılmıştır. Her araç belirli bir görevi yerine getirir:

```mermaid
flowchart TD
    subgraph dosya ["Dosya Araçları"]
        F1["Read\n(Dosya oku)"]
        F2["Write\n(Dosya yaz)"]
        F3["Edit\n(Dosya düzenle)"]
        F4["MultiEdit\n(Çoklu düzenleme)"]
        F5["Glob\n(Dosya ara)"]
        F6["Grep\n(İçerik ara)"]
    end

    subgraph shell ["Shell Araçları"]
        S1["Bash\n(Komut çalıştır)"]
        S2["Background\n(Arka plan)"]
    end

    subgraph web ["Web Araçları"]
        W1["WebFetch\n(Web sayfası oku)"]
        W2["WebSearch\n(Web'de ara)"]
    end

    subgraph task_tools ["Görev Araçları"]
        T1["TodoRead\n(Görev listesi oku)"]
        T2["TodoWrite\n(Görev listesi yaz)"]
        T3["Task\n(Alt görev oluştur)"]
    end

    subgraph other ["Diğer Araçlar"]
        O1["LSP\n(Kod analizi)"]
        O2["Notebook\n(Jupyter)"]
        O3["MCP\n(Harici araçlar)"]
    end

    CC["Claude Code"] --> dosya
    CC --> shell
    CC --> web
    CC --> task_tools
    CC --> other
```

### Araç Kategorileri

| Kategori | Araçlar | Açıklama |
|----------|---------|----------|
| **Dosya İşlemleri** | Read, Write, Edit, MultiEdit, Glob, Grep | Dosya okuma, yazma, arama |
| **Shell** | Bash | Terminal komutları çalıştırma |
| **Web** | WebFetch, WebSearch | İnternet erişimi |
| **Görev Yönetimi** | TodoRead, TodoWrite, Task | İç görev takibi ve subagent'lar |
| **Kod Analizi** | LSP | Sembol analizi, tanım bulma |
| **Notebook** | NotebookRead, NotebookEdit | Jupyter Notebook desteği |
| **MCP** | Dinamik araçlar | Harici entegrasyonlar |

---

## İzin Sistemi (Permission System)

Claude Code, güvenlik için katmanlı bir **permission** (izin) sistemi kullanır:

```mermaid
flowchart TD
    REQ["Araç çağrısı\n(Tool Call)"] --> TYPE{"Araç tipi?"}

    TYPE -->|"Salt okunur"| READONLY["Otomatik izin\n✅ İzin gerekmez"]
    TYPE -->|"Yazma/çalıştırma"| CHECK{"İzin kuralı\nvar mı?"}

    CHECK -->|"Evet, izin ver"| ALLOW["✅ Otomatik çalıştır"]
    CHECK -->|"Evet, reddet"| DENY["❌ Engelle"]
    CHECK -->|"Kural yok"| ASK["Kullanıcıya sor"]

    ASK --> USER{"Kullanıcı kararı"}
    USER -->|"Evet"| RUN["Çalıştır"]
    USER -->|"Evet, hep izin ver"| SAVE["İzni kaydet + çalıştır"]
    USER -->|"Hayır"| REJECT["Reddet"]

    READONLY ~~~ NOTE1["Örnekler:\nRead, Glob, Grep,\nLS, TodoRead"]
    ALLOW ~~~ NOTE2["Örnekler:\nÖnceden onaylanmış\nkomutlar"]

    style READONLY fill:#27AE60,color:#fff
    style ALLOW fill:#27AE60,color:#fff
    style DENY fill:#E74C3C,color:#fff
    style ASK fill:#F39C12,color:#fff
```

### İzin Kategorileri

| Kategori | Örnekler | İzin Gerekir mi? |
|----------|----------|-------------------|
| **Salt okunur** | Dosya okuma, arama, dizin listeleme | Hayır |
| **Yazma** | Dosya oluşturma, düzenleme, silme | Evet |
| **Komut çalıştırma** | Shell komutları, build, test | Evet |
| **Web erişimi** | Web sayfası okuma | Genellikle hayır |

### İzin Yanıt Seçenekleri

Bir izin istemi geldiğinde kullanıcıya şu seçenekler sunulur:

```
Claude Code wants to run: npm test

  Allow (Yalnız bu sefer izin ver)
  Allow always (Bu oturum için hep izin ver)
  Deny (Reddet)
```

---

## Bağlam Yönetimi (Context Management)

Claude Code, 200K token'lık **context window** (bağlam penceresi) içinde çalışır. Bağlamı verimli kullanmak için:

```mermaid
flowchart LR
    subgraph baglam ["200K Token Context Window"]
        direction TB
        SYS["Sistem Talimatları\n(~5K token)"]
        CLAUDE_MD["CLAUDE.md Kuralları\n(Değişken)"]
        CONV["Konuşma Geçmişi\n(Büyüyen)"]
        TOOL_OUT["Araç Çıktıları\n(Büyüyen)"]
        AVAIL["Kullanılabilir Alan\n(Azalan)"]
    end

    SYS --> CLAUDE_MD --> CONV --> TOOL_OUT --> AVAIL

    FULL{"Bağlam\ndoldu mu?"} -->|"Evet"| COMPACT["Compact\n(Sıkıştırma)"]
    COMPACT --> SUMMARY["Önceki konuşma\nözetlenir"]
    SUMMARY --> CONTINUE["Yeni bağlamla\ndevam"]
```

### Context Window Dolduğunda Ne Olur?

1. **Auto-compact:** Claude Code konuşma geçmişini otomatik olarak özetler
2. **Önemli bilgiler korunur:** Dosya içerikleri, araç çıktıları önceliklendirilir
3. **Kullanıcı fark etmez:** Süreç otomatik olarak gerçekleşir

---

## Çalışma Modları

Claude Code farklı **mode** (mod) seçenekleri sunar:

```mermaid
flowchart LR
    subgraph modlar ["Claude Code Modları"]
        IM["Interactive Mode\n(Varsayılan)\nOku + Yaz + Çalıştır"]
        PM["Plan Mode\nSadece Oku + Planla\nYazma yok"]
        FM["Fast Mode\nHızlı model\nBasit görevler için"]
    end

    IM -.->|"Ctrl+Shift+Tab"| PM
    PM -.->|"Ctrl+Shift+Tab"| IM
    IM -.->|"Shift+Tab"| FM
```

| Mod | Açıklama | Kullanım Senaryosu |
|-----|----------|--------------------|
| **Interactive** | Tam yetkili mod; okur, yazar, çalıştırır | Genel geliştirme |
| **Plan** | Salt okunur; plan yapar ama değişiklik yapmaz | Mimari inceleme, strateji |
| **Fast** | Hızlı model kullanır, basit görevler için | Kısa sorular, küçük düzenlemeler |

---

## Pratik Örnek: Bir Görevin Yaşam Döngüsü

Diyelim ki şu komutu verdiniz:

```bash
$ claude
> Bu projedeki tüm TODO yorumlarını bul ve bir rapor oluştur
```

Claude Code şu adımları izler:

| Adım | Araç | Eylem |
|------|------|-------|
| 1 | `Grep` | Tüm dosyalarda `TODO` arar |
| 2 | `Read` | Bulunan dosyaların ilgili bölümlerini okur |
| 3 | *(düşünme)* | Sonuçları kategorize eder |
| 4 | `Write` | `TODO-REPORT.md` dosyası oluşturur |
| 5 | *(yanıt)* | Kullanıcıya özet sunar |

Tüm bu süreçte kullanıcı yalnızca ilk komutu verir ve sonucu görür. Ara adımlarda izin isterse onay verir.

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Agentic Loop** | Oku → Düşün → Yap → Gözlemle → Tekrarla döngüsü |
| **Tool System** | 30+ dahili araç (dosya, shell, web, görev) |
| **Permission System** | Yazma/çalıştırma için kullanıcı onayı gerektiren güvenlik katmanı |
| **Context Window** | 200K token kapasiteli bağlam penceresi |
| **Auto-compact** | Bağlam dolduğunda otomatik özetleme |

---

## Sonraki Adım

Claude Code'un nasıl çalıştığını anladık. Şimdi onu bilgisayarınıza kuralım:

→ [Kurulum ve Gereksinimler](./03-kurulum-ve-gereksinimler.md)
