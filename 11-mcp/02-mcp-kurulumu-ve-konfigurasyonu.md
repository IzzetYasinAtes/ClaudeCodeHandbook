# MCP Kurulumu ve Konfigürasyonu

Claude Code'da MCP sunucularını yapılandırmak için `.mcp.json` dosyası kullanılır. Bu dosya, hangi sunucuların başlatılacağını, hangi komutlarla çalıştırılacağını ve hangi ortam değişkenlerinin kullanılacağını tanımlar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| MCP nedir, temel kavramlar | [MCP Nedir?](./01-mcp-nedir.md) |
| Claude Code kurulumu | [Bölüm 06](../06-claude-code-tanitim/03-kurulum-ve-gereksinimler.md) |
| JSON temel bilgisi | Harici kaynak |

---

## Konfigürasyon Dosyası: `.mcp.json`

MCP sunucuları `.mcp.json` adlı bir JSON dosyasında tanımlanır. Bu dosya iki farklı kapsamda yer alabilir:

```mermaid
flowchart TD
    subgraph proje ["Proje Kapsamı (Project Scope)"]
        PF[".mcp.json\n(Proje kökünde)"]
        PD["Sadece bu projede geçerli\nTakımla paylaşılabilir\nGit'e eklenebilir"]
    end

    subgraph kullanici ["Kullanıcı Kapsamı (User Scope)"]
        UF["~/.claude/.mcp.json\n(Home dizininde)"]
        UD["Tüm projelerde geçerli\nKişisel tercihler\nGit'e eklenmez"]
    end

    CC["Claude Code\nBaşlatılıyor"] --> PF
    CC --> UF
    PF --> MERGE["Birleştirilmiş\nMCP Sunucu Listesi"]
    UF --> MERGE

    style proje fill:#d4edda,stroke:#28a745
    style kullanici fill:#fff3cd,stroke:#ffc107
```

### Kapsam Karşılaştırması

| Özellik | Proje Kapsamı | Kullanıcı Kapsamı |
|---------|---------------|-------------------|
| **Dosya yolu** | `<proje-kökü>/.mcp.json` | `~/.claude/.mcp.json` |
| **Geçerlilik** | Sadece o proje | Tüm projeler |
| **Git'e dahil** | Evet (önerilir) | Hayır |
| **Kullanım** | Projeye özel sunucular | Kişisel araçlar |
| **Örnek** | Proje DB'si, proje GitHub'ı | Kişisel Slack, Brave Search |

---

## JSON Yapısı

`.mcp.json` dosyasının temel yapısı:

```jsonc
{
  "mcpServers": {
    "<sunucu-adi>": {
      "command": "<çalıştırılacak-komut>",
      "args": ["<argüman-1>", "<argüman-2>"],
      "env": {
        "<ORTAM_DEGISKENI>": "<deger>"
      }
    }
  }
}
```

### Alan Açıklamaları

| Alan | Zorunlu | Açıklama |
|------|---------|----------|
| `mcpServers` | ✅ | Tüm sunucu tanımlarını içeren üst düzey nesne |
| `<sunucu-adi>` | ✅ | Sunucu için benzersiz tanımlayıcı ad |
| `command` | ✅ | Sunucuyu başlatacak komut (`npx`, `node`, `python` vb.) |
| `args` | ❌ | Komuta geçirilecek argümanlar dizisi |
| `env` | ❌ | Sunucu sürecine aktarılacak ortam değişkenleri |

> **Not:** SSE taşıma yöntemi kullanan uzak sunucular için `command` ve `args` yerine `url` alanı kullanılır.

---

## Konfigürasyon Örnekleri

### Örnek 1: Basit stdio Sunucusu (Filesystem)

Dosya sistemi erişimi sağlayan temel bir MCP sunucusu:

```jsonc
// .mcp.json — Proje kökünde
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/yasin/projects/my-app/docs",
        "/Users/yasin/projects/my-app/config"
      ]
    }
  }
}
```

```bash
# Bu sunucu başlatıldığında Claude Code şunları yapabilir:
> docs klasöründeki tüm markdown dosyalarını listele
# ✅ MCP Filesystem → read_directory

> config/database.yml dosyasını oku
# ✅ MCP Filesystem → read_file
```

### Örnek 2: Ortam Değişkenleriyle GitHub Sunucusu

