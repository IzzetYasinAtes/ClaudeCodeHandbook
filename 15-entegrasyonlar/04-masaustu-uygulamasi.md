# Claude Code Masaüstü Uygulaması

Claude Code Desktop App (masaüstü uygulaması), Claude Code deneyimini terminal dışına taşıyarak görsel bir arayüz sunar. Parallel sessions (paralel oturumlar) ile Git izolasyonu, visual diff review (görsel fark inceleme), app previews (uygulama ön izleme), PR monitoring (PR izleme) ve gelişmiş permission modes (izin modları) gibi güçlü özellikler içerir.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code temelleri | [Claude Code Nedir](../06-claude-code-tanitim/01-claude-code-nedir.md) |
| Git temel bilgisi | Harici kaynak |
| İzin sistemi | [İzin Sistemi](../10-izinler-ve-guvenlik/01-izin-sistemi.md) |

---

## Genel Bakış

```mermaid
flowchart TB
    subgraph desktop ["Claude Code Desktop App"]
        direction TB
        UI["Görsel Arayüz<br/>Chat + kod editör"]
        PS["Paralel Oturumlar<br/>Git worktree izolasyonu"]
        DR["Diff Review<br/>Görsel fark inceleme"]
        AP["App Preview<br/>Canlı uygulama ön izleme"]
        PR["PR Monitoring<br/>PR durumu takibi"]
        PM["Permission Modes<br/>İzin yönetimi"]
        CN["Connectors<br/>Dış servis bağlantıları"]
    end

    style desktop fill:#F3E8FF,stroke:#7C3AED
```

---

## Kurulum

### Platform Desteği

| Platform | Durum | İndirme |
|----------|-------|---------|
| macOS (Apple Silicon) | ✅ GA | claude.ai/download |
| macOS (Intel) | ✅ GA | claude.ai/download |
| Windows | ✅ GA | claude.ai/download |
| Linux (.deb) | ✅ GA | claude.ai/download |

### Kurulum Adımları

```mermaid
flowchart LR
    A["claude.ai/download<br/>adresine gidin"] --> B["Platformunuzu<br/>seçin"]
    B --> C["İndirin ve<br/>kurun"]
    C --> D["Giriş yapın<br/>(Anthropic hesabı)"]
    D --> E["Proje klasörü<br/>seçin"]

    style A fill:#3B82F6,color:#fff
    style E fill:#10B981,color:#fff
```

---

## Paralel Oturumlar ve Git İzolasyonu

Desktop app'in en güçlü özelliklerinden biri, birden fazla oturumun aynı anda ve birbirinden bağımsız çalışabilmesidir:

### Nasıl Çalışır?

```mermaid
flowchart TB
    REPO["Ana Repo<br/>main branch"]

    subgraph sessions ["Paralel Oturumlar"]
        S1["Oturum 1<br/>Özellik: Auth sistemi<br/>(worktree: feature/auth)"]
        S2["Oturum 2<br/>Bug fix: #234<br/>(worktree: fix/234)"]
        S3["Oturum 3<br/>Refactoring: DB katmanı<br/>(worktree: refactor/db)"]
    end

    REPO --> S1
    REPO --> S2
    REPO --> S3

    S1 -->|"PR oluştur"| PR1["PR #45"]
    S2 -->|"PR oluştur"| PR2["PR #46"]
    S3 -->|"PR oluştur"| PR3["PR #47"]

    style REPO fill:#3B82F6,color:#fff
    style S1 fill:#10B981,color:#fff
    style S2 fill:#F59E0B,color:#fff
    style S3 fill:#EF4444,color:#fff
```

Her oturum için:
- **Ayrı Git worktree** oluşturulur — dosya değişiklikleri birbirini etkilemez
- **Bağımsız branch** — her oturum kendi branch'inde çalışır
- **İzole ortam** — bir oturumdaki hata diğerlerini etkilemez
- **Paralel çalışma** — birden fazla görev eş zamanlı ilerler

### Paralel Oturum Başlatma

