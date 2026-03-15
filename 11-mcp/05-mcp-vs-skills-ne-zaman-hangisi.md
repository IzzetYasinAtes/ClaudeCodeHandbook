# MCP vs Skills: Ne Zaman Hangisi?

Claude Code'un yeteneklerini genişletmenin iki temel yolu vardır: **MCP Server** (Model Context Protocol sunucusu) ve **Skill** (beceri dosyası). Her iki yaklaşımın güçlü yanları ve ideal kullanım alanları farklıdır. Bu bölüm, doğru aracı seçmeniz için bir karar rehberi sunar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| MCP nedir ve nasıl çalışır | [MCP Nedir?](./01-mcp-nedir.md) |
| MCP sunucu konfigürasyonu | [Kurulum ve Konfigürasyon](./02-mcp-kurulumu-ve-konfigurasyonu.md) |
| Skills ve Pluginler | [Bölüm 12](../12-skills-ve-pluginler/README.md) |
| Context window yönetimi | [Bölüm 09](../09-bellek-ve-baglam/05-context-window-yonetimi.md) |

---

## Temel Farklar

```mermaid
flowchart LR
    subgraph mcp_side ["MCP Server"]
        direction TB
        M1["Kalıcı süreç\n(Persistent process)"]
        M2["JSON-RPC iletişimi"]
        M3["Karmaşık API sarmalama"]
        M4["Gerçek zamanlı veri"]
        M5["Yüksek bağlam maliyeti"]
    end

    subgraph skill_side ["Skill"]
        direction TB
        S1["Markdown dosyası\n(Statik doküman)"]
        S2["İstek üzerine okunur"]
        S3["CLI komut talimatları"]
        S4["Düşük bağlam maliyeti"]
        S5["Hafif ve taşınabilir"]
    end

    mcp_side ---|"vs"| skill_side

    style mcp_side fill:#4A90D9,color:#fff
    style skill_side fill:#27AE60,color:#fff
```

| Özellik | MCP Server | Skill |
|---------|-----------|-------|
| **Format** | Çalışan süreç (process) | Markdown dosyası (.md) |
| **Çalışma şekli** | Kalıcı bağlantı, oturum boyunca aktif | İhtiyaç anında okunur, talimat olarak uygulanır |
| **Bağlam maliyeti** | Yüksek (~200-3000 token/sunucu) | Düşük (~100-500 token) |
| **Karmaşıklık** | Yüksek (Node.js/Python süreci) | Düşük (sadece metin dosyası) |
| **Gerçek zamanlı veri** | ✅ Evet | ❌ Hayır (CLI aracılığıyla dolaylı) |
| **API entegrasyonu** | ✅ Doğrudan | ❌ CLI komutları ile dolaylı |
| **Kurulum** | `.mcp.json` + bağımlılıklar | `.md` dosyası oluşturmak yeterli |
| **Bakım** | Sunucu güncellemeleri gerekir | Metin düzenlemesi yeterli |
| **Taşınabilirlik** | Ortama bağlı (Node.js vb.) | Her yerde çalışır (sadece metin) |

---

## Karar Ağacı

Hangi yaklaşımı kullanmanız gerektiğine karar vermek için şu akışı izleyin:

```mermaid
flowchart TD
    START["Harici bir yeteneğe\nihtiyacım var"] --> Q1{"Gerçek zamanlı\nveri gerekli mi?"}

    Q1 -->|"Evet"| Q2{"Karmaşık API\netkileşimi var mı?"}
    Q1 -->|"Hayır"| Q3{"CLI komutuyla\nyapılabilir mi?"}

    Q2 -->|"Evet\n(Çok endpoint, auth, webhook)"| MCP["✅ MCP Server\nKullan"]
    Q2 -->|"Hayır\n(Tek endpoint, basit)"| Q3

    Q3 -->|"Evet"| Q4{"Bağlam maliyeti\nönemli mi?"}
    Q3 -->|"Hayır"| MCP

    Q4 -->|"Evet\n(Token tasarrufu gerekli)"| SKILL["✅ Skill\nKullan"]
    Q4 -->|"Hayır"| Q5{"Takımla paylaşım\ngerekli mi?"}

    Q5 -->|"Evet, standart araç olarak"| MCP
    Q5 -->|"Evet, doküman olarak"| SKILL
    Q5 -->|"Hayır, kişisel"| SKILL

    style MCP fill:#4A90D9,color:#fff
    style SKILL fill:#27AE60,color:#fff
```

