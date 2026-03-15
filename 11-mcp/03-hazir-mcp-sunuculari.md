# Hazır MCP Sunucuları

MCP ekosistemi, topluluk ve resmi katkılarla büyüyen yüzlerce hazır sunucu barındırır. Bu bölümde en popüler ve kullanışlı MCP sunucularını, kurulumlarını ve gerçek kullanım örneklerini inceleyeceğiz.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| MCP nedir, client-server mimarisi | [MCP Nedir?](./01-mcp-nedir.md) |
| .mcp.json konfigürasyon yapısı | [Kurulum ve Konfigürasyon](./02-mcp-kurulumu-ve-konfigurasyonu.md) |
| Node.js ve npm/npx kurulumu | Harici kaynak |

---

## Sunucu Ekosistemi Genel Bakış

```mermaid
flowchart TD
    MCP_ECO["MCP Sunucu Ekosistemi"]

    MCP_ECO --> OFFICIAL["Resmi Sunucular<br/>(Anthropic / MCP Org)"]
    MCP_ECO --> COMMUNITY["Topluluk Sunucuları<br/>(Açık Kaynak)"]
    MCP_ECO --> CUSTOM["Özel Sunucular<br/>(Şirket içi)"]

    OFFICIAL --> O1["GitHub"]
    OFFICIAL --> O2["Slack"]
    OFFICIAL --> O3["Google Drive"]
    OFFICIAL --> O4["PostgreSQL"]
    OFFICIAL --> O5["Filesystem"]
    OFFICIAL --> O6["Brave Search"]
    OFFICIAL --> O7["Puppeteer"]

    COMMUNITY --> C1["MongoDB"]
    COMMUNITY --> C2["Jira"]
    COMMUNITY --> C3["Linear"]
    COMMUNITY --> C4["Notion"]
    COMMUNITY --> C5["1000+<br/>diğer..."]

    style OFFICIAL fill:#4A90D9,color:#fff
    style COMMUNITY fill:#27AE60,color:#fff
    style CUSTOM fill:#F39C12,color:#fff
```

---

## 1. GitHub — Repo Yönetimi

GitHub MCP sunucusu, repository yönetimi, issue takibi, pull request işlemleri ve daha fazlasını Claude Code'dan doğrudan yapmanızı sağlar.

### Kurulum

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

**Token oluşturma:** GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → `repo`, `issues`, `pull_requests` izinleri

### Sunduğu Araçlar

| Araç | Açıklama |
|------|----------|
| `list_issues` | Issue'ları listeleme |
| `get_issue` | Issue detayı görüntüleme |
| `create_issue` | Yeni issue oluşturma |
| `create_pull_request` | PR oluşturma |
| `list_pull_requests` | PR'ları listeleme |
| `get_pull_request_diff` | PR diff'ini görme |
| `search_repositories` | Repo arama |
| `get_file_contents` | Dosya içeriği okuma |
| `create_or_update_file` | Dosya oluşturma/güncelleme |
| `push_files` | Birden fazla dosya push'lama |
| `create_branch` | Branch oluşturma |
| `search_code` | Kod arama |

### Kullanım Örnekleri

```bash
# Issue yönetimi
> Bu repodaki "bug" etiketli açık issue'ları listele
> #42 numaralı issue'ya "v2.1'de düzeltilecek" yorumu ekle
> "Login sayfasında timeout hatası" başlığıyla yeni bir bug issue'su oluştur

# Pull Request işlemleri
> Açık PR'ları listele ve review durumlarını göster
> #15 numaralı PR'ın diff'ini incele ve code review yap
> feature/auth branch'ini main'e merge eden bir PR oluştur

# Kod keşfi
> Bu repodaki tüm .env.example dosyalarını bul
> README.md'deki kurulum talimatlarını oku
```

---

## 2. Slack — Mesajlaşma

Slack MCP sunucusu, kanalları okuma, mesaj gönderme ve arama yapma yeteneği sağlar.

### Kurulum

```jsonc
// .mcp.json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-xxxxxxxxxxxxxxxxxxxx",
        "SLACK_TEAM_ID": "T01234567"
      }
    }
  }
}
```

