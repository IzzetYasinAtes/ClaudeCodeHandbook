# MCP Örnekleri ve Tool Search

MCP (Model Context Protocol) sunucularının kategorize edilmiş örnekleri, kurulum komutları ve Tool Search mekanizmasının hızlı referansıdır. Detaylı anlatım için [Bölüm 11 - MCP](../11-mcp/README.md) bölümüne bakın.

> **Not:** Bu bölüm 00 - Hızlı Referans serisinin son konusudur.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| MCP nedir, temel kavramlar | [Bölüm 11 - MCP Nedir?](../11-mcp/01-mcp-nedir.md) |
| MCP kurulumu ve konfigürasyonu | [Bölüm 11 - Kurulum](../11-mcp/02-mcp-kurulumu-ve-konfigurasyonu.md) |
| Hazır MCP sunucuları | [Bölüm 11 - Hazır Sunucular](../11-mcp/03-hazir-mcp-sunuculari.md) |
| Tool Search mekanizması | [Bölüm 11 - Tool Search](../11-mcp/04-mcp-tool-search.md) |

---

## Hazır MCP Sunucuları

MCP ekosistemi yüzlerce hazır sunucu barındırır. Aşağıda en çok kullanılan sunucular kategorilere ayrılmıştır.

```mermaid
flowchart TD
    MCP["MCP Sunucu Kategorileri"]

    MCP --> BROWSER["Tarayıcı Otomasyonu"]
    MCP --> DB["Veritabanı"]
    MCP --> FS["Dosya Sistemi"]
    MCP --> SEARCH["Arama ve Bilgi"]
    MCP --> DEV["Geliştirme Araçları"]
    MCP --> API["Veri ve API"]

    BROWSER --> B1["Puppeteer"]
    BROWSER --> B2["Playwright"]

    DB --> D1["PostgreSQL"]
    DB --> D2["SQLite"]
    DB --> D3["MongoDB"]

    FS --> F1["Filesystem"]

    SEARCH --> S1["Brave Search"]
    SEARCH --> S2["Memory"]

    DEV --> G1["GitHub"]
    DEV --> G2["Linear"]
    DEV --> G3["Slack"]

    API --> A1["Fetch"]
    API --> A2["Google Maps"]
```

---

### Tarayıcı Otomasyonu

Web sayfalarını kontrol etme, ekran görüntüsü alma ve form doldurma işlemleri için kullanılır.

| Sunucu | Açıklama |
|--------|----------|
| **Puppeteer MCP** | Web sayfalarını kontrol etme, screenshot alma, form doldurma |
| **Playwright MCP** | Çoklu tarayıcı desteği (Chromium, Firefox, WebKit) |

```bash
# Puppeteer kurulumu
claude mcp add puppeteer -- npx -y @anthropic-ai/mcp-puppeteer

# Playwright kurulumu
claude mcp add playwright -- npx -y @anthropic-ai/mcp-playwright
```

### Veritabanı

Veritabanı sorgulama, şema inceleme ve veri yönetimi için kullanılır.

| Sunucu | Açıklama |
|--------|----------|
| **PostgreSQL MCP** | SQL sorgusu çalıştırma, şema inceleme |
| **SQLite MCP** | Yerel veritabanı işlemleri |
| **MongoDB MCP** | NoSQL veritabanı işlemleri |

```bash
# PostgreSQL kurulumu
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres \
  "postgresql://user:pass@localhost:5432/mydb"

# SQLite kurulumu
claude mcp add sqlite -- npx -y @modelcontextprotocol/server-sqlite \
  --db-path /path/to/database.db

# MongoDB kurulumu
claude mcp add mongodb -- npx -y @modelcontextprotocol/server-mongodb \
  "mongodb://localhost:27017/mydb"
```

### Dosya Sistemi

Proje dışındaki dizinlere kontrollü dosya erişimi sağlar.

| Sunucu | Açıklama |
|--------|----------|
| **Filesystem MCP** | Güvenli dosya erişimi, okuma, yazma, listeleme |

```bash
# Filesystem kurulumu (erişim verilecek dizinleri belirtin)
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem \
  /path/to/allowed/directory
```

### Arama ve Bilgi

Güncel web aramasına ve kalıcı bellek yönetimine erişim sağlar.

| Sunucu | Açıklama |
|--------|----------|
| **Brave Search MCP** | Güncel web araması |
| **Memory MCP** | Kalıcı bellek, bilgi grafiği |

```bash
# Brave Search kurulumu
claude mcp add brave-search -e BRAVE_API_KEY=BSA_xxx -- \
  npx -y @modelcontextprotocol/server-brave-search

# Memory kurulumu
claude mcp add memory -- npx -y @modelcontextprotocol/server-memory
```

