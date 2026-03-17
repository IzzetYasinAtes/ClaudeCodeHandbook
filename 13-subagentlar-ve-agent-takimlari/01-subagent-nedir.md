# Subagent Nedir?

Subagent (alt ajan), bir ana Claude Code oturumunun içinden başlatılan **bağımsız, izole bir agent örneğidir**. Ana agent (parent agent), karmaşık bir görevi daha küçük parçalara böler ve her parçayı ayrı bir subagent'a devreder. Subagent kendi bağlamında çalışır, işini bitirir ve yalnızca **nihai sonucu** ana agent'a döndürür.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code nasıl çalışır | [Claude Code Nasıl Çalışır?](../06-claude-code-tanitim/02-claude-code-nasil-calisir.md) |
| Araçlara genel bakış | [Araçlara Genel Bakış](../08-araclar/01-araclara-genel-bakis.md) |
| Context window yönetimi | [Context Window Yönetimi](../09-bellek-ve-baglam/05-context-window-yonetimi.md) |

---

## Subagent Mimarisi

Bir subagent, ana agent'tan tamamen izole bir şekilde çalışır. Kendi conversation context'i (konuşma bağlamı) vardır ve ana agent'ın context window'unu tüketmez.

```mermaid
flowchart TD
    USER["👤 Kullanıcı"] --> PARENT["🤖 Ana Agent<br/>(Parent Agent)"]

    PARENT --> |"Görev 1"| SUB1["🔧 Subagent 1<br/>Kod İnceleme"]
    PARENT --> |"Görev 2"| SUB2["🔧 Subagent 2<br/>Test Yazma"]
    PARENT --> |"Görev 3"| SUB3["🔧 Subagent 3<br/>Dokümantasyon"]

    SUB1 --> |"Sonuç 1"| PARENT
    SUB2 --> |"Sonuç 2"| PARENT
    SUB3 --> |"Sonuç 3"| PARENT

    PARENT --> |"Birleşik sonuç"| USER

```

---

## Subagent'ların Temel Avantajları

### 1. Context Isolation (Bağlam İzolasyonu)

Her subagent **temiz bir konuşma bağlamıyla** başlar. Ana agent'ın bağlamını kirletmez ve sadece nihai sonuç geri döner.

```mermaid
flowchart LR
    subgraph parent ["Ana Agent Context Window"]
        P1["Kullanıcı isteği"]
        P2["Proje analizi"]
        P3["Subagent sonucu<br/>(özet)"]
    end

    subgraph sub ["Subagent Context Window (İzole)"]
        S1["Görev talimatı"]
        S2["50 dosya okuma"]
        S3["Analiz ve karşılaştırma"]
        S4["Detaylı raporlama"]
    end

    P2 --> |"Sadece görev tanımı"| S1
    S4 --> |"Sadece özet sonuç"| P3

```

**Neden önemli?** Bir subagent 50 dosya okuyup binlerce satır kod analiz edebilir, ancak ana agent'ın context window'una yalnızca birkaç paragraflık sonuç eklenir.

### 2. Parallelization (Paralelleştirme)

Birden fazla subagent **eş zamanlı olarak** çalışabilir. Bu, büyük görevlerde dramatik hız artışı sağlar.

```mermaid
flowchart TD
    PARENT["Ana Agent"] --> |"t=0"| FORK["Görevleri dağıt"]

    FORK --> SUB1["Subagent 1<br/>Frontend analizi<br/>⏱️ 30 sn"]
    FORK --> SUB2["Subagent 2<br/>Backend analizi<br/>⏱️ 45 sn"]
    FORK --> SUB3["Subagent 3<br/>DB analizi<br/>⏱️ 20 sn"]

    SUB1 --> JOIN["Sonuçları birleştir"]
    SUB2 --> JOIN
    SUB3 --> JOIN

    JOIN --> RESULT["Toplam: ~45 sn<br/>(Seri olsaydı: ~95 sn)"]

```

### 3. Specialized Instructions (Özelleştirilmiş Talimatlar)

Her subagent'a özel system prompt'lar (sistem istemleri) ve çalışma kuralları verilebilir.

```
Ana Agent: "Bu projenin güvenlik denetimini yap"

├── Subagent 1 (Güvenlik Uzmanı rolü):
│   "SQL injection, XSS ve CSRF açıklarını tara"
│
├── Subagent 2 (Performans Uzmanı rolü):
│   "N+1 sorguları ve bellek sızıntılarını bul"
│
└── Subagent 3 (Kod Kalitesi rolü):
    "SOLID ihlallerini ve code smell'leri raporla"
```

### 4. Tool Restrictions (Araç Kısıtlamaları)

