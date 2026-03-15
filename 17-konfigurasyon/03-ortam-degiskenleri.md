# Ortam Değişkenleri

Claude Code, birçok davranışını environment variables (ortam değişkenleri) aracılığıyla kontrol etmeye olanak tanır. API anahtarlarından proxy ayarlarına, dosya yollarından davranış değişikliklerine kadar geniş bir yelpazede yapılandırma sağlar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Settings.json referansı | [Settings.json Referansı](./02-settings-json-referansi.md) |
| Ayar dosyaları hiyerarşisi | [Ayar Dosyaları Hiyerarşisi](./01-ayar-dosyalari-hiyerarsisi.md) |

---

## Ortam Değişkenleri Genel Bakış

```mermaid
flowchart TD
    subgraph sources ["Ortam Değişkeni Kaynakları"]
        direction TB
        SHELL["🖥️ Shell / Terminal\nexport ANTHROPIC_API_KEY=..."]
        ENV_FILE["📄 .env Dosyası\nCLAUDE_ENV_FILE ile yüklenir"]
        SETTINGS["⚙️ settings.json env bloğu\nenv: { KEY: value }"]
    end

    subgraph categories ["Değişken Kategorileri"]
        direction TB
        AUTH["🔑 Kimlik Doğrulama\nAPI anahtarları"]
        BEHAVIOR["🎛️ Davranış\nMod, yol, dizin ayarları"]
        NETWORK["🌐 Ağ\nProxy, SSL ayarları"]
        DISPLAY["🎨 Görünüm\nTema, renk ayarları"]
    end

    sources --> categories

    style AUTH fill:#E74C3C,color:#fff
    style BEHAVIOR fill:#3498DB,color:#fff
    style NETWORK fill:#F39C12,color:#fff
    style DISPLAY fill:#9B59B6,color:#fff
```

---

## Kimlik Doğrulama Değişkenleri

| Değişken | Varsayılan | Açıklama |
|----------|------------|----------|
| `ANTHROPIC_API_KEY` | — | Anthropic API anahtarı (doğrudan erişim) |
| `ANTHROPIC_AUTH_TOKEN` | — | OAuth/kurumsal kimlik doğrulama token'ı |
| `CLAUDE_API_KEY` | — | `ANTHROPIC_API_KEY` ile aynı (alternatif isim) |

```bash
# Doğrudan API anahtarı ile
export ANTHROPIC_API_KEY="sk-ant-api03-xxxxxxxxxxxx"

# Kurumsal OAuth token ile
export ANTHROPIC_AUTH_TOKEN="eyJhbGciOiJSUzI1NiIs..."
```

---

## Davranış Değişkenleri

| Değişken | Varsayılan | Açıklama |
|----------|------------|----------|
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | `false` | `true` olduğunda her Bash komutu proje kök dizininde başlar |
| `CLAUDE_CODE_MAX_TURNS` | — | Non-interactive modda maksimum turn sayısı |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | `false` | Telemetri ve analitik trafiğini devre dışı bırakır |
| `CLAUDE_CODE_USE_BEDROCK` | `false` | AWS Bedrock üzerinden Claude kullanımını etkinleştirir |
| `CLAUDE_CODE_USE_VERTEX` | `false` | Google Vertex AI üzerinden Claude kullanımını etkinleştirir |
| `CLAUDE_CODE_SKIP_CLAUDE_MD` | `false` | CLAUDE.md dosyasının yüklenmesini atlar |
| `CLAUDE_TEST_MODE` | `false` | Test modunu etkinleştirir |

```bash
# Bash komutlarını her zaman proje kökünde başlat
export CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=true

# CI/CD'de maksimum turn sınırı
export CLAUDE_CODE_MAX_TURNS=25

# Telemetri trafiğini kapat
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true
```

---

## Dosya ve Dizin Değişkenleri

| Değişken | Varsayılan | Açıklama |
|----------|------------|----------|
| `CLAUDE_ENV_FILE` | — | Ek ortam değişkenlerinin yükleneceği `.env` dosya yolu |
| `CLAUDE_CONFIG_DIR` | `~/.claude` | Claude Code konfigürasyon dizini |

```bash
# Özel .env dosyası kullanma
export CLAUDE_ENV_FILE="/path/to/project/.env.claude"

# Özel konfigürasyon dizini
export CLAUDE_CONFIG_DIR="/home/user/.config/claude"
```

