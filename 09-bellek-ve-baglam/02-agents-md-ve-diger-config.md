# AGENTS.md ve Diğer Config Dosyaları

**AGENTS.md**, birden fazla AI coding aracının ortak olarak desteklediği bir **cross-tool standard** (çapraz araç standardı) dosyasıdır. Claude Code bu dosyayı CLAUDE.md'ye **fallback** (yedek) olarak destekler. Bu bölümde AGENTS.md, `.cursorrules`, `.github/copilot-instructions.md` ve diğer config dosyalarını karşılaştıracağız.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| CLAUDE.md dosyası | [CLAUDE.md Dosyası](./01-claude-md-dosyasi.md) |
| Claude Code temel kullanım | [Bölüm 06](../06-claude-code-tanitim/README.md) |

---

## Config Dosyası Ekosistemi

AI destekli geliştirme araçlarının her biri kendi config dosya formatını kullanır, ancak sektörde ortak bir standart oluşmaya başlamıştır:

```mermaid
flowchart TD
    subgraph standart ["Çapraz Araç Standardı"]
        AGENTS["AGENTS.md<br/>(60K+ repo'da kullanılıyor)"]
    end

    subgraph araca_ozel ["Araca Özel Config"]
        CLAUDE_MD["CLAUDE.md<br/>(Claude Code)"]
        CURSOR[".cursorrules<br/>(Cursor)"]
        COPILOT[".github/copilot-instructions.md<br/>(GitHub Copilot)"]
        WINDSURF[".windsurfrules<br/>(Windsurf)"]
        CONTINUE[".continuerc.json<br/>(Continue.dev)"]
    end

    AGENTS -->|"Fallback olarak<br/>destekler"| CLAUDE_MD
    AGENTS -->|"Destekler"| CURSOR
    AGENTS -->|"Destekler"| CONTINUE

```

---

## AGENTS.md Nedir?

AGENTS.md, projenin kök dizinine yerleştirilen ve **birden fazla AI aracı** tarafından okunan bir talimat dosyasıdır. 60.000'den fazla GitHub deposunda kullanılmaktadır.

### Claude Code'da AGENTS.md Davranışı

```mermaid
flowchart TD
    START["Oturum Başlangıcı"] --> CHECK_CLAUDE{"CLAUDE.md<br/>var mı?"}
    CHECK_CLAUDE -->|"Evet"| LOAD_CLAUDE["CLAUDE.md yükle<br/>(birincil)"]
    CHECK_CLAUDE -->|"Hayır"| CHECK_AGENTS{"AGENTS.md<br/>var mı?"}

    LOAD_CLAUDE --> CHECK_AGENTS2{"AGENTS.md<br/>de var mı?"}
    CHECK_AGENTS2 -->|"Evet"| LOAD_BOTH["İkisini de yükle<br/>(CLAUDE.md öncelikli)"]
    CHECK_AGENTS2 -->|"Hayır"| ONLY_CLAUDE["Yalnızca CLAUDE.md"]

    CHECK_AGENTS -->|"Evet"| LOAD_AGENTS["AGENTS.md yükle<br/>(fallback)"]
    CHECK_AGENTS -->|"Hayır"| NO_CONFIG["Config dosyası yok"]

```

> **Önemli:** Hem CLAUDE.md hem AGENTS.md varsa, ikisi de yüklenir. Çelişen kurallar olursa CLAUDE.md öncelik kazanır.

---

## Uyumluluk Matrisi

Hangi araç hangi config dosyasını destekler:

| Config Dosyası | Claude Code | Cursor | GitHub Copilot | Windsurf | Continue.dev |
|---------------|:-----------:|:------:|:--------------:|:--------:|:------------:|
| **CLAUDE.md** | ✅ Birincil | ❌ | ❌ | ❌ | ❌ |
| **AGENTS.md** | ✅ Fallback | ✅ | ❌ | ❌ | ✅ |
| **.cursorrules** | ❌ | ✅ Birincil | ❌ | ❌ | ❌ |
| **.github/copilot-instructions.md** | ❌ | ❌ | ✅ Birincil | ❌ | ❌ |
| **.windsurfrules** | ❌ | ❌ | ❌ | ✅ Birincil | ❌ |
| **.continuerc.json** | ❌ | ❌ | ❌ | ❌ | ✅ Birincil |
| **.cursor/rules/*.md** | ❌ | ✅ | ❌ | ❌ | ❌ |
| **.claude/rules/*.md** | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## Config Dosyalarının Karşılaştırması

```mermaid
flowchart LR
    subgraph claude_config ["Claude Code"]
        direction TB
        CC1["CLAUDE.md<br/>(Markdown, esnek format)"]
        CC2[".claude/rules/*.md<br/>(Kapsam bazlı kurallar)"]
        CC3[".claude/settings.json<br/>(İzin ve ayarlar)"]
    end

    subgraph cursor_config ["Cursor"]
        direction TB
        CU1[".cursorrules<br/>(Serbest metin)"]
        CU2[".cursor/rules/*.md<br/>(Kapsam bazlı)"]
    end

    subgraph copilot_config ["GitHub Copilot"]
        direction TB
        CO1[".github/copilot-instructions.md<br/>(Markdown)"]
    end

    subgraph ortak ["Çapraz Araç"]
        direction TB
        AG1["AGENTS.md<br/>(Markdown)"]
    end

    AG1 -.->|"Desteklenir"| claude_config
    AG1 -.->|"Desteklenir"| cursor_config
```

### Detaylı Karşılaştırma

| Özellik | CLAUDE.md | AGENTS.md | .cursorrules | copilot-instructions.md |
|---------|-----------|-----------|--------------|------------------------|
| **Format** | Markdown | Markdown | Serbest metin | Markdown |
| **Konum** | Proje kökü | Proje kökü | Proje kökü | `.github/` dizini |
| **Hiyerarşi** | User + Project + Sub | Sadece proje kökü | Sadece proje kökü | Sadece `.github/` |
| **Alt dizin desteği** | ✅ Evet | ❌ Hayır | ❌ Hayır | ❌ Hayır |
| **Kural dosyaları** | `.claude/rules/` | ❌ | `.cursor/rules/` | ❌ |
| **Araç sayısı** | 1 (Claude Code) | 3+ araç | 1 (Cursor) | 1 (Copilot) |
| **Topluluk kullanımı** | Artıyor | 60K+ repo | Yaygın | Yaygın |

---

## Pratik Örnek 1: Çoklu Araç Projesi

Takımınızda farklı araçlar kullanan geliştiriciler varsa, birden fazla config dosyası tutabilirsiniz:

```
my-project/
├── CLAUDE.md              # Claude Code kullanıcıları için
├── AGENTS.md              # Tüm araçlar için ortak kurallar
├── .cursorrules           # Cursor kullanıcıları için
├── .github/
│   └── copilot-instructions.md  # Copilot kullanıcıları için
└── src/
    └── ...
```

### AGENTS.md (Ortak kurallar)

```markdown
# AGENTS.md — Proje Geliştirme Kuralları

## Kodlama Standartları
- TypeScript strict mode kullanılmalı
- Fonksiyon isimleri camelCase, sınıf isimleri PascalCase
- Dosya isimleri kebab-case
- Maximum satır uzunluğu: 100 karakter

## Test Kuralları
- Her yeni fonksiyon için birim testi yaz
- Test dosyaları `__tests__/` dizininde
- Test isimlendirmesi: `should [action] when [condition]`

## Git
- Conventional Commits formatı kullan
- PR'lar squash merge ile birleştirilir
```

### CLAUDE.md (Claude Code'a özel eklemeler)

```markdown
# CLAUDE.md

# Ortak kurallar AGENTS.md'de tanımlı, burası Claude'a özel

## Komutlar
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint:fix`

## Claude'a Özel Talimatlar
- Türkçe yanıt ver
- Dosya oluşturmadan önce mevcut yapıyı kontrol et
- Testleri çalıştırdıktan sonra sonuçları raporla
```

---

## Pratik Örnek 2: Yalnızca AGENTS.md Kullanan Proje

Eğer takımda birden fazla araç kullanılıyorsa ve ortak bir standart isteniyorsa:

```markdown
# AGENTS.md

## Proje: Müşteri Yönetim Sistemi
- **Stack:** Next.js 15, Prisma, PostgreSQL
- **Monorepo:** Turborepo ile yönetiliyor

## Paketler
- `apps/web` — Next.js frontend
- `apps/api` — Express API
- `packages/shared` — Ortak tipler ve yardımcı fonksiyonlar
- `packages/ui` — Paylaşılan UI bileşenleri

## Kurallar
- Her paket kendi `tsconfig.json` dosyasına sahip
- Paylaşılan tipler `packages/shared/types/` altında
- API endpoint'leri RESTful konvansiyona uymalı
- Veritabanı migration'ları `apps/api/prisma/migrations/` altında

## Yasaklar
- `any` tipi kullanma
- `console.log` commit'leme
- Doğrudan veritabanı sorgusu yazma (Prisma client kullan)
- Relative import yerine alias kullan (@shared/, @ui/)
```

---

## Pratik Örnek 3: .cursorrules'dan CLAUDE.md'ye Geçiş

Mevcut `.cursorrules` dosyanız varsa, Claude Code için CLAUDE.md'ye dönüştürmeniz önerilir:

```mermaid
flowchart LR
    CR[".cursorrules<br/>(mevcut)"] --> ANALYZE["İçeriği analiz et"]
    ANALYZE --> COMMON["Ortak kuralları<br/>AGENTS.md'ye taşı"]
    ANALYZE --> SPECIFIC["Claude'a özel kuralları<br/>CLAUDE.md'ye yaz"]
    COMMON --> RESULT["Hem Cursor<br/>hem Claude Code<br/>çalışır"]
    SPECIFIC --> RESULT

```

**Dönüştürme adımları:**

1. `.cursorrules` dosyasını okuyun
2. Araçtan bağımsız kuralları `AGENTS.md`'ye taşıyın
3. Claude Code'a özel komutlar ve talimatları `CLAUDE.md`'ye yazın
4. `.cursorrules`'ı olduğu gibi bırakın (Cursor kullanıcıları için)

---

## Hangi Dosyayı Ne Zaman Kullanmalı?

```mermaid
flowchart TD
    START["Config dosyası<br/>seçimi"] --> Q1{"Sadece Claude Code<br/>mu kullanıyorsunuz?"}
    Q1 -->|"Evet"| CLAUDE["CLAUDE.md<br/>kullanın"]
    Q1 -->|"Hayır"| Q2{"Takımda farklı<br/>araçlar var mı?"}
    Q2 -->|"Evet"| BOTH["AGENTS.md (ortak)<br/>+ araca özel dosyalar"]
    Q2 -->|"Hayır, tek araç"| Q3{"Hangi araç?"}
    Q3 -->|"Cursor"| CURSOR_F[".cursorrules<br/>kullanın"]
    Q3 -->|"Copilot"| COPILOT_F["copilot-instructions.md<br/>kullanın"]

```

| Senaryo | Önerilen Dosya(lar) |
|---------|-------------------|
| Yalnızca Claude Code | `CLAUDE.md` |
| Yalnızca Cursor | `.cursorrules` veya `.cursor/rules/` |
| Yalnızca Copilot | `.github/copilot-instructions.md` |
| Claude Code + Cursor | `AGENTS.md` + `CLAUDE.md` + `.cursorrules` |
| Tüm araçlar | `AGENTS.md` (ortak) + araca özel dosyalar |

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **AGENTS.md** | 60K+ repo'da kullanılan çapraz araç standardı |
| **Fallback** | Claude Code, CLAUDE.md yoksa AGENTS.md'yi okur |
| **Öncelik** | CLAUDE.md > AGENTS.md (ikisi de varsa) |
| **.cursorrules** | Cursor'a özel config dosyası |
| **copilot-instructions.md** | GitHub Copilot'a özel config dosyası |
| **Çoklu araç stratejisi** | AGENTS.md ortak + araca özel dosyalar |

---

## Sonraki Adım

Config dosyalarının ötesinde, belirli dosya tiplerine ve dizinlere özel kurallar tanımlamak için kurallar dizinini inceleyelim:

→ [Kurallar Dizini (.claude/rules/)](./03-kurallar-dizini.md)
