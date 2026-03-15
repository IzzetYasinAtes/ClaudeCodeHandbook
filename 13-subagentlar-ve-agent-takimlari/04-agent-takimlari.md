# Agent Takımları (Agent Teams)

Agent Teams (agent takımları), birden fazla Claude Code örneğinin **koordineli bir şekilde** birlikte çalışarak büyük ölçekli görevleri yürütmesini sağlayan bir orkestrasyon modelidir. Tek bir agent'ın kapasitesini aşan projelerde — büyük refactoring'ler, çok modüllü geliştirmeler ve çapraz repo koordinasyonlarında — agent takımları devreye girer.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Subagent nedir | [Subagent Nedir?](./01-subagent-nedir.md) |
| Dahili subagent'lar | [Dahili Subagent'lar](./02-dahili-subagentlar.md) |
| Özel subagent oluşturma | [Özel Subagent Oluşturma](./03-ozel-subagent-olusturma.md) |
| Worktree ile paralel çalışma | [Worktree](../09-bellek-ve-baglam/08-worktree-ile-paralel-calisma.md) |

---

## Agent Takımı Nedir?

Bir agent takımı, bir **orkestratör agent** (conductor) tarafından yönetilen birden fazla **çalışan agent'tan** (worker) oluşur. Orkestratör büyük görevi parçalara ayırır, her parçayı uygun bir worker'a atar ve sonuçları birleştirir.

```mermaid
flowchart TD
    USER["👤 Kullanıcı"] --> ORCH["🎯 Orkestratör Agent\n(Conductor)"]

    ORCH --> |"Modül A"| W1["🔧 Worker 1\nFrontend"]
    ORCH --> |"Modül B"| W2["🔧 Worker 2\nBackend API"]
    ORCH --> |"Modül C"| W3["🔧 Worker 3\nVeritabanı"]
    ORCH --> |"Test"| W4["🔧 Worker 4\nTest & QA"]

    W1 --> |"Sonuç"| ORCH
    W2 --> |"Sonuç"| ORCH
    W3 --> |"Sonuç"| ORCH
    W4 --> |"Sonuç"| ORCH

    ORCH --> |"Birleşik rapor"| USER

    style ORCH fill:#E74C3C,color:#fff
    style W1 fill:#3498DB,color:#fff
    style W2 fill:#27AE60,color:#fff
    style W3 fill:#E67E22,color:#fff
    style W4 fill:#9B59B6,color:#fff
```

---

## Takım Yapısı ve Roller

### Orkestratör (Conductor)

Orkestratör, takımın "beyni"dir:

- Büyük görevi alt görevlere böler
- Her alt görevi uygun worker'a atar
- Worker'lar arası bağımlılıkları yönetir
- Sonuçları toplar ve birleştirir
- Çakışmaları çözer

### Worker'lar (Çalışanlar)

Her worker belirli bir uzmanlık alanında çalışır:

- Kendi izole context window'unda çalışır
- Atanan alt göreve odaklanır
- Tamamlandığında sonucu orkestratöre raporlar

```mermaid
flowchart LR
    subgraph orch_role ["Orkestratör Sorumlulukları"]
        direction TB
        O1["📋 Görev planlama"]
        O2["📦 Görev dağıtımı"]
        O3["🔄 Bağımlılık yönetimi"]
        O4["🔗 Sonuç birleştirme"]
        O5["⚠️ Çakışma çözme"]
    end

    subgraph worker_role ["Worker Sorumlulukları"]
        direction TB
        W1["🎯 Atanan göreve odaklanma"]
        W2["🔧 Bağımsız çalışma"]
        W3["📊 Sonuç raporlama"]
        W4["✅ Kendi testlerini çalıştırma"]
    end

    style orch_role fill:#fce4ec,stroke:#e91e63
    style worker_role fill:#e8f5e9,stroke:#4caf50
```

---

## Takım Orkestrasyon Modelleri

### 1. Yıldız Modeli (Star Pattern)

Tüm worker'lar doğrudan orkestratöre bağlıdır. En yaygın ve basit model.

```mermaid
flowchart TD
    ORCH["🎯 Orkestratör"] --> W1["Worker 1"]
    ORCH --> W2["Worker 2"]
    ORCH --> W3["Worker 3"]
    ORCH --> W4["Worker 4"]

    W1 --> ORCH
    W2 --> ORCH
    W3 --> ORCH
    W4 --> ORCH

    style ORCH fill:#E74C3C,color:#fff
```

**Ne zaman:** Bağımsız, paralel çalışabilen görevler.

### 2. Pipeline Modeli (Pipeline Pattern)