1. Desktop app'te `+ New Session` butonuna tıklayın
2. Proje klasörünü seçin (veya mevcut projeyi kullanın)
3. Görev tanımını yazın
4. Claude Code otomatik olarak yeni bir worktree ve branch oluşturur

---

## Visual Diff Review

Dosya değişikliklerini görsel olarak inceleme ve onaylama:

```mermaid
sequenceDiagram
    participant CC as Claude Code
    participant UI as Desktop App
    participant U as Kullanıcı

    CC->>UI: Dosya düzenlemesi
    UI->>U: Görsel diff göster
    Note over UI: Yeşil: Eklenen satırlar
    Note over UI: Kırmızı: Silinen satırlar
    Note over UI: Mavi: Değiştirilen satırlar

    alt Tek tek onay
        U->>UI: Her hunk için Accept/Reject
    else Toplu onay
        U->>UI: Accept All / Reject All
    end
    
    UI->>CC: Onay durumunu bildir
```

### Diff Görünüm Modları

| Mod | Açıklama | Kullanım |
|-----|----------|----------|
| **Side-by-side** | Eski ve yeni kod yan yana | Büyük değişikliklerde |
| **Inline** | Değişiklikler tek sütunda | Küçük değişikliklerde |
| **Summary** | Sadece dosya listesi ve istatistik | Genel bakış için |

---

## App Preview (Uygulama Ön İzleme)

Claude Code web uygulamaları üzerinde çalışırken, Desktop app canlı ön izleme sunar:

```mermaid
flowchart LR
    CC["Claude Code<br/>dev server başlatır"]
    CC --> PREV["App Preview<br/>Paneli"]
    PREV --> |"localhost:3000"| WEB["Web Uygulaması<br/>Canlı Görünüm"]

    U["Kullanıcı"]
    U -->|"Bu butonu sola taşı"| CC
    CC -->|"Kod güncelle"| PREV
    PREV -->|"Anlık yenileme"| WEB

    style CC fill:#7C3AED,color:#fff
    style WEB fill:#10B981,color:#fff
```

Desteklenen framework'ler:
- React (Next.js, Vite, CRA)
- Vue.js (Nuxt, Vite)
- Angular
- Svelte/SvelteKit
- Statik HTML/CSS/JS

---

## PR Monitoring (PR İzleme)

Desktop app açık PR'ların durumunu izler ve güncellemeleri gösterir:

| Özellik | Açıklama |
|---------|----------|
| PR Durumu | Açık, inceleniyor, onaylandı, birleştirildi |
| CI/CD Sonuçları | GitHub Actions / GitLab CI sonuçları |
| Review Yorumları | Kod inceleme yorumları bildirim olarak |
| Çakışma Uyarısı | Merge conflict tespit edildiğinde uyarı |
| Otomatik Düzeltme | Review yorumlarına göre otomatik düzeltme önerisi |

```mermaid
flowchart TB
    PR["Açık PR"] --> MON["Desktop App<br/>PR Monitor"]
    MON --> S1["✅ CI Başarılı"]
    MON --> S2["💬 Yeni Review"]
    MON --> S3["⚠️ Conflict"]
    MON --> S4["🔀 Merge Hazır"]

    S2 -->|"Otomatik düzelt"| CC["Claude Code<br/>Düzeltme uygula"]
    S3 -->|"Çakışma çöz"| CC

    style PR fill:#3B82F6,color:#fff
    style CC fill:#7C3AED,color:#fff
```

---

## Permission Modes (İzin Modları)

Desktop app'te üç izin modu bulunur:

```mermaid
flowchart LR
    subgraph modes ["İzin Modları"]
        direction TB
        SM["🛡️ Supervised<br/>(Denetimli)"]
        BM["⚡ Balanced<br/>(Dengeli)"]
        AM["🚀 Autonomous<br/>(Otonom)"]
    end

    SM -->|"Her işlem için<br/>onay gerekir"| LOW["Düşük Risk<br/>Yeni projeler"]
    BM -->|"Okuma serbest<br/>yazma onaylı"| MED["Orta Risk<br/>Günlük geliştirme"]
    AM -->|"Tüm işlemler<br/>otomatik"| HIGH["Yüksek Risk<br/>Güvenilir görevler"]

    style SM fill:#10B981,color:#fff
    style BM fill:#F59E0B,color:#fff
    style AM fill:#EF4444,color:#fff
```