### .env Dosyası Kullanımı

`CLAUDE_ENV_FILE` ile harici bir ortam dosyası yükleyebilirsiniz:

```bash
# .env.claude dosyası
DATABASE_URL=postgresql://localhost:5432/mydb
REDIS_URL=redis://localhost:6379
API_SECRET=my-secret-key
NODE_ENV=development
```

```bash
# Kullanım
export CLAUDE_ENV_FILE=".env.claude"
claude
```

```mermaid
flowchart LR
    ENV_FILE[".env.claude\nDosyası"] --> LOAD["CLAUDE_ENV_FILE\nDeğişkeni ile yüklenir"]
    LOAD --> SESSION["Claude Code\nOturumu"]
    SESSION --> USE["Değişkenler\nBash komutlarında kullanılır"]

    style ENV_FILE fill:#27AE60,color:#fff
    style SESSION fill:#3498DB,color:#fff
```

---

## Ağ ve Proxy Değişkenleri

| Değişken | Varsayılan | Açıklama |
|----------|------------|----------|
| `ANTHROPIC_BASE_URL` | `https://api.anthropic.com` | Anthropic API base URL (proxy/gateway için) |
| `HTTP_PROXY` | — | HTTP proxy sunucusu |
| `HTTPS_PROXY` | — | HTTPS proxy sunucusu |
| `NO_PROXY` | — | Proxy'den hariç tutulacak adresler |
| `NODE_TLS_REJECT_UNAUTHORIZED` | `1` | `0` ile self-signed sertifika kabul edilir (güvenlik riski!) |
| `SSL_CERT_FILE` | — | Özel CA sertifika dosyası yolu |
| `NODE_EXTRA_CA_CERTS` | — | Ek CA sertifika dosyası |

```bash
# Kurumsal proxy ayarları
export HTTPS_PROXY="http://proxy.corporate.com:8080"
export NO_PROXY="localhost,127.0.0.1,.internal.com"

# Özel API endpoint (LLM Gateway)
export ANTHROPIC_BASE_URL="https://llm-gateway.company.com/v1"

# Özel CA sertifikası
export NODE_EXTRA_CA_CERTS="/etc/ssl/certs/corporate-ca.pem"
```

---

## Cloud Provider Değişkenleri

### AWS Bedrock

| Değişken | Varsayılan | Açıklama |
|----------|------------|----------|
| `CLAUDE_CODE_USE_BEDROCK` | `false` | Bedrock kullanımını etkinleştirir |
| `AWS_REGION` | — | AWS bölgesi |
| `AWS_ACCESS_KEY_ID` | — | AWS erişim anahtarı |
| `AWS_SECRET_ACCESS_KEY` | — | AWS gizli anahtar |
| `AWS_SESSION_TOKEN` | — | AWS oturum token'ı (geçici credential) |
| `ANTHROPIC_MODEL` | — | Kullanılacak model ID'si |

```bash
export CLAUDE_CODE_USE_BEDROCK=true
export AWS_REGION="us-east-1"
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export ANTHROPIC_MODEL="anthropic.claude-sonnet-4-20250514-v1:0"
```

### Google Vertex AI

| Değişken | Varsayılan | Açıklama |
|----------|------------|----------|
| `CLAUDE_CODE_USE_VERTEX` | `false` | Vertex AI kullanımını etkinleştirir |
| `CLOUD_ML_REGION` | — | GCP bölgesi |
| `ANTHROPIC_VERTEX_PROJECT_ID` | — | GCP proje ID'si |

```bash
export CLAUDE_CODE_USE_VERTEX=true
export CLOUD_ML_REGION="us-east5"
export ANTHROPIC_VERTEX_PROJECT_ID="my-gcp-project"
export ANTHROPIC_MODEL="claude-sonnet-4@20250514"
```

---

## Tam Referans Tablosu