---

## Bağlam Maliyeti Karşılaştırması

MCP sunucusu ve Skill'in bağlam (context) maliyetini somut bir örnekle karşılaştıralım:

### Senaryo: LangSmith Entegrasyonu

Bir geliştirici, LangSmith API'sini Claude Code'da kullanmak istiyor. İki yaklaşımı karşılaştıralım:

```mermaid
flowchart TD
    subgraph mcp_approach ["Yaklaşım 1: MCP Server"]
        M_DEF["Sunucu tanımı: ~500 token"]
        M_TOOLS["12 araç tanımı: ~2400 token"]
        M_INIT["Başlatma mesajları: ~300 token"]
        M_TOTAL["TOPLAM: ~3200 token\nher oturumda 😰"]

        M_DEF --> M_TOOLS --> M_INIT --> M_TOTAL
    end

    subgraph skill_approach ["Yaklaşım 2: CLI Skill"]
        S_FILE["Skill dosyası: ~200 token\nsadece ihtiyaç anında"]
        S_TOTAL["TOPLAM: ~200 token\nve sadece kullanıldığında 🚀"]

        S_FILE --> S_TOTAL
    end

    style mcp_approach fill:#f8d7da,stroke:#dc3545
    style skill_approach fill:#d4edda,stroke:#28a745
    style M_TOTAL fill:#dc3545,color:#fff
    style S_TOTAL fill:#28a745,color:#fff
```

### MCP Yaklaşımı: LangSmith MCP Server

```jsonc
// .mcp.json
{
  "mcpServers": {
    "langsmith": {
      "command": "npx",
      "args": ["-y", "langsmith-mcp-server"],
      "env": {
        "LANGSMITH_API_KEY": "${LANGSMITH_API_KEY}"
      }
    }
  }
}
```

**Maliyet:** Oturum başına ~3200 token (araçlar kullanılmasa bile)

### Skill Yaklaşımı: LangSmith CLI Skill

```markdown
# LangSmith CLI Skill

LangSmith trace'lerini sorgulamak için `langsmith` CLI aracını kullan.

## Kurulum Kontrolü
```bash
pip install langsmith
```

## Yaygın Komutlar

### Trace listesi
```bash
langsmith trace list --project "my-project" --limit 10
```

### Trace detayı
```bash
langsmith trace get <trace-id>
```

### Run istatistikleri
```bash
langsmith run stats --project "my-project" --last 24h
```

## Notlar
- API anahtarı: `LANGSMITH_API_KEY` ortam değişkeni gerekli
- Varsayılan proje: `LANGSMITH_PROJECT` ile ayarlanabilir
```

**Maliyet:** Sadece kullanıldığında ~200 token

### Sonuç

| Metrik | MCP Server | CLI Skill |
|--------|-----------|-----------|
| **Bağlam maliyeti** | ~3200 token/oturum | ~200 token (kullanıldığında) |
| **Tasarruf** | — | **%94** |
| **Kurulum karmaşıklığı** | Node.js + npm paketi | Metin dosyası |
| **Gerçek zamanlı veri** | ✅ | CLI üzerinden ✅ |
| **Bakım** | Paket güncellemeleri | Metin düzenlemesi |

> **Gerçek Dünya:** Bir geliştirici, LangSmith MCP sunucusunu 200 token'lık bir CLI Skill ile değiştirerek bağlam kullanımını **%95 azalttı**. Aynı işlevselliği CLI komutları aracılığıyla sağladı.

---

## Ne Zaman MCP Kullanmalı?

MCP sunucusu şu durumlarda daha uygundur:

### 1. Karmaşık API Etkileşimleri

```jsonc
// GitHub — çok sayıda endpoint, karmaşık auth, webhook desteği
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
// 12+ araç, karmaşık nesne yapıları, pagination...
// Bunu CLI Skill ile yapmak çok zor olurdu
```

### 2. Gerçek Zamanlı Veritabanı Erişimi

