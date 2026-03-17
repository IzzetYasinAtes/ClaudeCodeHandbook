# LLM Gateway

LLM Gateway (büyük dil modeli geçidi), Claude Code ile Anthropic API arasında bir ara katman olarak çalışarak yönlendirme, güvenlik, maliyet kontrolü ve izleme gibi kurumsal ihtiyaçları karşılar. Bu rehber, gateway çözümlerini, endpoint yapılandırmasını ve provider-specific (sağlayıcıya özel) kurulumları kapsar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Ağ ve proxy konfigürasyonu | [Ağ ve Proxy Konfigürasyonu](./04-ag-ve-proxy-konfigurasyonu.md) |
| Ortam değişkenleri | [Ortam Değişkenleri](../17-konfigurasyon/03-ortam-degiskenleri.md) |

---

## LLM Gateway Nedir?

LLM Gateway, tüm LLM API çağrılarını merkezi bir noktadan geçirerek kontrol, izleme ve güvenlik sağlayan bir reverse proxy'dir (ters vekil sunucu).

```mermaid
flowchart LR
    subgraph clients ["İstemciler"]
        CC1["Claude Code<br/>(Geliştirici 1)"]
        CC2["Claude Code<br/>(Geliştirici 2)"]
        CC3["Claude Code<br/>(CI/CD)"]
    end

    subgraph gateway ["LLM Gateway"]
        ROUTE["🔀 Yönlendirme"]
        AUTH["🔐 Kimlik Doğrulama"]
        RATE["⏱️ Rate Limiting"]
        LOG["📊 Loglama"]
        CACHE["💾 Caching"]
    end

    subgraph providers ["LLM Sağlayıcıları"]
        ANTHROPIC["Anthropic API"]
        BEDROCK["AWS Bedrock"]
        VERTEX["Google Vertex AI"]
    end

    clients --> gateway --> providers

```

### Gateway Avantajları

| Avantaj | Açıklama |
|---------|----------|
| Merkezi API key yönetimi | Bireysel API key dağıtmak yerine gateway üzerinden |
| Rate limiting | Kullanıcı/takım bazlı hız sınırı |
| Maliyet kontrolü | Harcama limitleri ve raporlama |
| Güvenlik | mTLS, IP whitelist, istek filtreleme |
| İzleme | Tüm isteklerin merkezi loglanması |
| Failover | Birden fazla provider arasında otomatik geçiş |
| Caching | Tekrarlayan isteklerin önbelleklenmesi |

---

## Claude Code Gateway Konfigürasyonu

Claude Code'u bir LLM Gateway üzerinden kullanmak için `ANTHROPIC_BASE_URL` değişkeni ayarlanır:

```bash
# Gateway endpoint'ini ayarla
export ANTHROPIC_BASE_URL="https://llm-gateway.company.com/v1"

# API key (gateway'in beklediği format)
export ANTHROPIC_API_KEY="gateway-key-xxxx"

# Claude Code başlat
claude
```

```mermaid
flowchart LR
    CC["Claude Code<br/>ANTHROPIC_BASE_URL=<br/>https://gateway/v1"] --> GW["LLM Gateway<br/>https://gateway/v1"]
    GW --> API["Anthropic API<br/>https://api.anthropic.com"]

```

---

## Popüler LLM Gateway Çözümleri

### Açık Kaynak Çözümler

```mermaid
flowchart TD
    subgraph opensource ["Açık Kaynak LLM Gateway'ler"]
        LITELLM["LiteLLM Proxy<br/>Python tabanlı<br/>Çoklu provider desteği"]
        PORTKEY["Portkey<br/>Enterprise-ready<br/>Observability"]
        KONG["Kong AI Gateway<br/>Kong tabanlı<br/>Plugin ekosistemi"]
    end

```

### LiteLLM Proxy Kurulumu

```bash
# LiteLLM proxy kurulumu
pip install litellm[proxy]

# Konfigürasyon dosyası
cat > litellm_config.yaml << 'EOF'
model_list:
  - model_name: "claude-sonnet-4"
    litellm_params:
      model: "claude-sonnet-4-20250514"
      api_key: "YOUR_API_KEY_HERE"

  - model_name: "claude-opus-4"
    litellm_params:
      model: "claude-opus-4-20250514"
      api_key: "YOUR_API_KEY_HERE"

general_settings:
  master_key: "YOUR_GATEWAY_KEY_HERE"

litellm_settings:
  max_budget: 1000
  budget_duration: "30d"
EOF

# Proxy başlat
litellm --config litellm_config.yaml --port 4000
```

Claude Code'da kullanım:

```bash
export ANTHROPIC_BASE_URL="http://localhost:4000/v1"
export ANTHROPIC_API_KEY="YOUR_GATEWAY_KEY_HERE"
claude
```

---

## Cloud Provider Entegrasyonları

### AWS Bedrock