```jsonc
// .mcp.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

```bash
# Çalıştığını doğrulamak için:
> Bu reponun açık issue'larını listele
# Claude Code → MCP GitHub → list_issues → sonuç ✅
```

### Örnek 3: SSE (Uzak) Sunucusu

HTTP üzerinden çalışan uzak bir MCP sunucusu:

```jsonc
// .mcp.json
{
  "mcpServers": {
    "remote-analytics": {
      "url": "https://mcp.my-company.com/analytics/sse",
      "headers": {
        "Authorization": "Bearer <API_TOKEN>"
      }
    }
  }
}
```

### Örnek 4: Birden Fazla Sunucu

Bir projede birden çok MCP sunucusunu aynı anda yapılandırma:

```jsonc
// .mcp.json — Tam teşekküllü proje konfigürasyonu
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxxxxxxxxxx"
      }
    },
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:pass@localhost:5432/mydb"
      ]
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-xxxxxxxxxxxxxxxxxxxx",
        "SLACK_TEAM_ID": "T01234567"
      }
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "BSA_xxxxxxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

---

## Sunucu Yaşam Döngüsü

Bir MCP sunucusu Claude Code oturumunda şu yaşam döngüsünü izler:

```mermaid
flowchart TD
    START["Claude Code\nBaşlatıldı"] --> READ[".mcp.json\nOkunuyor"]
    READ --> VALIDATE{"Konfigürasyon\nGeçerli mi?"}

    VALIDATE -->|"Evet"| SPAWN["Sunucu Süreci\nBaşlatılıyor"]
    VALIDATE -->|"Hayır"| ERROR["Hata Mesajı\nKonsola yazılır"]

    SPAWN --> INIT["Başlatma\n(initialize)"]
    INIT --> TOOLS["Araç Listesi\nAlınıyor"]
    TOOLS --> READY["Sunucu Hazır ✅"]

    READY --> USE["Claude Code\nAraç Çağırıyor"]
    USE --> RESPONSE["Sunucu\nYanıt Veriyor"]
    RESPONSE --> USE

    USE --> END["Oturum Sona\nErdi"]
    END --> SHUTDOWN["Sunucu Süreci\nKapatılıyor"]

    style READY fill:#d4edda,stroke:#28a745
    style ERROR fill:#f8d7da,stroke:#dc3545
```

---

## Sunucu Yönetimi: `/mcp` Komutu

Claude Code oturumu içinde MCP sunucularını yönetmek için `/mcp` slash komutunu kullanabilirsiniz:

```bash
# MCP sunucu durumunu görüntüleme
> /mcp

# Gösterir:
# ┌─────────────────────────────────────────────────┐
# │ MCP Servers                                     │
# │                                                 │
# │ ✅ github       - 12 tools available            │
# │ ✅ postgres     - 4 tools available             │
# │ ❌ slack        - Connection failed              │
# │ ✅ brave-search - 1 tool available              │
# │                                                 │
# │ [R]estart / [D]isable / [E]nable / [C]ancel     │
# └─────────────────────────────────────────────────┘
```

### Yaygın `/mcp` İşlemleri

| İşlem | Açıklama |
|-------|----------|
| **Restart** | Seçilen sunucuyu yeniden başlatır |
| **Disable** | Sunucuyu bu oturum için devre dışı bırakır |
| **Enable** | Devre dışı sunucuyu tekrar etkinleştirir |

---

## `claude mcp` CLI Komutları

Komut satırından MCP sunucularını yönetmek için `claude mcp` alt komutlarını kullanabilirsiniz:

```bash
# Sunucu ekleme (kullanıcı kapsamı, varsayılan)
claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx -- \
  npx -y @modelcontextprotocol/server-github

# Sunucu ekleme (proje kapsamı)
claude mcp add postgres --scope project -- \
  npx -y @modelcontextprotocol/server-postgres \
  "postgresql://user:pass@localhost:5432/mydb"

# Sunucuları listeleme
claude mcp list

# Sunucu bilgisi görüntüleme
claude mcp get github

# Sunucu kaldırma
claude mcp remove github
```

```mermaid
flowchart LR
    CLI["claude mcp add"] --> SCOPE{"--scope?"}
    SCOPE -->|"user\n(varsayılan)"| USER_FILE["~/.claude/.mcp.json"]
    SCOPE -->|"project"| PROJ_FILE[".mcp.json\n(proje kökü)"]
```

---

## Güvenlik ve API Anahtarları

API anahtarlarını `.mcp.json` dosyasına doğrudan yazmak güvenlik riski oluşturabilir. Güvenli yönetim için şu stratejileri kullanın:

### Strateji 1: Ortam Değişkenleri ile Referans

