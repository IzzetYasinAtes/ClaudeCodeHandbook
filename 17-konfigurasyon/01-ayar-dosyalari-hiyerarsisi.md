# Ayar Dosyaları Hiyerarşisi

Claude Code, farklı seviyelerde konfigürasyon dosyaları sunarak kişisel tercihleri, proje standartlarını ve kurumsal politikaları katmanlı bir şekilde yönetmenizi sağlar. Bu dosyalar belirli bir precedence (öncelik sırası) ile birleştirilir.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code temelleri | [Claude Code Nedir?](../06-claude-code-tanitim/01-claude-code-nedir.md) |
| CLAUDE.md dosyası | [CLAUDE.md Dosyası](../09-bellek-ve-baglam/01-claude-md-dosyasi.md) |
| İzin sistemi | [İzin Sistemi](../10-izinler-ve-guvenlik/01-izin-sistemi.md) |

---

## Dört Katmanlı Ayar Sistemi

Claude Code ayar dosyaları dört farklı scope (kapsam) seviyesinde çalışır:

```mermaid
flowchart TB
    subgraph hierarchy ["Ayar Dosyaları Hiyerarşisi (Artan Öncelik)"]
        direction TB
        USER["🧑 User Settings<br/>~/.claude/settings.json<br/>Kullanıcının tüm projelerinde geçerli"]
        PROJECT["📁 Project Settings<br/>.claude/settings.json<br/>Proje genelinde, Git'e commit edilir"]
        LOCAL["🔒 Local Settings<br/>.claude/settings.local.json<br/>Kişisel proje ayarları, .gitignore'da"]
        MANAGED["🏢 Managed Settings<br/>Kurumsal yönetimli<br/>Override edilemez"]
    end

    USER --> PROJECT --> LOCAL --> MANAGED

    style USER fill:#3498DB,color:#fff
    style PROJECT fill:#F39C12,color:#fff
    style LOCAL fill:#E74C3C,color:#fff
    style MANAGED fill:#8E44AD,color:#fff
```

### Öncelik Sırası (Precedence)

Aynı ayar birden fazla dosyada tanımlandığında, daha yüksek öncelikli dosya kazanır:

```
managed > local > project > user
```

| Öncelik | Dosya | Konum | Git'e Dahil | Geçersiz Kılınabilir |
|---------|-------|-------|-------------|----------------------|
| 1 (En düşük) | User Settings | `~/.claude/settings.json` | — | Evet |
| 2 | Project Settings | `.claude/settings.json` | ✅ Evet | Evet |
| 3 | Local Settings | `.claude/settings.local.json` | ❌ Hayır | Evet |
| 4 (En yüksek) | Managed Settings | Kurumsal politika | — | ❌ Hayır |

---

## User Settings (Kullanıcı Ayarları)

Kullanıcının tüm projelerinde geçerli olan kişisel tercihlerdir. Home dizininde saklanır.

**Konum:** `~/.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git status:*)",
      "Read",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo *)"
    ]
  },
  "env": {
    "CLAUDE_CODE_THEME": "dark"
  }
}
```

### Ne Zaman Kullanılır?

- Tüm projelerde geçerli olmasını istediğiniz izin kuralları
- Kişisel tercihler (tema, varsayılan model vb.)
- Her zaman izin vermek istediğiniz araç konfigürasyonları

---

## Project Settings (Proje Ayarları)

Proje genelinde tüm ekip üyeleri için geçerli olan ayarlardır. Git'e commit edilir ve takımla paylaşılır.

**Konum:** `<proje-kök>/.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(npx prettier:*)",
      "Bash(npx eslint:*)",
      "mcp__github__*"
    ],
    "deny": [
      "Bash(npx npm publish:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

### Ne Zaman Kullanılır?

- Projenin coding standartları ve kuralları
- Ekip genelinde paylaşılacak izinler
- Proje seviyesinde hook konfigürasyonları
- MCP sunucu tanımları

---

## Local Settings (Yerel Ayarlar)

Belirli bir projede yalnızca sizin için geçerli olan ayarlardır. `.gitignore`'a eklenmeli ve repository'ye commit edilmemelidir.

**Konum:** `<proje-kök>/.claude/settings.local.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(docker compose up:*)",
      "Bash(kubectl:*)"
    ]
  },
  "env": {
    "DATABASE_URL": "postgresql://localhost:5432/mydb_dev",
    "REDIS_URL": "redis://localhost:6379"
  }
}
```

### Ne Zaman Kullanılır?

- Kişisel geliştirme ortamına özgü ayarlar (veritabanı URL'leri vb.)
- Project settings'i geçersiz kılmak istediğiniz durumlar
- Denemek istediğiniz geçici ayarlar
- Diğer geliştiricileri etkilemeden yapılan özelleştirmeler

---

## Managed Settings (Yönetimli Ayarlar)

Kurum yöneticileri tarafından belirlenen, kullanıcıların geçersiz kılamayacağı ayarlardır. Enterprise (kurumsal) planlarda sunucu taraflı olarak uygulanır.

```mermaid
flowchart LR
    ADMIN["🏢 Kurum Yöneticisi<br/>Politika tanımlar"] --> SERVER["☁️ Sunucu<br/>Ayarları dağıtır"]
    SERVER --> CLIENT["💻 Claude Code<br/>Uygular"]
    CLIENT --> LOCK["🔒 Kullanıcı<br/>Override edemez"]

    style ADMIN fill:#8E44AD,color:#fff
    style SERVER fill:#2C3E50,color:#fff
    style CLIENT fill:#3498DB,color:#fff
    style LOCK fill:#E74C3C,color:#fff
