# MCP Tool Search

Birden fazla MCP sunucusu yapılandırdığınızda araç sayısı hızla artabilir. GitHub (12 araç) + Slack (7 araç) + PostgreSQL (3 araç) + Puppeteer (7 araç) = 29 araç. Bu sayı 100'ü aştığında tüm araç tanımlarını **context window** (bağlam penceresi) içine yüklemek hem token israfına hem de performans düşüşüne yol açar. **Tool Search** (Araç Arama) mekanizması bu sorunu çözer.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| MCP sunucu konfigürasyonu | [Kurulum ve Konfigürasyon](./02-mcp-kurulumu-ve-konfigurasyonu.md) |
| Context window yönetimi | [Bölüm 09](../09-bellek-ve-baglam/05-context-window-yonetimi.md) |
| Claude Code araçları | [Bölüm 08](../08-araclar/README.md) |

---

## Problem: Eager Loading (İstekli Yükleme)

Varsayılan davranışta Claude Code, oturum başladığında tüm MCP sunucularının tüm araç tanımlarını context window'a yükler:

```mermaid
flowchart TD
    subgraph eager ["Eager Loading (Varsayılan)"]
        START["Oturum Başlıyor"] --> LOAD_ALL["TÜM araç tanımları<br/>context window'a yükleniyor"]
        LOAD_ALL --> CTX["Context Window"]

        CTX --> T1["GitHub: 12 araç tanımı<br/>~2400 token"]
        CTX --> T2["Slack: 7 araç tanımı<br/>~1400 token"]
        CTX --> T3["PostgreSQL: 3 araç tanımı<br/>~600 token"]
        CTX --> T4["Puppeteer: 7 araç tanımı<br/>~1400 token"]
        CTX --> T5["Jira: 15 araç tanımı<br/>~3000 token"]
        CTX --> T6["Linear: 8 araç tanımı<br/>~1600 token"]
        CTX --> T7["Notion: 10 araç tanımı<br/>~2000 token"]
        CTX --> T8["MongoDB: 5 araç tanımı<br/>~1000 token"]

        TOTAL["Toplam: ~13.400 token<br/>sadece araç tanımları için! 😰"]
    end

    style eager fill:#f8d7da,stroke:#dc3545
    style TOTAL fill:#dc3545,color:#fff
```

**Sorun:** Kullanıcı sadece "GitHub'daki issue'ları listele" dese bile, Puppeteer, Jira, MongoDB ve diğer tüm sunucuların araç tanımları gereksiz yere bağlamda yer kaplar.

---

## Çözüm: Deferred Loading (Ertelenmiş Yükleme)

Tool Search etkinleştirildiğinde, araç tanımları bağlama yüklenmez. Bunun yerine aranabilir bir **index** (dizin) oluşturulur ve araçlar yalnızca ihtiyaç duyulduğunda yüklenir:

```mermaid
flowchart TD
    subgraph deferred ["Deferred Loading (Tool Search ile)"]
        START["Oturum Başlıyor"] --> INDEX["Araç tanımları<br/>indeksleniyor<br/>(bağlama yüklenmez)"]
        INDEX --> SMALL_CTX["Context Window"]

        SMALL_CTX --> TS["ToolSearch aracı<br/>~200 token"]

        USER["Kullanıcı: 'GitHub issue listele'"] --> SEARCH["ToolSearch:<br/>'github issue' araması"]
        SEARCH --> FOUND["Bulunan: list_issues,<br/>get_issue, create_issue"]
        FOUND --> LOAD["Sadece bu 3 araç<br/>bağlama yükleniyor<br/>~600 token"]

        TOTAL2["Toplam: ~800 token<br/>%94 tasarruf! 🚀"]
    end

    style deferred fill:#d4edda,stroke:#28a745
    style TOTAL2 fill:#28a745,color:#fff
```

---

## Eager vs Deferred Karşılaştırması