```mermaid
flowchart TD
    subgraph auth ["🔑 Kimlik Doğrulama"]
        A1["ANTHROPIC_API_KEY"]
        A2["ANTHROPIC_AUTH_TOKEN"]
        A3["CLAUDE_API_KEY"]
    end

    subgraph behavior ["🎛️ Davranış"]
        B1["CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR"]
        B2["CLAUDE_CODE_MAX_TURNS"]
        B3["CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC"]
        B4["CLAUDE_CODE_SKIP_CLAUDE_MD"]
    end

    subgraph files ["📁 Dosya / Dizin"]
        F1["CLAUDE_ENV_FILE"]
        F2["CLAUDE_CONFIG_DIR"]
    end

    subgraph network ["🌐 Ağ"]
        N1["ANTHROPIC_BASE_URL"]
        N2["HTTPS_PROXY"]
        N3["NODE_EXTRA_CA_CERTS"]
    end

    subgraph cloud ["☁️ Cloud Provider"]
        C1["CLAUDE_CODE_USE_BEDROCK"]
        C2["CLAUDE_CODE_USE_VERTEX"]
        C3["ANTHROPIC_MODEL"]
    end

    style auth fill:#E74C3C,color:#fff
    style behavior fill:#3498DB,color:#fff
    style files fill:#27AE60,color:#fff
    style network fill:#F39C12,color:#fff
    style cloud fill:#9B59B6,color:#fff
```

---

## Pratik Örnek: Kapsamlı Ortam Kurulumu

### Kurumsal Ortam için Shell Profili

```bash
# ~/.bashrc veya ~/.zshrc

# Claude Code Kimlik Doğrulama
export ANTHROPIC_API_KEY="sk-ant-api03-xxxx"

# Proxy Ayarları
export HTTPS_PROXY="http://proxy.company.com:8080"
export NO_PROXY="localhost,127.0.0.1,.company.internal"

# Özel CA Sertifikası
export NODE_EXTRA_CA_CERTS="/etc/ssl/certs/company-root-ca.pem"

# Davranış Ayarları
export CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=true
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true

# Proje-spesifik ortam dosyası
export CLAUDE_ENV_FILE=".env.claude"
```

### CI/CD Pipeline için Ortam

```yaml
# GitHub Actions örneği
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  CLAUDE_CODE_MAX_TURNS: "25"
  CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC: "true"
  CLAUDE_CODE_SKIP_CLAUDE_MD: "false"
```

### Docker Container İçinde

```dockerfile
FROM node:20-slim

ENV ANTHROPIC_API_KEY=""
ENV CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=true
ENV CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true

RUN npm install -g @anthropic-ai/claude-code

ENTRYPOINT ["claude"]
```

---

## Öncelik Sırası

Ortam değişkenleri birden fazla kaynaktan gelebilir. Öncelik sırası:

```
Shell ortamı > CLAUDE_ENV_FILE > settings.json env bloğu
```

| Öncelik | Kaynak | Örnek |
|---------|--------|-------|
| 1 (En yüksek) | Shell / terminal ortamı | `export API_KEY=xxx` |
| 2 | `CLAUDE_ENV_FILE` ile yüklenen dosya | `.env.claude` içeriği |
| 3 (En düşük) | `settings.json` → `env` bloğu | `"env": { "API_KEY": "xxx" }` |

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| API anahtarını koda gömmek | Ortam değişkeni veya secret manager kullanın |
| `NODE_TLS_REJECT_UNAUTHORIZED=0` üretimde kullanmak | Sadece geliştirme ortamında ve geçici olarak kullanın |
| Proxy ayarlarını unutmak | Kurumsal ağlarda `HTTPS_PROXY` gereklidir |
| Bedrock/Vertex flag'ini ayarlamamak | `CLAUDE_CODE_USE_BEDROCK=true` veya `CLAUDE_CODE_USE_VERTEX=true` gerekir |

---

## Özet

| Kategori | Anahtar Değişkenler |
|----------|-------------------|
| Kimlik Doğrulama | `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` |
| Davranış | `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR`, `CLAUDE_CODE_MAX_TURNS` |
| Dosya/Dizin | `CLAUDE_ENV_FILE`, `CLAUDE_CONFIG_DIR` |
| Ağ | `ANTHROPIC_BASE_URL`, `HTTPS_PROXY`, `NODE_EXTRA_CA_CERTS` |
| Cloud | `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX` |

---

## Sonraki Adım

Model seçimi ve Extended Thinking gibi performans ayarlarını konfigüre etmeyi öğrenelim:

→ [Model Konfigürasyonu](./04-model-konfigurasyonu.md)