```jsonc
// PostgreSQL — doğrudan SQL, canlı bağlantı
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:pass@localhost:5432/mydb"
      ]
    }
  }
}
// Anlık sorgu sonuçları, şema keşfi...
// CLI ile mümkün ama MCP çok daha doğal
```

### 3. Durum Bilgisi Gerektiren İşlemler

```bash
# Puppeteer — tarayıcı durumu korunur
> localhost:3000'e git
> Login formunu doldur
> Dashboard'a yönlendiğini doğrula
# Her adım bir öncekinin durumunu kullanır
# Bunu Skill ile yapmak pratik değil
```

---

## Ne Zaman Skill Kullanmalı?

Skill dosyası şu durumlarda daha uygundur:

### 1. CLI Araçları için Talimatlar

```markdown
# Docker Yönetim Skill'i

## Container listesi
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

## Log görüntüleme
```bash
docker logs --tail 100 -f <container-name>
```

## Container yeniden başlatma
```bash
docker compose restart <service-name>
```
```

### 2. API Kullanım Kılavuzları

```markdown
# Stripe CLI Skill

## Ödeme listesi
```bash
stripe payments list --limit 10
```

## Test ödeme oluşturma
```bash
stripe payment_intents create --amount 2000 --currency usd
```

## Webhook dinleme
```bash
stripe listen --forward-to localhost:3000/webhooks/stripe
```
```

### 3. Proje Özel Komut Setleri

```markdown
# Proje Deploy Skill

## Staging ortamına deploy
```bash
./scripts/deploy.sh staging
```

## Production deploy (approval gerekli)
```bash
./scripts/deploy.sh production --require-approval
```

## Rollback
```bash
./scripts/rollback.sh --to-version <version>
```
```

---

## Hibrit Yaklaşım: İkisini Birlikte Kullanma

En iyi strateji, MCP ve Skill'leri birlikte kullanmaktır:

```mermaid
flowchart TD
    subgraph hibrit ["Hibrit Yaklaşım"]
        direction TB

        subgraph mcp_katman ["MCP Sunucuları (Kalıcı Bağlantı)"]
            G["GitHub MCP\nPR, issue yönetimi"]
            P["PostgreSQL MCP\nVeritabanı sorguları"]
        end

        subgraph skill_katman ["Skill Dosyaları (Düşük Maliyet)"]
            S1["Docker Skill\nContainer yönetimi"]
            S2["AWS CLI Skill\nBulut operasyonları"]
            S3["Testing Skill\nTest stratejileri"]
        end
    end

    style mcp_katman fill:#4A90D9,color:#fff
    style skill_katman fill:#27AE60,color:#fff
```

```jsonc
// .mcp.json — Sadece gerçekten MCP gerektiren sunucular
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "${DATABASE_URL}"]
    }
  }
}
```

```bash
# Geri kalan her şey Skill dosyaları olarak:
# .claude/skills/docker.md
# .claude/skills/aws-cli.md
# .claude/skills/testing-strategy.md

# Bu yaklaşımla:
# - MCP bağlam maliyeti: ~4000 token (2 sunucu)
# - Skill maliyeti: ~0 token (kullanılana kadar)
# - Toplam başlangıç maliyeti: ~4000 token
# vs
# - Hepsini MCP yapma: ~12.000+ token
# - Tasarruf: %67
```

---

## Pratik Örnekler

### Örnek 1: Bir Aracı MCP'den Skill'e Dönüştürme

Sentry hata izleme aracını MCP'den Skill'e dönüştürme süreci:

**Önce (MCP):**

```jsonc
// .mcp.json
{
  "mcpServers": {
    "sentry": {
      "command": "npx",
      "args": ["-y", "sentry-mcp-server"],
      "env": {
        "SENTRY_AUTH_TOKEN": "${SENTRY_AUTH_TOKEN}",
        "SENTRY_ORG": "my-org"
      }
    }
  }
}
// Maliyet: ~1800 token/oturum
// 8 araç tanımı sürekli bağlamda
```

**Sonra (Skill):**