AWS Bedrock zaten bir tür gateway işlevi görür ve ek gateway gerekmeyebilir:

```bash
export CLAUDE_CODE_USE_BEDROCK=true
export AWS_REGION="us-east-1"
export ANTHROPIC_MODEL="anthropic.claude-sonnet-4-20250514-v1:0"
```

### Google Vertex AI

```bash
export CLAUDE_CODE_USE_VERTEX=true
export CLOUD_ML_REGION="us-east5"
export ANTHROPIC_VERTEX_PROJECT_ID="my-project"
export ANTHROPIC_MODEL="claude-sonnet-4@20250514"
```

### Cloud Provider + Gateway Kombinasyonu

```mermaid
flowchart TD
    CC["Claude Code"] --> GW["LLM Gateway"]

    GW --> |"Yüksek öncelik"| BEDROCK["AWS Bedrock<br/>(Birincil)"]
    GW --> |"Failover"| VERTEX["Google Vertex AI<br/>(Yedek)"]
    GW --> |"Son çare"| DIRECT["Anthropic API<br/>(Doğrudan)"]

```

---

## Gateway Güvenlik Konfigürasyonu

### API Key Yönetimi

```mermaid
flowchart TD
    ADMIN["🏢 Admin"] --> GW_KEY["Gateway Master Key<br/>oluştur"]
    GW_KEY --> USER_KEYS["Kullanıcı Anahtarları<br/>oluştur"]

    USER_KEYS --> K1["Dev Key 1<br/>Limit: $50/gün"]
    USER_KEYS --> K2["Dev Key 2<br/>Limit: $50/gün"]
    USER_KEYS --> K3["CI/CD Key<br/>Limit: $200/gün"]

    K1 --> GW["LLM Gateway"]
    K2 --> GW
    K3 --> GW

    GW --> API["Anthropic API<br/>Tek master key"]

```

### Rate Limiting Konfigürasyonu

```yaml
# Gateway rate limit konfigürasyonu
rate_limits:
  - key: "user"
    max_requests: 100
    time_window: "1m"

  - key: "team"
    max_requests: 500
    time_window: "1m"

  - key: "organization"
    max_requests: 2000
    time_window: "1m"
    max_tokens: 1000000
```

---

## Pratik Örnek: Kurumsal Gateway Mimarisi

```mermaid
flowchart TD
    subgraph devs ["Geliştiriciler"]
        D1["Dev 1<br/>Frontend"]
        D2["Dev 2<br/>Backend"]
        D3["Dev 3<br/>DevOps"]
    end

    subgraph cicd ["CI/CD"]
        CI["GitHub Actions"]
    end

    subgraph gateway_layer ["Gateway Katmanı"]
        LB["Load Balancer"]
        GW1["Gateway Node 1"]
        GW2["Gateway Node 2"]
    end

    subgraph monitoring ["İzleme"]
        METRICS["Prometheus<br/>Metrikler"]
        LOGS["ELK Stack<br/>Loglar"]
    end

    devs --> LB
    cicd --> LB
    LB --> GW1
    LB --> GW2

    GW1 --> ANTHROPIC["Anthropic API"]
    GW2 --> ANTHROPIC

    GW1 --> METRICS
    GW2 --> METRICS
    GW1 --> LOGS
    GW2 --> LOGS

```

---

## Sorun Giderme

| Sorun | Olası Neden | Çözüm |
|-------|-------------|-------|
| `Connection refused` | Gateway çalışmıyor | Gateway servisini başlatın |
| `401 Unauthorized` | Yanlış gateway key | API key'i kontrol edin |
| `429 Too Many Requests` | Rate limit aşıldı | Limit ayarlarını kontrol edin veya bekleyin |
| `502 Bad Gateway` | Gateway → API bağlantı sorunu | Gateway'in API'ye erişimini kontrol edin |
| Yavaş yanıtlar | Gateway overhead | Gateway performans metriklerini inceleyin |

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| Gateway olmadan her geliştiriciye API key vermek | Merkezi gateway kullanın |
| Rate limit koymamak | Kullanıcı ve takım bazlı limitler tanımlayın |
| Failover yapılandırmamak | Birden fazla provider tanımlayın |
| Gateway loglarını izlememek | Monitoring sistemi kurun |

---

## Özet

| Konu | Anahtar Bilgi |
|------|---------------|
| Konfigürasyon | `ANTHROPIC_BASE_URL` ile gateway endpoint'i |
| Açık kaynak | LiteLLM, Portkey, Kong AI Gateway |
| Cloud | Bedrock, Vertex AI (yerleşik gateway) |
| Güvenlik | Merkezi key yönetimi, rate limiting |
| İzleme | Gateway logları ve metrikleri |

---

## Sonraki Adım

Cihaz yönetimi altyapısı olmadan merkezi konfigürasyon uygulamak için:

→ [Sunucu Tabanlı Ayarlar](./06-sunucu-tabanli-ayarlar.md)
