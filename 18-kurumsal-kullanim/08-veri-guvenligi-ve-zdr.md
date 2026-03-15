# Veri Güvenliği ve Zero Data Retention (ZDR)

Kurumsal ortamlarda Claude Code kullanırken veri güvenliği en kritik konudur. Zero Data Retention (sıfır veri saklama) politikası, gönderilen verilerin Anthropic tarafından model eğitiminde veya kalıcı depolamada kullanılmamasını garanti eder.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code temelleri | [Claude Code Nedir?](../06-claude-code-tanitim/01-claude-code-nedir.md) |
| Güvenlik en iyi uygulamalar | [Güvenlik En İyi Uygulamalar](../10-izinler-ve-guvenlik/05-guvenlik-en-iyi-uygulamalar.md) |
| Ağ konfigürasyonu | [Ağ ve Proxy Konfigürasyonu](./04-ag-ve-proxy-konfigurasyonu.md) |

---

## Veri Akışı Genel Bakış

Claude Code kullandığınızda aşağıdaki veri akışı gerçekleşir:

```mermaid
flowchart LR
    subgraph local ["Yerel Ortam"]
        CC["Claude Code"]
        CODE["📁 Kaynak Kod"]
        CONTEXT["Konuşma Bağlamı"]
    end

    subgraph transit ["Aktarım"]
        TLS["🔒 TLS 1.3\nŞifreli bağlantı"]
    end

    subgraph api ["Anthropic API"]
        PROCESS["İşleme\nModel inference"]
        RESPONSE["Yanıt üretme"]
    end

    subgraph storage ["Veri Saklama"]
        ZDR_ON["ZDR Aktif ✅\nVeri saklanmaz\n30 gün log (sınırlı)"]
        ZDR_OFF["ZDR Pasif ⚠️\nGüvenlik logları\n30 gün saklama"]
    end

    CODE --> CC
    CONTEXT --> CC
    CC --> TLS
    TLS --> PROCESS
    PROCESS --> RESPONSE
    RESPONSE --> TLS
    TLS --> CC

    PROCESS --> ZDR_ON
    PROCESS --> ZDR_OFF

    style CC fill:#3498DB,color:#fff
    style TLS fill:#27AE60,color:#fff
    style PROCESS fill:#F39C12,color:#fff
    style ZDR_ON fill:#27AE60,color:#fff
    style ZDR_OFF fill:#E74C3C,color:#fff
```

---

## Zero Data Retention (ZDR) Nedir?

ZDR, Anthropic'in veri saklama politikasının en kısıtlayıcı seviyesidir. ZDR etkinleştirildiğinde:

### ZDR Kapsamı

| Kapsam | ZDR Aktif | ZDR Pasif |
|--------|-----------|-----------|
| Prompt'lar ve yanıtlar | ❌ Saklanmaz | 30 gün (güvenlik) |
| Model eğitimi | ❌ Kullanılmaz | ❌ Kullanılmaz |
| Kötüye kullanım tespiti | Sınırlı | Tam |
| Trust & Safety değerlendirmesi | Sınırlı | Tam |
| Hata ayıklama | Sınırlı | Mümkün |

### ZDR ile Devre Dışı Kalan Özellikler

```mermaid
flowchart TD
    ZDR["ZDR Etkinleştirildi"] --> DISABLED["Devre Dışı Kalan Özellikler"]

    DISABLED --> D1["📊 Kullanım Analitiği\nDetaylı kullanım logları"]
    DISABLED --> D2["🔍 Prompt Caching\nBazı cache özellikleri"]
    DISABLED --> D3["📝 Conversation History\nSunucu taraflı geçmiş"]
    DISABLED --> D4["🐛 Debug Support\nAnthropic debug desteği sınırlı"]

    style ZDR fill:#E74C3C,color:#fff
    style D1 fill:#95A5A6,color:#fff
    style D2 fill:#95A5A6,color:#fff
    style D3 fill:#95A5A6,color:#fff
    style D4 fill:#95A5A6,color:#fff
```

> **Not:** ZDR etkinleştirildiğinde bazı özellikler kısıtlanabilir. Güncel bilgi için Anthropic ile iletişime geçin.

---

## ZDR Etkinleştirme