Worker'lar sıralı bir zincir oluşturur. Bir öncekinin çıktısı sonrakinin girdisi olur.

```mermaid
flowchart LR
    ORCH["🎯 Orkestratör"] --> W1["Worker 1\nAnaliz"]
    W1 --> W2["Worker 2\nTasarım"]
    W2 --> W3["Worker 3\nUygulama"]
    W3 --> W4["Worker 4\nTest"]
    W4 --> ORCH

    style ORCH fill:#E74C3C,color:#fff
```

**Ne zaman:** Sıralı bağımlılığı olan görevler (analiz → tasarım → uygulama → test).

### 3. Hiyerarşik Model (Hierarchical Pattern)

Alt orkestratörler kendi worker gruplarını yönetir.

```mermaid
flowchart TD
    ORCH["🎯 Ana Orkestratör"] --> SUB1["🎯 Frontend Lideri"]
    ORCH --> SUB2["🎯 Backend Lideri"]

    SUB1 --> W1["Worker\nReact"]
    SUB1 --> W2["Worker\nCSS"]

    SUB2 --> W3["Worker\nAPI"]
    SUB2 --> W4["Worker\nDB"]

    W1 --> SUB1
    W2 --> SUB1
    W3 --> SUB2
    W4 --> SUB2

    SUB1 --> ORCH
    SUB2 --> ORCH

    style ORCH fill:#E74C3C,color:#fff
    style SUB1 fill:#E67E22,color:#fff
    style SUB2 fill:#E67E22,color:#fff
```

**Ne zaman:** Çok büyük projeler, farklı alan uzmanlıkları gereken durumlar.

---

## Kullanım Senaryoları

### Senaryo 1: Büyük Ölçekli Refactoring

**Görev:** Monolitik uygulamada ORM değişikliği (Sequelize → Prisma)

```mermaid
flowchart TD
    ORCH["🎯 Orkestratör\nMigrasyon Planlayıcı"] --> ANALYZE["🔍 Worker 1\nMevcut Şema Analizi"]
    ORCH --> SCHEMA["📋 Worker 2\nPrisma Schema Oluşturma"]
    ORCH --> MODELS["🔧 Worker 3\nModel Dönüşümü"]
    ORCH --> QUERIES["🔧 Worker 4\nSorgu Dönüşümü"]
    ORCH --> TESTS["✅ Worker 5\nTest Güncelleme"]

    ANALYZE --> |"Mevcut tablo/ilişki haritası"| SCHEMA
    SCHEMA --> |"Yeni şema"| MODELS
    SCHEMA --> |"Yeni şema"| QUERIES
    MODELS --> TESTS
    QUERIES --> TESTS

    style ORCH fill:#E74C3C,color:#fff
```

```bash
# Kullanıcı isteği:
> Projemizdeki Sequelize ORM'ini Prisma'ya taşı.
> 47 model ve 120+ sorgu var. Agent takımı ile yap.

# Orkestratör planı:
# 1. Worker 1: Mevcut Sequelize modellerini ve ilişkilerini analiz et
# 2. Worker 2: Prisma schema.prisma dosyasını oluştur
# 3. Worker 3: Model dosyalarını Prisma Client'a dönüştür (paralel)
# 4. Worker 4: Raw sorguları Prisma sorguya çevir (paralel)
# 5. Worker 5: Tüm testleri güncelle ve çalıştır
```

### Senaryo 2: Paralel Feature Development (Paralel Özellik Geliştirme)

**Görev:** Bir sprint'teki birden fazla özelliği aynı anda geliştirme

```mermaid
flowchart TD
    ORCH["🎯 Orkestratör\nSprint Planlayıcı"] --> F1["🔧 Worker 1\nKullanıcı Profili"]
    ORCH --> F2["🔧 Worker 2\nBildirim Sistemi"]
    ORCH --> F3["🔧 Worker 3\nRaporlama Modülü"]

    F1 --> |"Git branch:\nfeature/user-profile"| MERGE["🔗 Orkestratör\nMerge & Test"]
    F2 --> |"Git branch:\nfeature/notifications"| MERGE
    F3 --> |"Git branch:\nfeature/reporting"| MERGE

    MERGE --> FINAL["✅ Integration Test"]

    style ORCH fill:#E74C3C,color:#fff
    style MERGE fill:#27AE60,color:#fff
```

```bash
# Kullanıcı isteği:
> Sprint'teki şu 3 özelliği paralel olarak geliştir:
> 1. Kullanıcı profil sayfası
> 2. Push bildirim sistemi
> 3. Aylık raporlama modülü

# Her worker kendi git branch'inde çalışır
# Orkestratör sonunda merge eder ve integration test çalıştırır
```

