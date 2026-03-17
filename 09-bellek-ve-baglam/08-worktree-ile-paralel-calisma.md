# Worktree ile Paralel Çalışma

**Git worktree** (çalışma ağacı), aynı Git deposunun birden fazla çalışma kopyasını oluşturmanızı sağlayan bir Git özelliğidir. Claude Code bu özelliği `--worktree` / `-w` bayrağıyla entegre ederek, birden fazla görevi **paralel** ve **izole** şekilde yürütmenize olanak tanır.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Oturum yönetimi | [Oturum Yönetimi](./06-oturum-yonetimi.md) |
| Checkpointing | [Checkpointing](./07-checkpointing.md) |
| Git temel bilgisi | Harici kaynak |

---

## Neden Paralel Çalışma?

Geleneksel iş akışında, bir görev bitmeden diğerine geçmek **branch switching** (dal değiştirme) gerektirir ve bu risklidir:

```mermaid
flowchart TD
    subgraph seri ["❌ Seri Çalışma"]
        direction TB
        S1["Görev 1: Auth modülü<br/>(30 dk)"] --> S2["Görev 2: UI düzeltme<br/>(15 dk)"]
        S2 --> S3["Görev 3: API endpoint<br/>(20 dk)"]
        S3 --> TOTAL_S["Toplam: 65 dk<br/>(sıralı)"]
    end

    subgraph paralel ["✅ Paralel Çalışma (Worktree)"]
        direction TB
        P1["Görev 1: Auth modülü<br/>(Worktree 1)"]
        P2["Görev 2: UI düzeltme<br/>(Worktree 2)"]
        P3["Görev 3: API endpoint<br/>(Worktree 3)"]
        P1 --> TOTAL_P["Toplam: ~30 dk<br/>(paralel)"]
        P2 --> TOTAL_P
        P3 --> TOTAL_P
    end

    style TOTAL_S fill:#E74C3C,color:#fff
    style TOTAL_P fill:#27AE60,color:#fff
```

---

## Worktree Nasıl Çalışır?

Claude Code `--worktree` bayrağı kullanıldığında:

1. `.claude/worktrees/<isim>/` altında izole bir dizin oluşturur
2. `worktree-<isim>` adında yeni bir Git branch'i oluşturur
3. Claude Code bu izole dizinde çalışır
4. Oturum bittiğinde otomatik temizlik yapılır

```mermaid
flowchart TD
    subgraph main_repo ["Ana Repo (my-project/)"]
        MAIN["main branch<br/>(ana çalışma dizini)"]
    end

    subgraph worktrees [".claude/worktrees/"]
        WT1["auth-feature/<br/>(worktree-auth-feature branch)<br/>Görev: Auth modülü"]
        WT2["ui-fix/<br/>(worktree-ui-fix branch)<br/>Görev: UI düzeltme"]
        WT3["api-endpoint/<br/>(worktree-api-endpoint branch)<br/>Görev: API endpoint"]
    end

    MAIN --> WT1
    MAIN --> WT2
    MAIN --> WT3

    subgraph sessions ["Claude Code Oturumları"]
        CS1["Oturum 1<br/>(auth-feature üzerinde)"]
        CS2["Oturum 2<br/>(ui-fix üzerinde)"]
        CS3["Oturum 3<br/>(api-endpoint üzerinde)"]
    end

    WT1 --> CS1
    WT2 --> CS2
    WT3 --> CS3

    style main_repo fill:#4A90D9,color:#fff
    style worktrees fill:#F39C12,color:#fff
```

---

## Temel Kullanım

### Worktree ile Oturum Başlatma

```bash
# Yeni bir worktree ile Claude Code başlat
$ claude --worktree auth-feature "Auth modülüne rate limiting ekle"

# Kısa form
$ claude -w auth-feature "Auth modülüne rate limiting ekle"

# İsim vermeden (otomatik isim atanır)
$ claude -w "Login sayfasındaki hatayı düzelt"
```

### Oluşturulan Yapı

```
my-project/
├── .claude/
│   └── worktrees/
│       └── auth-feature/        # İzole çalışma dizini
│           ├── src/
│           ├── package.json
│           └── ...              # Ana repo'nun kopyası
├── src/                         # Ana çalışma dizini (dokunulmaz)
├── package.json
└── ...
```

---

## Paralel Oturum Diyagramı