### Adım 1: Uygunluk

ZDR, Enterprise plan kapsamında sunulmaktadır:

```mermaid
flowchart TD
    REQUEST["ZDR Talebi"] --> CONTACT["Anthropic Sales\nveya Account Manager"]
    CONTACT --> REVIEW["Talep İnceleme"]
    REVIEW --> AGREEMENT["ZDR Anlaşması\nİmzalama"]
    AGREEMENT --> ENABLE["ZDR Etkinleştirme\nOrganizasyon seviyesinde"]
    ENABLE --> VERIFY["Doğrulama"]

    style REQUEST fill:#3498DB,color:#fff
    style AGREEMENT fill:#F39C12,color:#fff
    style ENABLE fill:#27AE60,color:#fff
```

### Adım 2: Talep Süreci

1. **Anthropic sales ekibiyle iletişime geçin** — [sales@anthropic.com](mailto:sales@anthropic.com)
2. **ZDR gereksinimlerinizi belirtin** — Hangi organizasyon/takım için
3. **Anlaşma şartlarını gözden geçirin** — Devre dışı kalan özellikler dahil
4. **Anlaşmayı imzalayın** — ZDR addendum (ek sözleşme)
5. **Etkinleştirmeyi doğrulayın** — API yanıtlarında ZDR header kontrolü

### Adım 3: Doğrulama

```bash
# API yanıtında ZDR durumunu kontrol edin
# Yanıt header'larında ZDR bilgisi yer alır
curl -v https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "content-type: application/json" \
  -H "anthropic-version: 2024-01-01" \
  -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 10, "messages": [{"role": "user", "content": "test"}]}'
```

---

## Veri Kullanım Politikaları

### Anthropic Veri Politikası Özeti

```mermaid
flowchart TD
    subgraph api_usage ["API Kullanımı (Claude Code dahil)"]
        API_TRAIN["Model Eğitimi\n❌ Kullanılmaz"]
        API_SAFETY["Güvenlik\n✅ Sınırlı erişim"]
        API_LOG["Loglama\n30 gün (ZDR'sız)"]
    end

    subgraph consumer ["Tüketici Ürünleri (claude.ai)"]
        CON_TRAIN["Model Eğitimi\n⚠️ Opsiyonel (opt-out mevcut)"]
        CON_SAFETY["Güvenlik\n✅ Tam erişim"]
        CON_LOG["Loglama\n✅ Tam"]
    end

    style API_TRAIN fill:#27AE60,color:#fff
    style CON_TRAIN fill:#F39C12,color:#fff
```

### Claude Code Spesifik Politika

| Veri Türü | API Kullanımı | ZDR ile |
|-----------|---------------|---------|
| Kaynak kod | Eğitim için kullanılmaz | Saklanmaz |
| Prompt'lar | 30 gün güvenlik logu | Saklanmaz |
| Yanıtlar | 30 gün güvenlik logu | Saklanmaz |
| Dosya içerikleri | Eğitim için kullanılmaz | Saklanmaz |
| Metadata (token sayısı vb.) | Kullanım metrikleri | Sınırlı |

---

## Kurumsal Veri Güvenliği Stratejisi

### Katmanlı Güvenlik Yaklaşımı

```mermaid
flowchart TD
    subgraph layers ["Güvenlik Katmanları"]
        L1["1️⃣ Veri Sınıflandırma\nHangi veriler Claude'a gönderilebilir?"]
        L2["2️⃣ Erişim Kontrolü\nKim hangi verilere erişebilir?"]
        L3["3️⃣ Ağ Güvenliği\nProxy, mTLS, firewall"]
        L4["4️⃣ ZDR Politikası\nSunucu tarafı veri koruma"]
        L5["5️⃣ İzleme ve Denetim\nLog, audit, alert"]
    end

    L1 --> L2 --> L3 --> L4 --> L5

    style L1 fill:#E74C3C,color:#fff
    style L2 fill:#E67E22,color:#fff
    style L3 fill:#F39C12,color:#fff
    style L4 fill:#3498DB,color:#fff
    style L5 fill:#27AE60,color:#fff
```

### Veri Sınıflandırma Rehberi