```jsonc
// .mcp.json — Anahtar doğrudan dosyada DEĞİL
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

```bash
# Anahtarı shell ortam değişkeni olarak tanımlayın:
# Linux/macOS: ~/.bashrc veya ~/.zshrc
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxxxxxxxxx"

# Windows PowerShell:
$env:GITHUB_TOKEN = "ghp_xxxxxxxxxxxxxxxxxxxx"
```

### Strateji 2: Kullanıcı Kapsamını Kullanma

```bash
# Kişisel API anahtarlarını kullanıcı kapsamına ekleyin (Git'e dahil olmaz)
claude mcp add github --scope user \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx -- \
  npx -y @modelcontextprotocol/server-github

# Proje .mcp.json'ı Git'e eklenirken,
# ~/.claude/.mcp.json kişisel kalır
```

### Strateji 3: `.gitignore` Kontrolü

```gitignore
# .gitignore — API anahtarları içeren dosyayı hariç tutun
# Eğer .mcp.json'a anahtar yazdıysanız:
.mcp.json

# Ama daha iyi yaklaşım: anahtarları .env'de tutun
.env
.env.local
```

---

## Pratik Örnekler

### Örnek 1: Tam Proje Kurulumu — E-ticaret Uygulaması

```jsonc
// e-commerce-app/.mcp.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://dev:dev123@localhost:5432/ecommerce_dev"
      ]
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    }
  }
}
```

```bash
# Tipik kullanım senaryosu:
$ claude

> Bu projenin veritabanı şemasını göster
# Claude Code → MCP PostgreSQL → list_tables + describe_table

> orders tablosunda son 24 saatte kaç sipariş var?
# Claude Code → MCP PostgreSQL → run_query
# → "Son 24 saatte 847 sipariş var"

> Bu bilgiyi #dev-updates Slack kanalına özetle
# Claude Code → MCP Slack → send_message
```

### Örnek 2: Kademeli Kurulum — Sıfırdan Başlangıç

```bash
# Adım 1: Önce tek bir sunucu ile başlayın
claude mcp add github --scope project -- \
  npx -y @modelcontextprotocol/server-github

# Adım 2: Çalıştığını doğrulayın
claude
> /mcp
# ✅ github - 12 tools available

> Bu reponun açık issue'larını listele
# Çalışıyorsa → sonraki adım

# Adım 3: İhtiyaca göre yeni sunucular ekleyin
claude mcp add postgres --scope project -- \
  npx -y @modelcontextprotocol/server-postgres \
  "postgresql://user:pass@localhost:5432/mydb"

# Adım 4: Ekiple paylaşın
git add .mcp.json
git commit -m "MCP sunucu konfigürasyonu eklendi"
```

### Örnek 3: Sorun Giderme — Sunucu Bağlanamıyor

```bash
# Problem: MCP sunucusu bağlanamıyor
> /mcp
# ❌ postgres - Connection failed

# Kontrol 1: Sunucuyu elle çalıştırmayı deneyin
npx -y @modelcontextprotocol/server-postgres \
  "postgresql://user:pass@localhost:5432/mydb"
# Eğer hata verirse → bağlantı bilgilerini kontrol edin

# Kontrol 2: Node.js ve npx versiyonlarını kontrol edin
node --version   # v18+ gerekli
npx --version    # npm ile birlikte gelir

# Kontrol 3: .mcp.json sözdizimini doğrulayın
cat .mcp.json | python -m json.tool
# JSON parse hatası varsa → düzeltin

# Kontrol 4: Sunucuyu yeniden başlatın
> /mcp
# → [R]estart seçin
```

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **`.mcp.json`** | MCP sunucu konfigürasyon dosyası |
| **Proje kapsamı** | Proje kökünde, takımla paylaşılır |
| **Kullanıcı kapsamı** | `~/.claude/.mcp.json`, kişisel |
| **`command` + `args`** | stdio sunucu tanımı |
| **`url`** | SSE (uzak) sunucu tanımı |
| **`env`** | Sunucuya aktarılan ortam değişkenleri |
| **`/mcp`** | Oturum içi sunucu yönetim komutu |
| **`claude mcp`** | CLI üzerinden sunucu yönetimi |

---

## Sonraki Adım

Konfigürasyon dosyası yapısını öğrendik. Şimdi en popüler hazır MCP sunucularını ve kullanım senaryolarını inceleyelim:

→ [Hazır MCP Sunucuları](./03-hazir-mcp-sunuculari.md)