3-5+ paralel oturumu aynı anda çalıştırabilirsiniz:

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant T1 as Terminal 1
    participant T2 as Terminal 2
    participant T3 as Terminal 3
    participant GIT as Git Repo

    Note over U: 3 paralel görev başlatıyor

    par Paralel Başlatma
        U->>T1: claude -w auth "Rate limiting ekle"
        T1->>GIT: worktree-auth branch oluştur
        GIT-->>T1: .claude/worktrees/auth/

    and
        U->>T2: claude -w ui-fix "Dark mode düzelt"
        T2->>GIT: worktree-ui-fix branch oluştur
        GIT-->>T2: .claude/worktrees/ui-fix/

    and
        U->>T3: claude -w api "Pagination ekle"
        T3->>GIT: worktree-api branch oluştur
        GIT-->>T3: .claude/worktrees/api/
    end

    Note over T1,T3: Her oturum izole çalışır

    par Paralel Çalışma
        T1->>T1: Auth modülünde çalışıyor...
    and
        T2->>T2: UI dosyalarında çalışıyor...
    and
        T3->>T3: API endpoint'lerinde çalışıyor...
    end

    Note over U: Görevler tamamlanınca

    T1-->>U: "Rate limiting eklendi ✅"
    T2-->>U: "Dark mode düzeltildi ✅"
    T3-->>U: "Pagination eklendi ✅"

    Note over U,GIT: Branch'leri merge et
    U->>GIT: git merge worktree-auth
    U->>GIT: git merge worktree-ui-fix
    U->>GIT: git merge worktree-api
```

---

## Desktop App ile Otomatik İzolasyon

Claude Code **Desktop** uygulaması kullanıldığında, her oturum otomatik olarak izole edilir:

```mermaid
flowchart TD
    subgraph desktop ["Claude Code Desktop"]
        DA["Oturum otomatik<br/>izole edilir"]
        DA --> ISO1["Her görev kendi<br/>worktree'sinde çalışır"]
        ISO1 --> SAFE["Ana branch<br/>guvenli kalır"]
    end

    subgraph terminal ["Claude Code Terminal"]
        TA["-w bayrağı ile<br/>manuel izolasyon"]
        TA --> ISO2["İstenen görevler<br/>için worktree oluşturulur"]
    end

    style desktop fill:#4A90D9,color:#fff
    style terminal fill:#F39C12,color:#fff
```

---

## Subagent İzolasyonu

Claude Code'un **subagent** (alt ajan) sistemi de worktree ile izole çalışabilir:

```mermaid
flowchart TD
    MAIN["Ana Claude Code<br/>Oturumu"] --> SUB1["Subagent 1<br/>isolation: worktree<br/>(auth görevi)"]
    MAIN --> SUB2["Subagent 2<br/>isolation: worktree<br/>(test görevi)"]
    MAIN --> SUB3["Subagent 3<br/>isolation: worktree<br/>(docs görevi)"]

    SUB1 --> WT1["worktree-sub-1/"]
    SUB2 --> WT2["worktree-sub-2/"]
    SUB3 --> WT3["worktree-sub-3/"]

    SUB1 -->|"Tamamlandı"| MERGE["Ana oturuma<br/>birleştir"]
    SUB2 -->|"Tamamlandı"| MERGE
    SUB3 -->|"Tamamlandı"| MERGE

    style MAIN fill:#4A90D9,color:#fff
    style MERGE fill:#27AE60,color:#fff
```

Subagent oluştururken izolasyon modunu belirtebilirsiniz:

```bash
# CLAUDE.md veya oturum içinde subagent izolasyonu
> Bu 3 görevi paralel subagent'lar ile yap,
> her biri kendi worktree'sinde çalışsın
```

---

## Pratik Örnek 1: Feature Geliştirme

3 farklı feature'ı paralel olarak geliştirme:

```bash
# Terminal 1: Kullanıcı profil sayfası
$ claude -w user-profile "Kullanıcı profil sayfası oluştur:
  - Avatar yükleme
  - Bilgi düzenleme formu
  - Aktivite geçmişi"

# Terminal 2: Bildirim sistemi
$ claude -w notifications "Bildirim sistemi kur:
  - WebSocket ile gerçek zamanlı bildirimler
  - Bildirim tercihleri sayfası
  - Email bildirimleri"

# Terminal 3: Arama özelliği
$ claude -w search "Tam metin arama özelliği ekle:
  - Elasticsearch entegrasyonu
  - Arama sonuç sayfası
  - Otomatik tamamlama"
```

Her terminal bağımsız çalışır, birbirini etkilemez.

---

## Pratik Örnek 2: Bug Fix + Feature Paralel

Acil bir bug fix ile planlı bir feature'ı aynı anda çalıştırma:

```bash
# Terminal 1: Acil bug fix (ana branch'ten)
$ claude -w hotfix "Login sayfasında 500 hatası var,
  acil düzelt ve test et"

# Terminal 2: Planlı feature (devam eden iş)
$ claude -w feature-dashboard "Dashboard sayfasını geliştirmeye
  devam et, grafikleri ekle"
```

```
main          ●───────────────────●── merge hotfix ──●── merge feature ──●
               \                 /                  /
worktree-      ●── fix: login ──●                  /
hotfix              500                           /
               \                                 /
worktree-      ●── feat: chart ──●── feat: ─────●
feature-           component        dashboard
dashboard                           layout
```

---

## Pratik Örnek 3: Kod İnceleme ve Refactoring

Birden fazla modülü paralel olarak refactor etme:

```bash
# Görev dosyası oluşturun
$ cat > refactor-tasks.md << 'EOF'
1. auth modülü: SOLID prensiplere göre yeniden yapılandır
2. database katmanı: Repository pattern uygula
3. API handler'lar: Middleware chain'i optimize et
EOF