| Mod | Okuma | Yazma | Terminal | Git |
|-----|-------|-------|----------|-----|
| **Supervised** | Onay gerekir | Onay gerekir | Onay gerekir | Onay gerekir |
| **Balanced** | Otomatik | Onay gerekir | Onay gerekir | Onay gerekir |
| **Autonomous** | Otomatik | Otomatik | Otomatik | Otomatik |

---

## Connectors (Bağlayıcılar)

Desktop app, dış servislerle entegrasyon için connector'lar sunar:

| Connector | İşlev | Kullanım |
|-----------|-------|----------|
| GitHub | Repo, PR, Issue erişimi | PR oluşturma, issue yönetimi |
| GitLab | Repo, MR erişimi | Merge request otomasyonu |
| Jira | Issue takibi | Görev durumu güncelleme |
| Linear | Issue takibi | Görev yönetimi |
| Slack | Mesajlaşma | Bildirim ve durum paylaşımı |
| Sentry | Hata izleme | Bug raporlarını analiz etme |

---

## Enterprise Konfigürasyonu

Kurumsal ortamlar için özel yapılandırma seçenekleri:

```json
{
  "enterprise": {
    "allowedModels": ["claude-sonnet-4-20250514"],
    "maxParallelSessions": 5,
    "defaultPermissionMode": "balanced",
    "proxy": {
      "host": "proxy.company.com",
      "port": 8080
    },
    "sso": {
      "provider": "okta",
      "domain": "company.okta.com"
    },
    "audit": {
      "enabled": true,
      "destination": "s3://audit-logs/"
    }
  }
}
```

---

## Pratik Örnekler

### Örnek 1: Paralel Özellik Geliştirme

```mermaid
flowchart TB
    START["Proje: E-ticaret App"]
    
    START --> S1["Oturum 1:<br/>Ödeme sistemi entegrasyonu"]
    START --> S2["Oturum 2:<br/>Kullanıcı profil sayfası"]
    START --> S3["Oturum 3:<br/>Arama optimizasyonu"]

    S1 --> PR1["PR #101<br/>+450 satır"]
    S2 --> PR2["PR #102<br/>+200 satır"]
    S3 --> PR3["PR #103<br/>+150 satır"]

    PR1 --> REV["Review & Merge"]
    PR2 --> REV
    PR3 --> REV

    style START fill:#3B82F6,color:#fff
    style REV fill:#10B981,color:#fff
```

1. Desktop app'te 3 yeni oturum açın
2. Her birine farklı bir görev atayın
3. Claude Code her oturum için ayrı branch oluşturur
4. Oturumlar paralel çalışır
5. Her biri tamamlandığında PR oluşturulur

### Örnek 2: PR Review Yorumlarını İşleme

1. Desktop app PR monitor'da yeni review yorumu bildirimi gelir
2. Yoruma tıklayın — ilgili oturum açılır
3. Claude Code yorumu okur ve düzeltme önerir
4. Diff review ile düzeltmeyi onaylayın
5. Claude Code commit eder ve PR'ı günceller

---

## Özet

| Özellik | Açıklama |
|---------|----------|
| **Paralel Oturumlar** | Git worktree ile izole, eş zamanlı çalışma |
| **Visual Diff** | Görsel fark inceleme ve onaylama |
| **App Preview** | Web uygulaması canlı ön izleme |
| **PR Monitoring** | PR durumu, CI sonuçları, review takibi |
| **Permission Modes** | Supervised, Balanced, Autonomous |
| **Connectors** | GitHub, GitLab, Jira, Slack vb. entegrasyonlar |
| **Enterprise** | SSO, proxy, audit log, model kısıtlamaları |

---

## Sonraki Adım

Chrome tarayıcı bağlantısı ile web uygulaması testi ve debug işlemlerini inceleyelim:

→ [Chrome Entegrasyonu](./05-chrome-entegrasyonu.md)
