# MCP Nedir?

**Model Context Protocol (MCP)**, Anthropic tarafından Kasım 2024'te tanıtılan açık bir standarttır. Yapay zeka modellerini harici veri kaynakları ve araçlarla bağlamak için tasarlanmıştır. Sıklıkla **"AI'ın USB-C'si"** olarak tanımlanan MCP, farklı bağlantı noktaları yerine tek bir evrensel standart sunar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code araçları (Tools) | [Bölüm 08](../08-araclar/README.md) |
| Claude Code nasıl çalışır | [Bölüm 06](../06-claude-code-tanitim/02-claude-code-nasil-calisir.md) |
| JSON-RPC temel bilgisi | Harici kaynak |

---

## USB-C Benzetmesi

Telefonlar, dizüstü bilgisayarlar ve tabletler bir dönemde farklı şarj ve veri kablolarına ihtiyaç duyuyordu. USB-C bu sorunu tek bir standart bağlantı noktasıyla çözdü. MCP de aynı şeyi yapay zeka dünyasında yapıyor:

```mermaid
flowchart LR
    subgraph eski ["USB-C Öncesi (Eski Dünya)"]
        direction TB
        E1["Micro-USB<br/>(Android)"]
        E2["Lightning<br/>(iPhone)"]
        E3["USB-A<br/>(PC)"]
        E4["Barrel Jack<br/>(Laptop)"]
    end

    subgraph yeni ["USB-C Sonrası"]
        direction TB
        Y1["USB-C<br/>(Hepsi için tek kablo)"]
    end

    subgraph ai_eski ["MCP Öncesi"]
        direction TB
        A1["OpenAI Function Calling"]
        A2["LangChain Tools"]
        A3["Özel REST Wrapper"]
        A4["Proprietary Plugin API"]
    end

    subgraph ai_yeni ["MCP Sonrası"]
        direction TB
        B1["MCP<br/>(Hepsi için tek protokol)"]
    end

    eski -.->|"Benzetme"| ai_eski
    yeni -.->|"Benzetme"| ai_yeni
```

---

## MCP Öncesi Dünya: Sorun Ne?

MCP'den önce her LLM sağlayıcısı ve her veri kaynağı için ayrı ayrı entegrasyon geliştirmek gerekiyordu:

```mermaid
flowchart TD
    subgraph modeller ["AI Modelleri"]
        M1["Claude"]
        M2["GPT"]
        M3["Gemini"]
    end

    subgraph kaynaklar ["Veri Kaynakları"]
        K1["GitHub"]
        K2["Slack"]
        K3["PostgreSQL"]
        K4["Google Drive"]
    end

    M1 -->|"Özel entegrasyon 1"| K1
    M1 -->|"Özel entegrasyon 2"| K2
    M1 -->|"Özel entegrasyon 3"| K3
    M1 -->|"Özel entegrasyon 4"| K4
    M2 -->|"Özel entegrasyon 5"| K1
    M2 -->|"Özel entegrasyon 6"| K2
    M2 -->|"Özel entegrasyon 7"| K3
    M2 -->|"Özel entegrasyon 8"| K4
    M3 -->|"Özel entegrasyon 9"| K1
    M3 -->|"Özel entegrasyon 10"| K2
    M3 -->|"Özel entegrasyon 11"| K3
    M3 -->|"Özel entegrasyon 12"| K4

    style modeller fill:#4A90D9,color:#fff
    style kaynaklar fill:#E74C3C,color:#fff
```

**Sonuç:** 3 model × 4 kaynak = **12 ayrı entegrasyon** 😰

### MCP Sonrası Dünya

```mermaid
flowchart TD
    subgraph modeller ["AI Modelleri (MCP Client)"]
        M1["Claude"]
        M2["GPT"]
        M3["Gemini"]
    end

    MCP_LAYER["MCP Protokolü<br/>(Evrensel Standart)"]

    subgraph sunucular ["MCP Sunucuları"]
        S1["GitHub<br/>MCP Server"]
        S2["Slack<br/>MCP Server"]
        S3["PostgreSQL<br/>MCP Server"]
        S4["Google Drive<br/>MCP Server"]
    end

    M1 --> MCP_LAYER
    M2 --> MCP_LAYER
    M3 --> MCP_LAYER

    MCP_LAYER --> S1
    MCP_LAYER --> S2
    MCP_LAYER --> S3
    MCP_LAYER --> S4

    style MCP_LAYER fill:#27AE60,color:#fff,stroke-width:3px
```

**Sonuç:** 3 model + 4 sunucu = **7 entegrasyon** 🚀

---

## Client-Server Mimarisi

MCP, klasik bir **client-server** (istemci-sunucu) mimarisi kullanır. Bu mimari üç ana bileşenden oluşur:

```mermaid
flowchart LR
    subgraph host ["Host (Ana Uygulama)"]
        H["Claude Code<br/>(Terminal)"]
    end

    subgraph clients ["MCP Client'ları"]
        C1["Client 1"]
        C2["Client 2"]
        C3["Client 3"]
    end

    subgraph servers ["MCP Server'ları"]
        S1["GitHub<br/>Server"]
        S2["Slack<br/>Server"]
        S3["PostgreSQL<br/>Server"]
    end

    H --> C1
    H --> C2
    H --> C3

    C1 <-->|"JSON-RPC 2.0<br/>(stdio)"| S1
    C2 <-->|"JSON-RPC 2.0<br/>(stdio)"| S2
    C3 <-->|"JSON-RPC 2.0<br/>(SSE)"| S3

    style host fill:#4A90D9,color:#fff
    style clients fill:#F39C12,color:#fff
    style servers fill:#27AE60,color:#fff
```

### Bileşenler

| Bileşen | Rol | Örnek |
|---------|-----|-------|
| **Host** | MCP client'larını barındıran ana uygulama | Claude Code, Claude Desktop, Cursor |
| **MCP Client** | Sunucuyla 1:1 bağlantı kuran protokol istemcisi | Her sunucu tanımı için otomatik oluşturulur |
| **MCP Server** | Belirli bir yeteneği açığa çıkaran hafif program | `@modelcontextprotocol/server-github` |

### İletişim Protokolü: JSON-RPC 2.0

MCP, **JSON-RPC 2.0** standardını kullanır. Bu, istemci ve sunucu arasındaki tüm mesajların JSON formatında olduğu anlamına gelir:

```json
// İstek (Client → Server)
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_issue",
    "arguments": {
      "owner": "anthropics",
      "repo": "claude-code",
      "issue_number": 42
    }
  }
}

// Yanıt (Server → Client)
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Issue #42: Fix memory leak in context window..."
      }
    ]
  }
}
```

### Transport (Taşıma) Yöntemleri

MCP iki ana taşıma yöntemi destekler:

| Yöntem | Açıklama | Kullanım |
|--------|----------|----------|
| **stdio** | Standart giriş/çıkış üzerinden iletişim | Yerel sunucular (varsayılan) |
| **SSE** | Server-Sent Events üzerinden HTTP iletişimi | Uzak sunucular |

```mermaid
flowchart TD
    subgraph stdio_transport ["stdio Transport"]
        direction LR
        CC1["Claude Code"] -->|"stdin"| SP["MCP Server<br/>(Yerel süreç)"]
        SP -->|"stdout"| CC1
    end

    subgraph sse_transport ["SSE Transport"]
        direction LR
        CC2["Claude Code"] -->|"HTTP POST"| RS["MCP Server<br/>(Uzak URL)"]
        RS -->|"SSE stream"| CC2
    end

    style stdio_transport fill:#d4edda,stroke:#28a745
    style sse_transport fill:#fff3cd,stroke:#ffc107
```

---

## MCP Sunucusunun Sunabileceği Yetenekler

Bir MCP sunucusu üç tür yetenek sunabilir:

```mermaid
flowchart LR
    MCP["MCP Server"] --> T["Tools<br/>(Araçlar)"]
    MCP --> R["Resources<br/>(Kaynaklar)"]
    MCP --> P["Prompts<br/>(Şablonlar)"]

    T --> T1["Fonksiyonlar<br/>Model tarafından çağrılır"]
    R --> R1["Veri kaynakları<br/>Dosya benzeri veriler"]
    P --> P1["Hazır şablonlar<br/>Kullanıcı tetikler"]

    style T fill:#4A90D9,color:#fff
    style R fill:#27AE60,color:#fff
    style P fill:#F39C12,color:#fff
```

| Yetenek | Açıklama | Örnek |
|---------|----------|-------|
| **Tools** (Araçlar) | Model tarafından çağrılabilen fonksiyonlar | `create_issue`, `run_query`, `send_message` |
| **Resources** (Kaynaklar) | Modelin okuyabileceği veri kaynakları | Dosya içerikleri, veritabanı şemaları |
| **Prompts** (Şablonlar) | Önceden tanımlanmış prompt şablonları | Kod inceleme şablonu, SQL oluşturma şablonu |

> **Not:** Claude Code şu anda öncelikli olarak MCP **Tools** yeteneğini kullanır. Resources ve Prompts desteği geliştirilmeye devam etmektedir.

---

## Endüstri Genelinde Benimseme

MCP, sadece Anthropic'in değil, tüm AI endüstrisinin benimsediği bir standart haline gelmiştir:

```mermaid
timeline
    title MCP Benimseme Zaman Çizelgesi
    Kasım 2024 : Anthropic MCP'yi açık standart olarak yayınladı
    Aralık 2024 : İlk topluluk MCP sunucuları yayınlandı
    Ocak 2025 : 100+ topluluk sunucusu
    Mart 2025 : OpenAI, MCP desteğini duyurdu
    Nisan 2025 : Google DeepMind, MCP'yi benimsedi
    2025 Q3-Q4 : 1000+ topluluk sunucusu
    2026 Q1 : MCP endüstri standardı olarak yerleşti
```

| Şirket | MCP Desteği |
|--------|-------------|
| **Anthropic** | Standart yaratıcısı — Claude Code, Claude Desktop |
| **OpenAI** | ChatGPT ve API'de MCP desteği |
| **Google** | Gemini entegrasyonlarında MCP desteği |
| **Microsoft** | Copilot ekosisteminde MCP uyumluluğu |
| **Topluluk** | 1000+ açık kaynak MCP sunucusu |

---

## Pratik Örnekler

### Örnek 1: MCP Olmadan GitHub Entegrasyonu

```python
# MCP öncesi: Her API için özel kod yazmanız gerekiyordu
import requests

def get_github_issue(owner, repo, issue_number):
    url = f"https://api.github.com/repos/{owner}/{repo}/issues/{issue_number}"
    headers = {"Authorization": f"token {GITHUB_TOKEN}"}
    response = requests.get(url, headers=headers)
    return response.json()

def create_github_issue(owner, repo, title, body):
    url = f"https://api.github.com/repos/{owner}/{repo}/issues"
    headers = {"Authorization": f"token {GITHUB_TOKEN}"}
    data = {"title": title, "body": body}
    response = requests.post(url, headers=headers, json=data)
    return response.json()

# Bu kodu her AI aracı için ayrı ayrı yazmanız gerekiyordu
# Claude Code için ayrı, Cursor için ayrı, Copilot için ayrı...
```

### Örnek 2: MCP ile GitHub Entegrasyonu

```jsonc
// .mcp.json — tek bir konfigürasyon dosyası yeterli
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    }
  }
}
```

```bash
# Artık doğal dille konuşarak GitHub işlemleri yapabilirsiniz:
> anthropics/claude-code reposundaki #42 numaralı issue'yu göster
# Claude Code → MCP GitHub Server → get_issue çağrısı → sonuç

> Bu bug için yeni bir issue oluştur, başlık: "Memory leak in context window"
# Claude Code → MCP GitHub Server → create_issue çağrısı → sonuç

> Son 5 PR'ı listele ve hangilerinin CI'dan geçtiğini göster
# Claude Code → MCP GitHub Server → list_pull_requests + get_checks → sonuç
```

### Örnek 3: MCP'nin Çözdüğü Gerçek Dünya Problemi

```mermaid
flowchart TD
    subgraph problem ["Problem: Bir Bug Düzeltme Akışı (MCP Öncesi)"]
        P1["1. GitHub'da issue oku<br/>(Tarayıcıya geç)"] --> P2["2. Slack'te tartışmayı bul<br/>(Başka bir sekmede)"]
        P2 --> P3["3. Veritabanı şemasına bak<br/>(Terminal aç, psql bağlan)"]
        P3 --> P4["4. Kodu düzenle<br/>(IDE'ye dön)"]
        P4 --> P5["5. PR oluştur<br/>(Tarayıcıya tekrar geç)"]
    end

    subgraph cozum ["Çözüm: Aynı Akış (MCP ile)"]
        C1["Claude Code tek arayüzden<br/>her şeyi yapar"]
        C1 --> C2["GitHub issue → MCP"]
        C1 --> C3["Slack mesaj → MCP"]
        C1 --> C4["PostgreSQL → MCP"]
        C1 --> C5["Kod + PR → Dahili araçlar"]
    end

    problem -.->|"MCP dönüşümü"| cozum
```

---

## Temel Kavramlar Özeti

| Kavram | Açıklama |
|--------|----------|
| **MCP** | Model Context Protocol — AI modellerini harici kaynaklarla bağlayan açık standart |
| **Host** | MCP client'larını barındıran uygulama (Claude Code) |
| **Client** | Sunucuyla 1:1 bağlantı kuran protokol istemcisi |
| **Server** | Belirli yetenekleri açığa çıkaran hafif program |
| **JSON-RPC 2.0** | İstemci-sunucu iletişim formatı |
| **stdio** | Yerel sunucular için standart giriş/çıkış taşıma yöntemi |
| **SSE** | Uzak sunucular için HTTP tabanlı taşıma yöntemi |
| **Tools** | Sunucunun açığa çıkardığı çağrılabilir fonksiyonlar |

---

## Sonraki Adım

MCP'nin ne olduğunu ve neden önemli olduğunu öğrendik. Şimdi Claude Code'da MCP sunucularını nasıl yapılandıracağımızı görelim:

→ [Kurulum ve Konfigürasyon](./02-mcp-kurulumu-ve-konfigurasyonu.md)