```

### Managed Settings Özellikleri

| Özellik | Açıklama |
|---------|----------|
| Override edilemez | Kullanıcı veya proje ayarları bunları geçersiz kılamaz |
| Merkezi yönetim | Kurum genelinde tek noktadan kontrol |
| Otomatik dağıtım | Cihaz yönetimi altyapısı gerektirmez |
| Güvenlik politikaları | `deny` listesindeki araçlar hiçbir şekilde çalışamaz |

---

## Ayar Birleştirme (Merge) Mantığı

Farklı seviyelerdeki ayar dosyaları şu kurallara göre birleştirilir:

```mermaid
flowchart TD
    subgraph merge ["Ayar Birleştirme Süreci"]
        direction TB
        U["User Settings<br/>allow: [Read, Grep]<br/>deny: [Bash(rm -rf)]"]
        P["Project Settings<br/>allow: [Bash(npm run:*)]<br/>deny: [Bash(npm publish)]"]
        L["Local Settings<br/>allow: [Bash(docker:*)]"]
        M["Managed Settings<br/>deny: [Bash(curl|*)]"]
    end

    subgraph result ["Sonuç"]
        R["Birleştirilmiş Ayarlar<br/>allow: [Read, Grep, Bash(npm run:*), Bash(docker:*)]<br/>deny: [Bash(rm -rf), Bash(npm publish), Bash(curl|*)]"]
    end

    U --> R
    P --> R
    L --> R
    M --> R

    style R fill:#27AE60,color:#fff
```

### Birleştirme Kuralları

| Alan | Birleştirme Davranışı |
|------|----------------------|
| `permissions.allow` | Tüm seviyelerden birleştirilir (union) |
| `permissions.deny` | Tüm seviyelerden birleştirilir, managed olanlar kaldırılamaz |
| `hooks` | Her seviyeden hook'lar birlikte çalışır |
| `env` | Üst seviye alt seviyeyi geçersiz kılar |
| Skaler değerler | Üst seviye alt seviyeyi geçersiz kılar |

---

## Pratik Örnek: Tam Bir Kurulum

### Senaryo: Full-Stack Proje Ekibi

**1. User Settings** (`~/.claude/settings.json`):

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo *)"
    ]
  }
}
```

**2. Project Settings** (`.claude/settings.json`):

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(npx:*)",
      "mcp__github__*",
      "mcp__linear__*"
    ],
    "deny": [
      "Bash(npm publish:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

**3. Local Settings** (`.claude/settings.local.json`):

```json
{
  "permissions": {
    "allow": [
      "Bash(docker compose:*)",
      "Bash(psql:*)"
    ]
  },
  "env": {
    "DATABASE_URL": "postgresql://dev:dev@localhost:5432/myapp",
    "API_KEY": "dev-local-key-12345"
  }
}
```

Bu üç dosyanın birleştirilmiş sonucu:

```json
{
  "permissions": {
    "allow": [
      "Read", "Grep", "Bash(git:*)",
      "Bash(npm run:*)", "Bash(npx:*)",
      "mcp__github__*", "mcp__linear__*",
      "Bash(docker compose:*)", "Bash(psql:*)"
    ],
    "deny": [
      "Bash(rm -rf /)", "Bash(sudo *)",
      "Bash(npm publish:*)"
    ]
  }
}
```

---

## Dosya Konumlarını Doğrulama

Hangi ayar dosyalarının yüklendiğini görmek için:

```bash
# Debug modunda başlatarak yüklenen dosyaları görün
claude --debug

# Mevcut ayarları yazdırma
claude config list
```

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| `settings.local.json` Git'e commit etmek | `.gitignore`'a `.claude/settings.local.json` ekleyin |
| User settings'te proje spesifik ayar koymak | Proje ayarları için `.claude/settings.json` kullanın |
| Managed settings'i override etmeye çalışmak | Managed ayarlar kurum politikasıdır, yöneticiyle iletişime geçin |
| Dosya yolunu yanlış yazmak | `~/.claude/settings.json` (user), `.claude/settings.json` (project) |

---

## Özet

| Seviye | Dosya | Paylaşılır? | Override Edilir? | Kullanım Alanı |
|--------|-------|-------------|-----------------|----------------|
| User | `~/.claude/settings.json` | Hayır | Evet | Kişisel tercihler |
| Project | `.claude/settings.json` | Evet (Git) | Evet | Ekip standartları |
| Local | `.claude/settings.local.json` | Hayır | Evet | Yerel ortam ayarları |
| Managed | Kurumsal | Hayır | Hayır | Güvenlik politikaları |

---

## Sonraki Adım

Ayar dosyalarının yapısını anladıktan sonra, `settings.json` içindeki tüm anahtarları ve seçenekleri detaylı inceleyelim:

→ [Settings.json Referansı](./02-settings-json-referansi.md)