```markdown
# Sentry CLI Skill

Sentry hata izleme için `sentry-cli` aracını kullan.

## Kurulum
```bash
npm install -g @sentry/cli
export SENTRY_AUTH_TOKEN="your-token"
export SENTRY_ORG="my-org"
```

## Son hataları listeleme
```bash
sentry-cli issues list --project my-project --status unresolved
```

## Hata detayı
```bash
sentry-cli issues show <issue-id>
```

## Release oluşturma
```bash
sentry-cli releases new <version>
sentry-cli releases set-commits <version> --auto
sentry-cli releases finalize <version>
```

## Source map yükleme
```bash
sentry-cli sourcemaps upload --release <version> ./dist
```
```

```
Maliyet: ~250 token (sadece kullanıldığında)
Tasarruf: %86
```

### Örnek 2: Karar Matrisi — Gerçek Senaryo

Bir e-ticaret projesinde hangi entegrasyonlar için ne kullanılmalı:

| Entegrasyon | Karar | Gerekçe |
|-------------|-------|---------|
| GitHub | **MCP** | Karmaşık API, çok sayıda endpoint, PR review akışları |
| PostgreSQL | **MCP** | Gerçek zamanlı sorgu, şema keşfi, canlı bağlantı |
| Slack | **MCP** | Mesaj gönderme/alma, thread takibi, gerçek zamanlı |
| Docker | **Skill** | CLI komutları yeterli, basit talimatlar |
| AWS S3 | **Skill** | `aws s3` CLI komutu yeterli |
| Sentry | **Skill** | `sentry-cli` yeterli, token tasarrufu |
| Redis | **Skill** | `redis-cli` yeterli, basit komutlar |
| Stripe | **Skill** | `stripe` CLI yeterli, test/üretim ayrımı kolay |
| Puppeteer | **MCP** | Durum bilgisi gerektiren tarayıcı otomasyonu |

### Örnek 3: Token Bütçesi Planlaması

```mermaid
flowchart LR
    subgraph butce ["200K Token Context Window"]
        SYSTEM["System Prompt\n~5K token"]
        TOOLS["Dahili Araçlar\n~3K token"]
        MCP_TOOLS["MCP Araçları\n~4K token\n(2 sunucu)"]
        SKILLS["Skill'ler\n~0K token\n(kullanılana kadar)"]
        CONVERSATION["Konuşma\n~188K token\nkullanılabilir ✅"]
    end

    style CONVERSATION fill:#27AE60,color:#fff
    style MCP_TOOLS fill:#4A90D9,color:#fff
    style SKILLS fill:#F39C12,color:#fff
```

```bash
# Token bütçesi hesaplama:
# Toplam context window: 200K token
#
# Sabit maliyetler:
#   System prompt: ~5K
#   Dahili araçlar: ~3K
#
# MCP maliyeti (optimize):
#   GitHub MCP: ~2.5K
#   PostgreSQL MCP: ~1K
#   Tool Search ile diğerleri: ~0.2K
#   Toplam MCP: ~3.7K
#
# Skill maliyeti:
#   Başlangıçta: 0K
#   Kullanıldığında: ~0.2K per skill
#
# Konuşma için kalan: ~188K token (%94)
#
# Karşılaştırma: Hepsini MCP yapsaydık:
#   7 MCP sunucu: ~13K token
#   Konuşma için kalan: ~179K token (%89.5)
#   Fark: 9K token (%4.5 kayıp)
```

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **MCP Server** | Kalıcı süreç, karmaşık API'ler, gerçek zamanlı veri |
| **Skill** | Markdown dosyası, CLI talimatları, düşük bağlam maliyeti |
| **Karar kriteri** | Gerçek zamanlı + karmaşık → MCP; CLI yeterli → Skill |
| **Hibrit yaklaşım** | MCP ve Skill'leri birlikte kullanma (önerilen) |
| **Token tasarrufu** | Skill'ler MCP'ye göre %85-95 daha az bağlam kullanır |
| **Bakım kolaylığı** | Skill = metin düzenleme; MCP = paket yönetimi |

---

## Sonraki Adım

MCP ve Skill arasındaki farkları, ne zaman hangisini kullanmanız gerektiğini öğrendik. Şimdi Skill sisteminin detaylarını ve kendi Skill'lerinizi nasıl oluşturacağınızı inceleyelim:

→ [Skills ve Pluginler](../12-skills-ve-pluginler/README.md)