```mermaid
flowchart LR
    subgraph eager ["Eager Loading"]
        direction TB
        E_START["Oturum başlangıcı"]
        E_LOAD["67 araç tanımı yüklendi<br/>~13.400 token"]
        E_USE["Kullanıcı 3 araç kullandı<br/>~600 token faydalı"]
        E_WASTE["64 araç = ~12.800 token<br/>boşa gitti 😰"]

        E_START --> E_LOAD --> E_USE --> E_WASTE
    end

    subgraph deferred ["Deferred Loading"]
        direction TB
        D_START["Oturum başlangıcı"]
        D_INDEX["67 araç indekslendi<br/>~200 token (ToolSearch)"]
        D_SEARCH["Arama: 3 araç bulundu<br/>~600 token yüklendi"]
        D_TOTAL["Toplam: ~800 token<br/>%94 tasarruf 🚀"]

        D_START --> D_INDEX --> D_SEARCH --> D_TOTAL
    end

    style eager fill:#f8d7da,stroke:#dc3545
    style deferred fill:#d4edda,stroke:#28a745
```

| Metrik | Eager Loading | Deferred Loading |
|--------|--------------|-----------------|
| **Başlangıç token maliyeti** | ~13.400 token | ~200 token |
| **Kullanılan araç token'ı** | ~600 token | ~600 token |
| **İsraf** | ~12.800 token | ~0 token |
| **Tasarruf** | — | **%94** |
| **İlk yanıt hızı** | Yavaş | Hızlı |
| **Araç keşfi** | Otomatik | Bir adım ek arama |

---

## Tool Search Nasıl Etkinleştirilir?

Tool Search, Claude Code ayarlarından etkinleştirilir:

```jsonc
// ~/.claude/settings.json
{
  "mcpToolSearch": true
}
```

Veya CLI üzerinden:

```bash
claude config set mcpToolSearch true
```

### Ne Zaman Etkinleştirmeli?

| Durum | Tavsiye |
|-------|---------|
| 1-2 MCP sunucusu, toplam <20 araç | Eager loading yeterli |
| 3-5 MCP sunucusu, toplam 20-50 araç | İsteğe bağlı |
| 5+ MCP sunucusu, toplam 50+ araç | **Tool Search şiddetle önerilir** |
| 100+ araç | **Tool Search zorunlu** |

---

## ToolSearch Aracı Nasıl Çalışır?

Tool Search etkinleştirildiğinde, Claude Code'a `ToolSearch` adında yeni bir dahili araç eklenir:

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant CC as Claude Code
    participant TS as ToolSearch
    participant MCP as MCP Sunucusu

    U->>CC: "GitHub'daki açık issue'ları listele"
    CC->>TS: search("github issue list")
    TS-->>CC: Sonuç: list_issues, get_issue,<br/>create_issue (tanımlarıyla)
    CC->>CC: list_issues aracını<br/>bağlama yükle
    CC->>MCP: list_issues({state: "open"})
    MCP-->>CC: Issue listesi
    CC->>U: "Açık issue'lar: ..."
```

### ToolSearch Aracının Parametreleri

```json
{
  "name": "ToolSearch",
  "description": "Search for MCP tools by keyword or description",
  "parameters": {
    "query": {
      "type": "string",
      "description": "Search query to find relevant tools"
    },
    "server": {
      "type": "string",
      "description": "Optional: filter by server name"
    }
  }
}
```

### Örnek ToolSearch Çağrıları

```bash
# Claude Code arka planda şu çağrıları yapar:

# 1. Genel arama
ToolSearch(query: "send message slack")
# → post_message, reply_to_thread

# 2. Sunucu filtreli arama
ToolSearch(query: "create", server: "github")
# → create_issue, create_pull_request, create_branch

# 3. Açıklama bazlı arama
ToolSearch(query: "database table schema")
# → list_tables, describe_table, query
```

---

## Pratik Örnekler

### Örnek 1: Çok Sunuculu Proje — Token Tasarrufu

```jsonc
// .mcp.json — 7 sunucu, 67 araç
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": { "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}", "SLACK_TEAM_ID": "${SLACK_TEAM_ID}" }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "${DATABASE_URL}"]
    },
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": { "BRAVE_API_KEY": "${BRAVE_API_KEY}" }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/yasin/docs"]
    },
    "linear": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-linear"],
      "env": { "LINEAR_API_KEY": "${LINEAR_API_KEY}" }
    }
  }
}
```

```bash
# Tool Search etkinleştirildiğinde:
claude config set mcpToolSearch true