**Token oluşturma:** Slack API → Create New App → OAuth & Permissions → Bot Token Scopes: `channels:history`, `channels:read`, `chat:write`, `users:read`

### Sunduğu Araçlar

| Araç | Açıklama |
|------|----------|
| `list_channels` | Kanal listesi |
| `post_message` | Mesaj gönderme |
| `reply_to_thread` | Thread'e yanıt verme |
| `get_channel_history` | Kanal geçmişi okuma |
| `search_messages` | Mesaj arama |
| `get_users` | Kullanıcı listesi |
| `get_user_profile` | Kullanıcı profili |

### Kullanım Örnekleri

```bash
# Mesaj okuma ve arama
> #backend kanalındaki son 10 mesajı göster
> Slack'te "deployment hatası" ile ilgili mesajları ara
> @ahmet'in son gönderdiği mesajları bul

# Mesaj gönderme
> #dev-updates kanalına bugünkü sprint özeti gönder
> @ayse'nin mesajına thread'de "tamam, bakıyorum" yaz
```

---

## 3. Google Drive — Doküman Erişimi

Google Drive MCP sunucusu, Drive'daki dosyaları arama ve okuma yeteneği sağlar.

### Kurulum

```jsonc
// .mcp.json
{
  "mcpServers": {
    "google-drive": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-google-drive"],
      "env": {
        "GOOGLE_CLIENT_ID": "xxxxxxxxxxxx.apps.googleusercontent.com",
        "GOOGLE_CLIENT_SECRET": "GOCSPX-xxxxxxxxxxxxxxxxxxxx",
        "GOOGLE_REDIRECT_URI": "http://localhost:3000/oauth/callback"
      }
    }
  }
}
```

**Kurulum:** Google Cloud Console → APIs & Services → Credentials → OAuth 2.0 Client → Drive API etkinleştir

### Sunduğu Araçlar

| Araç | Açıklama |
|------|----------|
| `search_files` | Drive'da dosya arama |
| `read_file` | Dosya içeriği okuma |
| `list_files` | Klasör içeriği listeleme |

### Kullanım Örnekleri

```bash
# Doküman keşfi ve okuma
> Google Drive'da "API spesifikasyonu" dokümanını bul
> Tasarım dokümanını oku ve bu proje için gerekli endpoint'leri listele
> "Sprint Review" notlarındaki aksiyonları özetle
```

---

## 4. PostgreSQL — Veritabanı Sorguları

PostgreSQL MCP sunucusu, veritabanı şemasını keşfetme ve SQL sorguları çalıştırma yeteneği sağlar.

### Kurulum

```jsonc
// .mcp.json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://app_user:secure_pass@localhost:5432/myapp_dev"
      ]
    }
  }
}
```

> **Güvenlik:** Üretim (production) veritabanına bağlanmak yerine, salt okunur (read-only) bir kullanıcı ve geliştirme (development) veritabanı kullanmanız şiddetle önerilir.

### Sunduğu Araçlar

| Araç | Açıklama |
|------|----------|
| `query` | SQL sorgusu çalıştırma |
| `list_tables` | Tablo listesi |
| `describe_table` | Tablo şeması görüntüleme |

### Kullanım Örnekleri

```bash
# Şema keşfi
> Veritabanındaki tüm tabloları listele
> users tablosunun şemasını göster
> orders ve order_items tabloları arasındaki ilişkiyi açıkla

# Veri sorguları
> Son 7 günde kaç yeni kullanıcı kayıt olmuş?
> En çok sipariş veren 10 müşteriyi listele
> Ürün kategorilerine göre toplam satış tutarlarını göster

# Geliştirme desteği
> Bu tablo yapısına göre bir TypeScript interface oluştur
> Bu şemaya uygun seed data SQL'i yaz
> orders tablosuna "discount_code" kolonu eklemek için migration yaz
```

### Güvenli PostgreSQL Kullanıcısı Oluşturma

```sql
-- Geliştirme veritabanında salt okunur kullanıcı
CREATE ROLE mcp_readonly WITH LOGIN PASSWORD 'mcp_secure_pass';
GRANT CONNECT ON DATABASE myapp_dev TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT ON TABLES TO mcp_readonly;
```