| Sınıf | Açıklama | Claude Code ile Kullanım |
|-------|----------|--------------------------|
| 🟢 Genel | Açık kaynak kod, dökümantasyon | Serbestçe kullanılabilir |
| 🟡 İç kullanım | İç projeler, standart kod | ZDR önerilir |
| 🟠 Gizli | Müşteri verileri, iş mantığı | ZDR gerekli + ek önlemler |
| 🔴 Çok gizli | Credential'lar, PII, finansal | Claude Code ile kullanılmamalı |

### .gitignore ve Hook Korumaları

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"import sys,json; path=json.load(sys.stdin).get('file_path',''); sensitive=['.env','.pem','.key','credentials','secret','password']; sys.exit(1) if any(s in path.lower() for s in sensitive) else sys.exit(0)\""
          }
        ]
      }
    ]
  }
}
```

---

## Cloud Provider ile ZDR

### AWS Bedrock

AWS Bedrock varsayılan olarak verilerinizi Anthropic ile paylaşmaz:

```bash
export CLAUDE_CODE_USE_BEDROCK=true
# Bedrock'ta verileriniz AWS altyapınızda kalır
# Anthropic model eğitimi için kullanılmaz
```

### Google Vertex AI

Vertex AI'da da benzer şekilde verileriniz Google Cloud altyapınızda kalır:

```bash
export CLAUDE_CODE_USE_VERTEX=true
# Vertex AI'da verileriniz GCP projenizde kalır
```

```mermaid
flowchart TD
    subgraph options ["Veri Koruma Seçenekleri"]
        DIRECT["Doğrudan API\n+ ZDR"]
        BEDROCK["AWS Bedrock\nAWS altyapısında kalır"]
        VERTEX["Google Vertex AI\nGCP altyapısında kalır"]
    end

    DIRECT --> PROTECTION["Veri Koruması ✅"]
    BEDROCK --> PROTECTION
    VERTEX --> PROTECTION

    style DIRECT fill:#E74C3C,color:#fff
    style BEDROCK fill:#FF9900,color:#fff
    style VERTEX fill:#4285F4,color:#fff
    style PROTECTION fill:#27AE60,color:#fff
```

---

## Pratik Örnek: Güvenlik Kontrol Listesi

### Claude Code Dağıtımı Öncesi

| # | Kontrol | Durum |
|---|---------|-------|
| 1 | Veri sınıflandırma politikası tanımlandı mı? | ☐ |
| 2 | ZDR etkinleştirildi mi (gerekiyorsa)? | ☐ |
| 3 | Hassas dosya filtreleme hook'u kuruldu mu? | ☐ |
| 4 | Ağ güvenliği (proxy, TLS) yapılandırıldı mı? | ☐ |
| 5 | Managed settings ile güvenlik politikaları uygulandı mı? | ☐ |
| 6 | Denetim loglaması aktif mi? | ☐ |
| 7 | Harcama limitleri tanımlandı mı? | ☐ |
| 8 | Ekip güvenlik eğitimi verildi mi? | ☐ |

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| Credential'ları Claude Code ile paylaşmak | Hassas verileri asla prompt'a eklemeyin |
| ZDR olmadan hassas kodla çalışmak | Gizli veriler için ZDR zorunlu kılın |
| Veri sınıflandırma yapmamak | Hangi verilerin AI ile kullanılabileceğini tanımlayın |
| Bedrock/Vertex'i tercih etmemek | Kurumsal ortamda cloud provider seçeneğini değerlendirin |

---

## Özet

| Konu | Anahtar Bilgi |
|------|---------------|
| ZDR | Prompt ve yanıtlar saklanmaz, model eğitiminde kullanılmaz |
| API politikası | API verileri varsayılan olarak eğitimde kullanılmaz |
| Cloud providers | Bedrock/Vertex veriler sağlayıcı altyapısında kalır |
| Veri sınıflandırma | Yeşil, sarı, turuncu, kırmızı katmanlar |
| Koruma hook'ları | Hassas dosya erişimini otomatik engelleme |

---

## Sonraki Adım

Yasal uyumluluk, GDPR/KVKK ve sertifika gereksinimlerini inceleyelim:

→ [Yasal Uyumluluk](./09-yasal-uyumluluk.md)