### Senaryo 3: Multi-Repo Koordinasyonu (Çoklu Repo Koordinasyonu)

**Görev:** Bir API değişikliğinin birden fazla repo'yu etkilediği durum

```mermaid
flowchart TD
    ORCH["🎯 Orkestratör\nAPI Değişiklik Koordinatörü"] --> W1["🔧 Worker 1\nBackend API Repo\nEndpoint değişikliği"]
    ORCH --> W2["🔧 Worker 2\nFrontend Repo\nClient güncelleme"]
    ORCH --> W3["🔧 Worker 3\nMobile Repo\nSDK güncelleme"]
    ORCH --> W4["🔧 Worker 4\nDocs Repo\nAPI dokümantasyonu"]

    W1 --> |"Yeni API kontratı"| W2
    W1 --> |"Yeni API kontratı"| W3
    W1 --> |"Yeni API kontratı"| W4

    style ORCH fill:#E74C3C,color:#fff
```

```bash
# Kullanıcı isteği:
> User API'sine 'preferences' alanı ekle.
> Backend, frontend, mobile ve docs repo'larının hepsini güncelle.

# Orkestratör:
# 1. Backend worker: API endpoint'ini günceller, migration oluşturur
# 2. Diğer worker'lar Backend API kontratını referans alır
# 3. Frontend worker: React bileşenlerini günceller
# 4. Mobile worker: SDK'yı günceller
# 5. Docs worker: API dokümantasyonunu günceller
```

---

## Takım Koordinasyon Mekanizmaları

### Paylaşılan Bağlam (Shared Context)

Worker'lar arası bilgi aktarımı orkestratör üzerinden gerçekleşir:

```mermaid
sequenceDiagram
    participant O as Orkestratör
    participant W1 as Worker 1
    participant W2 as Worker 2

    O->>W1: "API şemasını analiz et"
    activate W1
    W1-->>O: API şeması raporu
    deactivate W1

    Note over O: Şema raporunu W2'ye ilet

    O->>W2: "Bu API şemasına göre<br/>client SDK oluştur"
    activate W2
    Note over W2: W1'in çıktısını<br/>girdi olarak kullanır
    W2-->>O: SDK oluşturuldu
    deactivate W2
```

### Çakışma Yönetimi (Conflict Resolution)

Birden fazla worker aynı dosyayı değiştirmeye çalışırsa:

```mermaid
flowchart TD
    W1["Worker 1\nDosyaA'yı değiştirdi"] --> CHECK{"Çakışma\nvar mı?"}
    W2["Worker 2\nDosyaA'yı değiştirdi"] --> CHECK

    CHECK --> |"Evet"| RESOLVE["Orkestratör\nçakışmayı çözer"]
    CHECK --> |"Hayır"| MERGE["Doğrudan birleştir"]

    RESOLVE --> STRATEGIES{"Çözüm Stratejisi"}
    STRATEGIES --> S1["Sıralı uygulama"]
    STRATEGIES --> S2["Manuel birleştirme"]
    STRATEGIES --> S3["Bir worker'ı önceliklendir"]

    style CHECK fill:#ffc107,color:#000
    style RESOLVE fill:#E74C3C,color:#fff
```

### Worktree ile İzolasyon

Her worker kendi git worktree'sinde çalışarak çakışma riskini minimize eder:

```mermaid
flowchart LR
    REPO["Ana Repository"] --> WT1["Worktree 1\nfeature/auth\nWorker 1"]
    REPO --> WT2["Worktree 2\nfeature/billing\nWorker 2"]
    REPO --> WT3["Worktree 3\nfeature/search\nWorker 3"]

    WT1 --> MERGE["Main branch\nmerge"]
    WT2 --> MERGE
    WT3 --> MERGE

    style REPO fill:#4A90D9,color:#fff
    style MERGE fill:#27AE60,color:#fff
```

---

## Agent Takımı Oluşturma Adımları

```mermaid
flowchart TD
    S1["1. Görevi analiz et"] --> S2["2. Alt görevleri belirle"]
    S2 --> S3["3. Bağımlılıkları haritle"]
    S3 --> S4["4. Worker rollerini tanımla"]
    S4 --> S5["5. Orkestrasyon modelini seç"]
    S5 --> S6["6. Her worker için\nagent tanımı oluştur"]
    S6 --> S7["7. Çalıştır ve izle"]
    S7 --> S8["8. Sonuçları birleştir"]

    style S1 fill:#3498DB,color:#fff
    style S5 fill:#E67E22,color:#fff
    style S8 fill:#27AE60,color:#fff
```

### Adım Adım Uygulama