### Geliştirme Araçları

Repo yönetimi, proje takibi ve takım iletişimi için kullanılır.

| Sunucu | Açıklama |
|--------|----------|
| **GitHub MCP** | Repo, issue, PR yönetimi |
| **Linear MCP** | Proje ve sprint yönetimi |
| **Slack MCP** | Kanal okuma, mesaj gönderme, arama |

```bash
# GitHub kurulumu
claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx -- \
  npx -y @modelcontextprotocol/server-github

# Linear kurulumu
claude mcp add linear -e LINEAR_API_KEY=lin_api_xxx -- \
  npx -y @modelcontextprotocol/server-linear

# Slack kurulumu
claude mcp add slack -e SLACK_BOT_TOKEN=xoxb-xxx -e SLACK_TEAM_ID=T01234567 -- \
  npx -y @modelcontextprotocol/server-slack
```

### Veri ve API

HTTP istekleri ve konum tabanlı hizmetler için kullanılır.

| Sunucu | Açıklama |
|--------|----------|
| **Fetch MCP** | HTTP istekleri gönderme |
| **Google Maps MCP** | Konum bilgisi, yol tarifi |

```bash
# Fetch kurulumu
claude mcp add fetch -- npx -y @modelcontextprotocol/server-fetch

# Google Maps kurulumu
claude mcp add google-maps -e GOOGLE_MAPS_API_KEY=xxx -- \
  npx -y @modelcontextprotocol/server-google-maps
```

---

## MCP Kurulum Formatı

Tüm MCP sunucuları aynı genel formatla kurulur:

```bash
# Genel format
claude mcp add <isim> -- <komut> [argümanlar]

# Ortam değişkenleri ile
claude mcp add <isim> -e API_KEY=xxx -- <komut>

# Birden fazla ortam değişkeni ile
claude mcp add <isim> -e KEY1=val1 -e KEY2=val2 -- <komut>

# Scope belirtme
claude mcp add --scope user <isim> -- <komut>     # kullanıcı kapsamlı (global)
claude mcp add --scope project <isim> -- <komut>   # proje bazlı
```

### Kapsam Seçimi

| Kapsam | Dosya | Kullanım |
|--------|-------|----------|
| **user** (varsayılan) | `~/.claude/.mcp.json` | Kişisel, tüm projelerde geçerli |
| **project** | `<proje-kökü>/.mcp.json` | Projeye özel, takımla paylaşılabilir |

---

## MCP Tool Search

Bu bölüm, çok sayıda MCP sunucusuyla çalışan kullanıcılar için kritik öneme sahiptir.

### Problem

MCP sunucu sayısı arttıkça tüm araç tanımlarını başlangıçta yüklemek (eager loading) ciddi token israfına yol açar. Örneğin 7 sunucu ile 67 araç, başlangıçta yaklaşık 13.400 token harcar.

### Çözüm: Deferred Loading

Tool Search etkinleştirildiğinde araçlar "deferred" (ertelenmiş) olarak yüklenir. Sadece ihtiyaç duyulduğunda tam şema çekilir.

```mermaid
sequenceDiagram
    participant K as Kullanıcı
    participant CC as Claude Code
    participant TS as ToolSearch
    participant MCP as MCP Sunucusu

    K->>CC: "GitHub issue'ları listele"
    CC->>TS: search("github issue list")
    TS-->>CC: list_issues, get_issue (tanımlarla)
    CC->>MCP: list_issues({state: "open"})
    MCP-->>CC: Issue listesi
    CC->>K: Sonuç
```

### Neden Önemli

| Metrik | Eager Loading | Tool Search (Deferred) |
|--------|--------------|----------------------|
| Başlangıç token maliyeti | ~13.400 token | ~200 token |
| Kullanılan araç token'ı | ~600 token | ~600 token |
| İsraf | ~12.800 token | ~0 token |
| Tasarruf | -- | %94 |
| İlk yanıt hızı | Yavaş | Hızlı |

### Etkinleştirme

```bash
# CLI ile etkinleştirme
claude config set mcpToolSearch true
```

```jsonc
// Veya settings.json ile
// ~/.claude/settings.json
{
  "mcpToolSearch": true
}
```

### Ne Zaman Etkinleştirilmeli

| Durum | Tavsiye |
|-------|---------|
| 1-2 MCP sunucusu, toplam 20'den az araç | Eager loading yeterli |
| 3-5 MCP sunucusu, toplam 20-50 araç | İsteğe bağlı |
| 5 ve üzeri MCP sunucusu, toplam 50 ve üzeri araç | Tool Search şiddetle önerilir |
| 100 ve üzeri araç | Tool Search zorunlu |