Subagent'ların erişebileceği araçlar sınırlandırılabilir. Örneğin salt okunur bir subagent'a yalnızca `Read`, `Glob`, `Grep` araçları verilebilir.

```mermaid
flowchart LR
    subgraph full ["Ana Agent — Tam Araç Seti"]
        F1["Read"] ~~~ F2["Write"] ~~~ F3["Edit"]
        F4["Bash"] ~~~ F5["WebSearch"]
    end

    subgraph restricted ["Subagent — Kısıtlı Araçlar"]
        R1["Read"]
        R2["Glob"]
        R3["Grep"]
    end

    full --> |"allowedTools:<br/>Read, Glob, Grep"| restricted

```

---

## Subagent Tanımlama Yöntemleri

Claude Code'da subagent'lar üç farklı yöntemle tanımlanabilir:

```mermaid
flowchart TD
    ROOT["Subagent Tanımlama<br/>Yöntemleri"] --> BUILTIN["1. Dahili Subagent'lar<br/>(Built-in)"]
    ROOT --> FILESYSTEM["2. Dosya Sistemi<br/>(Filesystem)"]
    ROOT --> PROGRAMMATIC["3. Programatik<br/>(SDK)"]

    BUILTIN --> B1["Explore"]
    BUILTIN --> B2["Plan"]
    BUILTIN --> B3["GeneralPurpose"]

    FILESYSTEM --> FS1[".claude/agents/<br/>dizinindeki<br/>.md dosyaları"]

    PROGRAMMATIC --> PR1["Claude SDK ile<br/>AgentDefinition<br/>nesnesi"]

```

### 1. Dahili Subagent'lar (Built-in Subagents)

Claude Code ile birlikte gelen, önceden tanımlanmış subagent'lardır. Herhangi bir yapılandırma gerektirmeden kullanılabilir.

| Subagent | Açıklama | Model |
|----------|----------|-------|
| **Explore** | Salt okunur dosya keşfi, hızlı arama | Hafif model |
| **Plan** | Araştırma ve planlama | Varsayılan model |
| **GeneralPurpose** | Çok adımlı karmaşık görevler, tam araç erişimi | Varsayılan model |