$ claude
# Oturum başlangıcı:
#   Eager Loading olsaydı: ~13.400 token araç tanımı
#   Tool Search ile: ~200 token (sadece ToolSearch aracı)
#   Tasarruf: ~13.200 token = %98!

> Linear'daki aktif sprint görevlerimi göster
# ToolSearch("linear sprint tasks") → list_issues, get_cycle
# Sadece 2 araç yüklendi → ~400 token
```

### Örnek 2: Tool Search Olmadan ve İle Karşılaştırma

```bash
# --- Tool Search KAPALI ---
$ claude
# [13.400 token araç tanımı yüklendi]
# Context window: %12 dolu (sadece araçlardan!)

> Bu repodaki açık PR'ları listele
# Zaten yüklü olan list_pull_requests kullanılır
# Hızlı ama pahalı başlangıç

# --- Tool Search AÇIK ---
$ claude
# [200 token ToolSearch aracı yüklendi]
# Context window: %0.2 dolu

> Bu repodaki açık PR'ları listele
# ToolSearch("list pull requests github") → list_pull_requests
# [600 token yüklendi]
# 1 ekstra adım ama çok daha verimli
```

### Örnek 3: ToolSearch Tarafından Bulunamayan Araçlar

```bash
# Bazen ToolSearch doğru aracı bulamayabilir
> Veritabanındaki tabloları listele

# Eğer ToolSearch yanlış sonuç verirse:
# Claude Code otomatik olarak sorguyu yeniden düzenler
# ToolSearch("database tables") → list_tables ✅

# Manuel müdahale gerekirse:
> /mcp
# Tüm sunucuları ve araçları görüntüleyin
# İhtiyaç duyduğunuz aracı belirleyin
# Doğrudan araç adını kullanarak isteyin:
> postgres sunucusundaki list_tables aracını kullan
```

---

## Mimari Diyagram

```mermaid
flowchart TD
    subgraph claudecode ["Claude Code"]
        CC["Ana Motor"]
        TS["ToolSearch<br/>Aracı"]
        IDX["Araç İndeksi<br/>(Düşük token maliyeti)"]
        CTX["Context Window"]
    end

    subgraph mcpservers ["MCP Sunucuları"]
        S1["GitHub<br/>12 araç"]
        S2["Slack<br/>7 araç"]
        S3["PostgreSQL<br/>3 araç"]
        S4["...diğer<br/>N araç"]
    end

    S1 -->|"Araç listesi"| IDX
    S2 -->|"Araç listesi"| IDX
    S3 -->|"Araç listesi"| IDX
    S4 -->|"Araç listesi"| IDX

    CC -->|"1. Arama"| TS
    TS -->|"2. İndekste ara"| IDX
    IDX -->|"3. Eşleşen araçlar"| TS
    TS -->|"4. Tanımları yükle"| CTX
    CC -->|"5. Aracı çağır"| S1

    style TS fill:#4A90D9,color:#fff
    style IDX fill:#27AE60,color:#fff
    style CTX fill:#F39C12,color:#fff
```

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Eager Loading** | Tüm araç tanımlarını baştan yükleme (varsayılan) |
| **Deferred Loading** | Araçları ihtiyaç anında yükleme (Tool Search ile) |
| **ToolSearch** | Araç indeksinde arama yapan dahili araç |
| **Araç İndeksi** | Tüm araçların düşük maliyetli aranabilir listesi |
| **`mcpToolSearch`** | Tool Search'ü etkinleştiren ayar |
| **Token tasarrufu** | Çok sunuculu ortamlarda %90+ tasarruf mümkün |

---

## Sonraki Adım

Tool Search'ün ne zaman kullanılacağını öğrendik. Şimdi MCP sunucuları ile Skill'ler arasındaki farkı ve hangisini ne zaman tercih etmeniz gerektiğini inceleyelim:

→ [MCP vs Skills: Ne Zaman Hangisi?](./05-mcp-vs-skills-ne-zaman-hangisi.md)