### Çalışma Mantığı

Claude Code, Tool Search etkinleştirildiğinde `ToolSearch` adında bir dahili araç ekler. Bu araç, tüm MCP araçlarının düşük maliyetli bir indeksinde arama yapar ve yalnızca eşleşen araçların tam tanımlarını bağlama yükler.

```bash
# Claude Code arka planda şu çağrıları yapar:

# Genel arama
ToolSearch(query: "send message slack")
# Sonuç: post_message, reply_to_thread

# Sunucu filtreli arama
ToolSearch(query: "create", server: "github")
# Sonuç: create_issue, create_pull_request, create_branch

# Açıklama bazlı arama
ToolSearch(query: "database table schema")
# Sonuç: list_tables, describe_table, query
```

---

## Pratik Senaryo Örnekleri

### Senaryo 1: Web Sitesinin Screenshot'ını Al

```bash
# Kullanılan MCP: Puppeteer
> localhost:3000'e git ve ana sayfanın ekran görüntüsünü al
# Claude Code -> ToolSearch("screenshot navigate") -> Puppeteer MCP
# -> navigate(url: "http://localhost:3000")
# -> screenshot()
```

### Senaryo 2: Veritabanındaki Kullanıcıları Listele

```bash
# Kullanılan MCP: PostgreSQL
> Veritabanındaki users tablosundaki kayıtları listele
# Claude Code -> ToolSearch("database query users") -> PostgreSQL MCP
# -> query("SELECT * FROM users LIMIT 50")
```

### Senaryo 3: GitHub Issue Oluştur

```bash
# Kullanılan MCP: GitHub
> "Login sayfasında timeout hatası" başlığıyla bug issue'su oluştur
# Claude Code -> ToolSearch("github create issue") -> GitHub MCP
# -> create_issue({title: "Login sayfasında timeout hatası", labels: ["bug"]})
```

### Senaryo 4: Slack'te Mesaj Gönder

```bash
# Kullanılan MCP: Slack
> #dev-updates kanalına "Deployment tamamlandı" mesajı gönder
# Claude Code -> ToolSearch("slack send message") -> Slack MCP
# -> post_message({channel: "#dev-updates", text: "Deployment tamamlandı"})
```

---

## Özet Tablosu: En Popüler 10 MCP Sunucusu

| Sunucu | Kategori | Ne İşe Yarar | Kurulum Komutu |
|--------|----------|-------------|----------------|
| **Puppeteer** | Tarayıcı | Web sayfası kontrolü, screenshot | `claude mcp add puppeteer -- npx -y @anthropic-ai/mcp-puppeteer` |
| **Playwright** | Tarayıcı | Çoklu tarayıcı otomasyonu | `claude mcp add playwright -- npx -y @anthropic-ai/mcp-playwright` |
| **PostgreSQL** | Veritabanı | SQL sorgu, şema keşfi | `claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres <url>` |
| **SQLite** | Veritabanı | Yerel DB işlemleri | `claude mcp add sqlite -- npx -y @modelcontextprotocol/server-sqlite --db-path <yol>` |
| **MongoDB** | Veritabanı | NoSQL işlemleri | `claude mcp add mongodb -- npx -y @modelcontextprotocol/server-mongodb <url>` |
| **Filesystem** | Dosya | Güvenli dosya erişimi | `claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem <dizin>` |
| **Brave Search** | Arama | Güncel web araması | `claude mcp add brave-search -e BRAVE_API_KEY=xxx -- npx -y @modelcontextprotocol/server-brave-search` |
| **GitHub** | Geliştirme | Repo, issue, PR yönetimi | `claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=xxx -- npx -y @modelcontextprotocol/server-github` |
| **Slack** | Geliştirme | Mesajlaşma, kanal yönetimi | `claude mcp add slack -e SLACK_BOT_TOKEN=xxx -- npx -y @modelcontextprotocol/server-slack` |
| **Fetch** | API | HTTP istekleri | `claude mcp add fetch -- npx -y @modelcontextprotocol/server-fetch` |

---

## İlgili Bölümler

- [Bölüm 11 - MCP Nedir?](../11-mcp/01-mcp-nedir.md)
- [Bölüm 11 - Kurulum ve Konfigürasyon](../11-mcp/02-mcp-kurulumu-ve-konfigurasyonu.md)
- [Bölüm 11 - Hazır MCP Sunucuları](../11-mcp/03-hazir-mcp-sunuculari.md)
- [Bölüm 11 - MCP Tool Search](../11-mcp/04-mcp-tool-search.md)