> 📖 Detaylı bilgi: [Dahili Subagent'lar](./02-dahili-subagentlar.md)

### 2. Dosya Sistemi ile Tanımlama (Filesystem)

`.claude/agents/` dizininde markdown dosyaları oluşturarak özel subagent'lar tanımlanır.

```
proje/
├── .claude/
│   └── agents/
│       ├── code-reviewer.md
│       ├── test-writer.md
│       └── docs-generator.md
```

> 📖 Detaylı bilgi: [Özel Subagent Oluşturma](./03-ozel-subagent-olusturma.md)

### 3. Programatik Tanımlama (SDK)

Claude SDK üzerinden `AgentDefinition` nesnesi ile runtime'da subagent tanımlanır.

```python
from claude_sdk import AgentDefinition

reviewer = AgentDefinition(
    name="code-reviewer",
    description="Kod inceleme uzmanı",
    instructions="SOLID prensiplerini ve clean code kurallarını kontrol et.",
    allowedTools=["Read", "Glob", "Grep"],
    model="claude-sonnet-4-20250514"
)
```

> 📖 Detaylı bilgi: [Agent Tool Kullanımı](./05-agent-tool-kullanimi.md)

---

## Parent-Subagent Yaşam Döngüsü

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant P as Ana Agent
    participant S as Subagent

    U->>P: "Projedeki tüm TODO'ları bul ve raporla"
    P->>P: Görevi analiz et

    P->>S: Agent aracını çağır<br/>(görev tanımı + talimatlar)
    activate S

    Note over S: Yeni context window<br/>başlar (izole)

    S->>S: Glob("**/*.ts")
    S->>S: Grep("TODO|FIXME|HACK")
    S->>S: Read(bulunan dosyalar)
    S->>S: Sonuçları derle

    S-->>P: Sonuç: "47 TODO bulundu,<br/>12 dosyada dağılmış..."
    deactivate S

    Note over P: Sadece özet sonuç<br/>context'e eklenir

    P-->>U: Raporu sun
```

---

## Subagent Ne Zaman Kullanılmalı?

| Senaryo | Subagent Gerekli mi? | Neden? |
|---------|:--------------------:|--------|
| Tek dosyada küçük düzenleme | ❌ | Ana agent yeterli |
| 50+ dosyada kod araması | ✅ | Context izolasyonu gerekli |
| Paralel test ve lint çalıştırma | ✅ | Paralelleştirme avantajı |
| Basit soru-cevap | ❌ | Gereksiz overhead |
| Büyük refactoring projeleri | ✅ | Hem izolasyon hem paralellik |
| Farklı uzmanlıklar gerektiren görevler | ✅ | Özelleştirilmiş talimatlar |

```mermaid
flowchart TD
    START["Görev geldi"] --> Q1{"Görev karmaşık mı?<br/>(3+ adım)"}
    Q1 --> |"Hayır"| DIRECT["Ana agent ile yap"]
    Q1 --> |"Evet"| Q2{"Paralel yapılabilir mi?"}

    Q2 --> |"Evet"| PARALLEL["Paralel subagent'lar<br/>oluştur"]
    Q2 --> |"Hayır"| Q3{"Context window<br/>dolacak mı?"}

    Q3 --> |"Evet"| SINGLE_SUB["Tek subagent<br/>devret"]
    Q3 --> |"Hayır"| DIRECT

```

---

## Pratik Örnekler

### Örnek 1: Büyük Kod Tabanı Analizi

```bash
> Bu projedeki tüm deprecated API kullanımlarını bul

# Claude Code otomatik olarak bir Explore subagent başlatır:
# 1. Subagent tüm dosyaları tarar (Glob + Grep)
# 2. Deprecated kullanımları tespit eder
# 3. Özet raporu ana agent'a döndürür
# 4. Ana agent raporu kullanıcıya sunar
```

### Örnek 2: Paralel Kod İnceleme

```bash
> Bu PR'daki değişiklikleri incele: güvenlik, performans ve kod kalitesi açısından

# Claude Code üç paralel subagent başlatır:
# Subagent 1: Güvenlik açıklarını tarar
# Subagent 2: Performans sorunlarını analiz eder
# Subagent 3: Kod kalitesi ve SOLID uyumunu kontrol eder
# Sonuçlar birleştirilip sunulur
```

### Örnek 3: Çok Modüllü Test Yazma

```bash
> Tüm service katmanı için unit test yaz

# Claude Code her servis için ayrı subagent başlatır:
# Subagent 1: UserService testleri
# Subagent 2: OrderService testleri
# Subagent 3: PaymentService testleri
# Her subagent kendi servisine odaklanır
```

### Örnek 4: Proje Keşfi ve Dokümantasyon

```bash
> Bu projenin mimarisini analiz et ve API dokümantasyonu oluştur

# İki aşamalı subagent kullanımı:
# Aşama 1 - Explore subagent: Proje yapısını keşfeder
# Aşama 2 - GeneralPurpose subagent: Dokümantasyonu yazar
```

---

## Subagent ve Ana Agent Karşılaştırması

```mermaid
flowchart LR
    subgraph parent_ctx ["Ana Agent"]
        direction TB
        PA["✅ Kullanıcıyla doğrudan iletişim"]
        PB["✅ Tüm araçlara erişim"]
        PC["✅ Oturum bağlamını korur"]
        PD["❌ Sınırlı context window"]
        PE["❌ Tek seferde bir iş"]
    end

    subgraph sub_ctx ["Subagent"]
        direction TB
        SA["✅ İzole context window"]
        SB["✅ Paralel çalışabilir"]
        SC["✅ Özelleştirilebilir talimatlar"]
        SD["❌ Kullanıcıyla doğrudan iletişim yok"]
        SE["❌ Sadece ana agent'a rapor verir"]
    end

```

| Özellik | Ana Agent | Subagent |
|---------|-----------|----------|
| Context window | Paylaşımlı (kullanıcı ile) | Bağımsız (izole) |
| Kullanıcı etkileşimi | Doğrudan | Yok (ana agent üzerinden) |
| Araç erişimi | Tam | Kısıtlanabilir |
| Paralel çalışma | Sınırlı | Tam destek |
| Yaşam süresi | Oturum boyunca | Görev bitene kadar |
| Maliyet | Tek context | Ek context maliyeti |

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Subagent** | Ana agent tarafından başlatılan bağımsız, izole agent örneği |
| **Context Isolation** | Her subagent kendi bağlamında çalışır, ana context'i kirletmez |
| **Parallelization** | Birden fazla subagent eş zamanlı çalışabilir |
| **Tool Restrictions** | Subagent'ın erişebileceği araçlar sınırlandırılabilir |
| **Tanımlama** | Dahili (built-in), dosya sistemi (.claude/agents/), programatik (SDK) |

---

## Sonraki Adım

Subagent kavramını anladık. Şimdi Claude Code ile birlikte gelen dahili subagent'ları inceleyelim:

→ [Dahili Subagent'lar](./02-dahili-subagentlar.md)