```jsonc
// .mcp.json — Salt okunur bağlantı
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://mcp_readonly:mcp_secure_pass@localhost:5432/myapp_dev"
      ]
    }
  }
}
```

---

## 5. Filesystem — Dosya İşlemleri

Filesystem MCP sunucusu, belirli dizinlere kontrollü dosya erişimi sağlar. Claude Code'un dahili dosya araçlarından farklı olarak, projenin dışındaki dizinlere erişim açar.

### Kurulum

```jsonc
// .mcp.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/yasin/documents/specs",
        "/Users/yasin/documents/designs"
      ]
    }
  }
}
```

> **Not:** `args` dizisinde birden fazla dizin yolu belirtebilirsiniz. Sunucu, yalnızca belirtilen dizinlere erişim sağlar.

### Sunduğu Araçlar

| Araç | Açıklama |
|------|----------|
| `read_file` | Dosya okuma |
| `write_file` | Dosya yazma |
| `list_directory` | Dizin listeleme |
| `create_directory` | Dizin oluşturma |
| `move_file` | Dosya taşıma |
| `search_files` | Dosya arama |
| `get_file_info` | Dosya bilgisi |
| `read_multiple_files` | Birden fazla dosya okuma |

### Kullanım Örnekleri

```bash
# Proje dışı dosya erişimi
> specs klasöründeki API spesifikasyon dokümanını oku
> Bu spesifikasyona göre TypeScript interface'leri oluştur
> designs klasöründeki wireframe notlarını incele
```

---

## 6. Brave Search — Web Araması

Brave Search MCP sunucusu, güncel web araması yapma yeteneği sağlar. Claude Code'un eğitim verilerinde bulunmayan güncel bilgilere erişim için idealdir.

### Kurulum

```jsonc
// .mcp.json
{
  "mcpServers": {
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

**API Key:** [brave.com/search/api](https://brave.com/search/api/) → Free plan: 2000 sorgu/ay

### Sunduğu Araçlar

| Araç | Açıklama |
|------|----------|
| `brave_web_search` | Web araması |
| `brave_local_search` | Yerel işletme araması |

### Kullanım Örnekleri

```bash
# Güncel bilgi araştırma
> React 19'un yeni özellikleri neler?
> Next.js App Router ile ilgili en güncel best practice'leri araştır
> TypeScript 5.5'teki yeni type narrowing özelliklerini bul

# Hata çözümü
> "ECONNREFUSED 127.0.0.1:5432" hatası için çözüm ara
> "Module not found: @tanstack/react-query" sorunu nasıl çözülür?
```

---

## 7. Puppeteer — Tarayıcı Otomasyonu

Puppeteer MCP sunucusu, headless tarayıcı kontrolü sağlar. Web sayfalarını ziyaret etme, ekran görüntüsü alma ve form doldurma gibi işlemleri otomatikleştirir.

### Kurulum

```jsonc
// .mcp.json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

### Sunduğu Araçlar

| Araç | Açıklama |
|------|----------|
| `navigate` | URL'ye gitme |
| `screenshot` | Ekran görüntüsü alma |
| `click` | Element'e tıklama |
| `fill` | Form alanı doldurma |
| `evaluate` | JavaScript çalıştırma |
| `select` | Dropdown seçme |
| `hover` | Element üzerine gelme |

### Kullanım Örnekleri

```bash
# UI test ve doğrulama
> localhost:3000'e git ve ana sayfanın ekran görüntüsünü al
> Login formunu doldur (user: test@test.com, pass: test123) ve giriş yap
> Giriş sonrası dashboard sayfasının yüklendiğini doğrula

# Web scraping
> https://example.com/pricing sayfasındaki fiyatlandırma tablosunu oku
> Rakip ürünlerin özellik karşılaştırma tablosunu al
```

---

## Sunucu Karşılaştırma Tablosu

```mermaid
flowchart LR
    subgraph veri ["Veri Erişimi"]
        PostgreSQL
        Google_Drive["Google Drive"]
        Filesystem
    end

    subgraph iletisim ["İletişim"]
        GitHub
        Slack
    end

    subgraph arastirma ["Araştırma"]
        Brave_Search["Brave Search"]
        Puppeteer
    end

    style veri fill:#4A90D9,color:#fff
    style iletisim fill:#27AE60,color:#fff
    style arastirma fill:#F39C12,color:#fff
```