**1. Görevi Analiz Et**
```bash
> Bu e-ticaret platformunu Next.js 15'e yükselt.
> 120 bileşen, 45 API route, 30 test dosyası var.
```

**2. Alt Görevleri Belirle**
- Paket bağımlılıklarını güncelle
- App Router'a geçiş yap
- Server Component'lara dönüştür
- API Route'ları güncelle
- Testleri güncelle

**3. Worker Rollerini Tanımla**

```
.claude/agents/
├── nextjs-upgrade-deps.md      # Bağımlılık güncelleme
├── nextjs-upgrade-router.md    # Router dönüşümü
├── nextjs-upgrade-components.md # Bileşen dönüşümü
├── nextjs-upgrade-api.md       # API Route güncelleme
└── nextjs-upgrade-tests.md     # Test güncelleme
```

---

## Pratik Örnekler

### Örnek 1: Kod Tabanı Sağlık Kontrolü

```bash
> Tüm projenin sağlık kontrolünü yap:
> güvenlik, performans, kod kalitesi ve test kapsamı

# Agent Takımı:
# Orkestratör: Projeyi modüllere ayırır
# Worker 1 (security-auditor): Güvenlik denetimi
# Worker 2 (perf-analyzer): Performans analizi
# Worker 3 (code-reviewer): Kod kalitesi incelemesi
# Worker 4 (test-coverage): Test kapsam analizi
# Sonuç: Birleşik sağlık raporu
```

### Örnek 2: Dil / Framework Migasyonu

```bash
> JavaScript projesini TypeScript'e dönüştür.
> 85 dosya, strict mode ile.

# Agent Takımı:
# Orkestratör: Dosyaları modüllere göre gruplar
# Worker 1: src/models/ dizinini dönüştür
# Worker 2: src/services/ dizinini dönüştür
# Worker 3: src/controllers/ dizinini dönüştür
# Worker 4: src/utils/ dizinini dönüştür
# Worker 5: tsconfig ve build pipeline güncelle
# Worker 6: Tüm testleri güncelle ve çalıştır
```

### Örnek 3: Çoklu Mikroservis Güncellemesi

```bash
> Tüm mikroservislerde logging kütüphanesini winston'dan pino'ya geçir.
> 8 mikroservis, her birinde farklı konfigürasyon.

# Agent Takımı:
# Orkestratör: Her mikroservis için bir worker atar
# Worker 1-8: Her biri bir mikroservisi dönüştürür
# Her worker: Bağımlılık güncelle → Konfigürasyon değiştir
#             → Import'ları güncelle → Test çalıştır
```

### Örnek 4: Monorepo'da Cross-Package Değişiklik

```bash
> Shared types paketindeki User interface'ine 'role' alanı ekle
> ve tüm bağımlı paketleri güncelle.

# Agent Takımı:
# Orkestratör: Bağımlılık grafiğini çıkarır
# Worker 1: @shared/types paketini günceller
# Worker 2: @app/frontend — UI bileşenlerini günceller
# Worker 3: @app/backend — API handler'larını günceller
# Worker 4: @app/admin — Admin panelini günceller
# Orkestratör: Tüm paketleri build eder ve test eder
```

---

## Takım Boyutu ve Sınırlar

| Parametre | Önerilen Değer | Açıklama |
|-----------|:--------------:|----------|
| Maksimum worker sayısı | 4-6 | Daha fazlası koordinasyon overhead'i artırır |
| Minimum görev büyüklüğü | 3+ dosya | Daha küçük görevler için subagent yeterli |
| Worktree kullanımı | Paralel yazma işlemlerinde | Çakışma riskini ortadan kaldırır |
| Orkestratör modeli | Güçlü model | Planlama ve koordinasyon kapasitesi kritik |
| Worker modeli | Göreve uygun | Basit görevler için hafif model yeterli |

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Agent Team** | Birden fazla Claude Code örneğinin koordineli çalışması |
| **Orkestratör** | Görev planlama ve sonuç birleştirmeden sorumlu ana agent |
| **Worker** | Belirli bir alt göreve odaklanan çalışan agent |
| **Yıldız modeli** | Bağımsız görevler için — paralel worker'lar |
| **Pipeline modeli** | Sıralı bağımlılıklar için — zincir yapısı |
| **Hiyerarşik model** | Büyük projeler için — alt orkestratörler |
| **Worktree** | Her worker'ın kendi git branch'inde çalışması |

---

## Sonraki Adım

Agent takımlarının konseptini ve organizasyonunu öğrendik. Şimdi Agent aracının teknik detaylarına ve SDK ile programatik kullanımına bakalım:

→ [Agent Tool Kullanımı](./05-agent-tool-kullanimi.md)