# 3 paralel worktree başlatın
$ claude -w refactor-auth "auth modülünü SOLID prensiplere göre yeniden yapılandır"
$ claude -w refactor-db "database katmanına repository pattern uygula"
$ claude -w refactor-api "API handler'ları için middleware chain'i optimize et"
```

---

## Otomatik Temizlik

Worktree oturumları sona erdiğinde otomatik temizlik yapılır:

```mermaid
flowchart TD
    SESSION["Oturum sona erdi"] --> CLEANUP["Otomatik Temizlik"]
    CLEANUP --> REMOVE_DIR["Worktree dizini<br/>silinir"]
    CLEANUP --> KEEP_BRANCH["Git branch'i<br/>korunur"]
    CLEANUP --> KEEP_CHANGES["Commit'lenmiş değişiklikler<br/>korunur"]

    KEEP_BRANCH --> MERGE{"Merge<br/>etmek ister<br/>misiniz?"}
    MERGE -->|"Evet"| GIT_MERGE["git merge worktree-<isim>"]
    MERGE -->|"Hayır"| DELETE_BRANCH["git branch -d worktree-<isim>"]

    style CLEANUP fill:#F39C12,color:#fff
    style GIT_MERGE fill:#27AE60,color:#fff
```

### Manuel Temizlik

Gerekirse worktree'leri manuel olarak temizleyebilirsiniz:

```bash
# Aktif worktree'leri listele
$ git worktree list

# Belirli bir worktree'yi kaldır
$ git worktree remove .claude/worktrees/auth-feature

# Tüm ölü worktree referanslarını temizle
$ git worktree prune
```

---

## Worktree Kullanım Rehberi

```mermaid
flowchart TD
    Q1{"Kaç görev<br/>var?"} -->|"1 görev"| NORMAL["Normal oturum<br/>(worktree gerekmez)"]
    Q1 -->|"2-3 görev"| Q2{"Görevler<br/>bağımsız mı?"}
    Q1 -->|"3+ görev"| MULTI["Kesinlikle worktree<br/>kullanın"]

    Q2 -->|"Evet, bağımsız"| WORKTREE["Worktree ile<br/>paralel çalışın"]
    Q2 -->|"Hayır, birbirine<br/>bağlı"| SERIAL["Sıralı oturumlar<br/>(biri bitsin diğeri başlasın)"]

    style NORMAL fill:#95A5A6,color:#fff
    style WORKTREE fill:#27AE60,color:#fff
    style MULTI fill:#27AE60,color:#fff
    style SERIAL fill:#F39C12,color:#fff
```

| Senaryo | Worktree Gerekli mi? | Neden |
|---------|:--------------------:|-------|
| Tek bir bug fix | ❌ | Tek oturum yeterli |
| 3 bağımsız feature | ✅ | Zaman kazanır |
| Acil hotfix + feature | ✅ | Ana iş akışını bozmaz |
| Sıralı migration adımları | ❌ | Adımlar birbirine bağlı |
| Paralel refactoring | ✅ | Farklı modüller izole çalışır |
| Kod inceleme + geliştirme | ✅ | Biri okuma, diğeri yazma |

---

## Sınırlamalar ve Dikkat Edilecekler

| Konu | Açıklama |
|------|----------|
| **Aynı dosya çakışması** | İki worktree aynı dosyayı değiştirirse merge conflict oluşur |
| **Disk alanı** | Her worktree projenin bir kopyasını oluşturur |
| **node_modules** | Her worktree kendi `node_modules`'ını kurmalıdır |
| **Bağımlı görevler** | Sıralı bağımlılığı olan görevler worktree'ye uygun değildir |
| **CPU/RAM** | Paralel oturumlar sistem kaynaklarını tüketir |

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Git Worktree** | Aynı repo'nun birden fazla çalışma kopyasını oluşturma |
| **--worktree / -w** | Claude Code'da worktree ile izole oturum başlatma |
| **İzolasyon** | Her worktree kendi branch'inde bağımsız çalışır |
| **Paralel oturumlar** | 3-5+ görevi aynı anda yürütme |
| **Subagent isolation** | Alt ajanların worktree ile izole çalışması |
| **Otomatik temizlik** | Oturum bittiğinde worktree dizini otomatik silinir |
| **Desktop uygulaması** | Oturumlar otomatik olarak izole edilir |

---

## Bölüm Sonu

Bu, "Bellek ve Bağlam Yönetimi" bölümünün son konusuydu. CLAUDE.md'den context window yönetimine, oturum yönetiminden paralel çalışmaya kadar Claude Code'un bellek ve bağlam mekanizmalarını kapsamlı şekilde inceledik.

→ Sonraki bölüm: [10 - İzinler ve Güvenlik](../10-izinler-ve-guvenlik/README.md)