| Sunucu | Kategori | Araç Sayısı | Tipik Kullanım |
|--------|----------|-------------|----------------|
| **GitHub** | İletişim | ~12 | Repo yönetimi, PR, issue |
| **Slack** | İletişim | ~7 | Mesajlaşma, bildirim |
| **Google Drive** | Veri Erişimi | ~3 | Doküman okuma |
| **PostgreSQL** | Veri Erişimi | ~3 | DB sorgu, şema keşfi |
| **Filesystem** | Veri Erişimi | ~8 | Harici dosya erişimi |
| **Brave Search** | Araştırma | ~2 | Güncel web araması |
| **Puppeteer** | Araştırma | ~7 | Tarayıcı otomasyon |

---

## Pratik Örnekler

### Örnek 1: Full-Stack Geliştirme Ortamı

Bir full-stack projesinde tüm ihtiyaçları karşılayan kapsamlı konfigürasyon:

```jsonc
// fullstack-app/.mcp.json
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
        "postgresql://dev:dev123@localhost:5432/fullstack_dev"
      ]
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    },
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

```bash
# Gerçek iş akışı:
$ claude

> Veritabanındaki users tablosunun şemasını göster
# → PostgreSQL MCP → describe_table

> Bu şemaya göre bir User TypeScript interface'i oluştur
# → Claude Code dahili araçları ile dosya oluşturma

> React ile bir UserProfile componenti yaz ve localhost:3000'de test et
# → Dahili araçlar ile kod yazma
# → Puppeteer MCP ile ekran görüntüsü alma

> Her şey çalışıyorsa bir PR oluştur
# → GitHub MCP → create_pull_request
```

### Örnek 2: Belirli Bir Sunucunun Sorun Giderilmesi

```bash
# 1. Sunucu durumunu kontrol edin
> /mcp
# ❌ postgres - Connection failed

# 2. Bağlantı bilgilerini doğrulayın
> PostgreSQL'e bağlanamıyor. Bağlantı bilgilerini kontrol et.
# Claude Code .mcp.json'ı okur ve bağlantı URL'sini kontrol eder

# 3. Alternatif yaklaşımlar:
# a) Sunucuyu yeniden başlatın
> /mcp → [R]estart

# b) Bağlantı URL'sini güncelleyin
# .mcp.json'da port veya credentials'ı düzeltin

# c) Sunucuyu komut satırından test edin
npx -y @modelcontextprotocol/server-postgres \
  "postgresql://dev:dev123@localhost:5432/fullstack_dev"
```

### Örnek 3: Yeni Topluluk Sunucusu Ekleme

```bash
# Topluluk sunucularını keşfetme
# https://github.com/modelcontextprotocol/servers — resmi dizin
# https://mcp.so — topluluk dizini

# Örnek: Linear (proje yönetimi) sunucusu ekleme
claude mcp add linear --scope project \
  -e LINEAR_API_KEY=lin_api_xxxx -- \
  npx -y @modelcontextprotocol/server-linear

# Doğrulama
claude
> /mcp
# ✅ linear - 8 tools available

> Linear'daki aktif sprint'teki görevlerimi listele
# ✅ Çalışıyor!
```

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Resmi sunucular** | Anthropic/MCP org tarafından bakımı yapılan güvenilir sunucular |
| **Topluluk sunucuları** | Açık kaynak topluluğunun geliştirdiği 1000+ sunucu |
| **GitHub MCP** | Repo, issue, PR yönetimi |
| **PostgreSQL MCP** | Veritabanı sorgu ve şema keşfi |
| **Brave Search MCP** | Güncel web araması |
| **Puppeteer MCP** | Headless tarayıcı otomasyonu |
| **Salt okunur kullanıcı** | Veritabanı güvenliği için önerilen yaklaşım |

---

## Sonraki Adım

Hazır sunucuları tanıdık. Çok sayıda MCP aracıyla çalışırken performansı optimize etmek için Tool Search mekanizmasını inceleyelim:

→ [Tool Search](./04-mcp-tool-search.md)
