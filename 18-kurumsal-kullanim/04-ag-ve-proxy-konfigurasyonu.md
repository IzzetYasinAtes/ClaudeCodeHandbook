# Ağ ve Proxy Konfigürasyonu

Kurumsal ağ ortamlarında Claude Code'un internet erişimi genellikle proxy servers (vekil sunucular), custom CA certificates (özel CA sertifikaları) ve mTLS (mutual TLS) authentication (karşılıklı TLS kimlik doğrulama) gibi güvenlik katmanlarından geçer. Bu rehber, bu ortamlarda Claude Code'un sorunsuz çalışması için gerekli konfigürasyonu kapsar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Ortam değişkenleri | [Ortam Değişkenleri](../17-konfigurasyon/03-ortam-degiskenleri.md) |
| Temel ağ bilgisi | Harici kaynak |

---

## Kurumsal Ağ Topolojisi

Tipik bir kurumsal ortamda Claude Code'un Anthropic API'sine erişim yolu:

```mermaid
flowchart LR
    CC["💻 Claude Code<br/>(Geliştirici Makinesi)"] --> PROXY["🔒 Proxy Server<br/>(Forward Proxy)"]
    PROXY --> FIREWALL["🧱 Firewall<br/>(Çıkış kuralları)"]
    FIREWALL --> INTERNET["🌐 İnternet"]
    INTERNET --> API["☁️ Anthropic API<br/>api.anthropic.com"]

    subgraph corporate ["Kurumsal Ağ"]
        CC
        PROXY
        FIREWALL
    end

    style CC fill:#3498DB,color:#fff
    style PROXY fill:#E67E22,color:#fff
    style FIREWALL fill:#E74C3C,color:#fff
    style API fill:#27AE60,color:#fff
    style corporate fill:#ECF0F1,stroke:#2C3E50
```

---

## Proxy Konfigürasyonu

### HTTP/HTTPS Proxy

```bash
# HTTPS proxy ayarı (en yaygın)
export HTTPS_PROXY="http://proxy.company.com:8080"

# HTTP proxy ayarı
export HTTP_PROXY="http://proxy.company.com:8080"

# Proxy'den hariç tutulacak adresler
export NO_PROXY="localhost,127.0.0.1,.company.internal,10.0.0.0/8"
```

### Kimlik Doğrulamalı Proxy

```bash
# Kullanıcı adı ve şifre ile proxy
export HTTPS_PROXY="http://user:password@proxy.company.com:8080"

# Özel karakterler URL-encoded olmalıdır
# @ → %40, : → %3A, # → %23
export HTTPS_PROXY="http://john.doe%40company.com:p%40ssw0rd@proxy.company.com:8080"
```

### SOCKS5 Proxy

```bash
# SOCKS5 proxy kullanımı
export HTTPS_PROXY="socks5://proxy.company.com:1080"
export HTTP_PROXY="socks5://proxy.company.com:1080"
```

---

## CA Sertifika Konfigürasyonu

Kurumsal ortamlarda TLS/SSL trafiğini inspect (denetim) eden sistemler genellikle kendi CA (Certificate Authority) sertifikalarını kullanır. Claude Code'un bu sertifikalara güvenmesi gerekir.

```mermaid
flowchart TD
    CC["Claude Code"] --> TLS["TLS Bağlantısı"]
    TLS --> INSPECT{"SSL Inspection<br/>Proxy?"}
    INSPECT -->|Evet| CORP_CA["Kurumsal CA<br/>Sertifikası gerekli"]
    INSPECT -->|Hayır| DIRECT["Doğrudan bağlantı"]

    CORP_CA --> CONFIG["NODE_EXTRA_CA_CERTS<br/>ile sertifika tanımla"]
    CONFIG --> SUCCESS["✅ Bağlantı başarılı"]

    DIRECT --> SUCCESS

    style CC fill:#3498DB,color:#fff
    style CORP_CA fill:#E74C3C,color:#fff
    style CONFIG fill:#F39C12,color:#fff
    style SUCCESS fill:#27AE60,color:#fff
```

### Sertifika Ayarları

```bash
# Node.js ek CA sertifikası (önerilen yöntem)
export NODE_EXTRA_CA_CERTS="/etc/ssl/certs/corporate-root-ca.pem"

# Alternatif: SSL sertifika dosyası
export SSL_CERT_FILE="/etc/ssl/certs/ca-bundle.crt"

# ⚠️ DİKKAT: Sadece geliştirme/debug için!
# Üretimde ASLA kullanmayın
export NODE_TLS_REJECT_UNAUTHORIZED=0
```

### Sertifika Zinciri Oluşturma

Birden fazla CA sertifikası gerektiğinde hepsini tek bir dosyada birleştirin:

```bash
# Sertifika zinciri oluşturma
cat /etc/ssl/certs/corporate-root-ca.pem \
    /etc/ssl/certs/corporate-intermediate-ca.pem \
    > /etc/ssl/certs/corporate-ca-chain.pem

# Birleşik sertifikayı kullan
export NODE_EXTRA_CA_CERTS="/etc/ssl/certs/corporate-ca-chain.pem"
```

### Sertifika Doğrulama

```bash
# Sertifikanın geçerli olduğunu kontrol et
openssl x509 -in /etc/ssl/certs/corporate-root-ca.pem -text -noout

# Sertifika süresini kontrol et
openssl x509 -in /etc/ssl/certs/corporate-root-ca.pem -enddate -noout

# Claude Code ile bağlantıyı test et
curl -v --cacert /etc/ssl/certs/corporate-ca-chain.pem https://api.anthropic.com/v1/messages
```

---

## mTLS (Mutual TLS) Kimlik Doğrulama

Bazı kurumsal ortamlarda mutual TLS (karşılıklı TLS) gereklidir. Bu durumda hem sunucu hem de istemci sertifika sunar.

```mermaid
flowchart LR
    subgraph client ["İstemci (Claude Code)"]
        CC["Claude Code"]
        CLIENT_CERT["İstemci Sertifikası<br/>+ Özel Anahtar"]
    end

    subgraph server ["Sunucu (API / Gateway)"]
        API["API Endpoint"]
        SERVER_CERT["Sunucu Sertifikası"]
    end

    CC --> |"1. TLS Handshake<br/>(sunucu sertifikası doğrula)"| API
    API --> |"2. İstemci sertifikası iste"| CC
    CC --> |"3. İstemci sertifikası gönder"| API
    API --> |"4. Doğrulama başarılı ✅"| CC

    style CC fill:#3498DB,color:#fff
    style API fill:#27AE60,color:#fff
    style CLIENT_CERT fill:#F39C12,color:#fff
    style SERVER_CERT fill:#9B59B6,color:#fff
```

### mTLS Konfigürasyonu

```bash
# İstemci sertifikası ve anahtarı
export NODE_EXTRA_CA_CERTS="/etc/ssl/certs/corporate-ca.pem"

# mTLS için özel proxy/gateway kullanılır
# LLM Gateway üzerinden mTLS yapılandırması önerilir
export ANTHROPIC_BASE_URL="https://llm-gateway.company.com/v1"
```

---

## Firewall Kuralları

Claude Code'un düzgün çalışması için gereken ağ erişimleri:

### Gerekli Endpoint'ler

| Endpoint | Port | Protokol | Amaç |
|----------|------|----------|------|
| `api.anthropic.com` | 443 | HTTPS | Ana API endpoint |
| `cdn.anthropic.com` | 443 | HTTPS | Model indirme (gerekirse) |
| `console.anthropic.com` | 443 | HTTPS | OAuth kimlik doğrulama |
| `sentry.io` | 443 | HTTPS | Hata raporlama (opsiyonel) |
| `statsig.anthropic.com` | 443 | HTTPS | Feature flags (opsiyonel) |

### Opsiyonel Endpoint'ler

| Endpoint | Amaç | Devre Dışı Bırakma |
|----------|-------|---------------------|
| `sentry.io` | Crash raporlama | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true` |
| `statsig.anthropic.com` | Telemetri | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true` |

```bash
# Gereksiz trafik kaynaklarını kapat
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true
```

---

## Pratik Örnek: Tam Kurumsal Ağ Kurulumu

### Shell Profili

```bash
# ~/.bashrc veya ~/.zshrc - Kurumsal ağ konfigürasyonu

# Proxy ayarları
export HTTPS_PROXY="http://proxy.company.com:8080"
export HTTP_PROXY="http://proxy.company.com:8080"
export NO_PROXY="localhost,127.0.0.1,.company.internal,10.0.0.0/8,172.16.0.0/12"

# CA sertifikası
export NODE_EXTRA_CA_CERTS="/etc/ssl/certs/company-ca-chain.pem"

# Claude Code API
export ANTHROPIC_API_KEY="YOUR_API_KEY_HERE"

# Opsiyonel trafiği kapat
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true
```

### Doğrulama Kontrol Listesi

```mermaid
flowchart TD
    START["🔧 Kurulum Başla"] --> PROXY_CHECK{"Proxy<br/>erişimi var mı?"}
    PROXY_CHECK -->|Evet| CERT_CHECK{"CA sertifikası<br/>yüklendi mi?"}
    PROXY_CHECK -->|Hayır| FIX_PROXY["HTTPS_PROXY<br/>ayarlayın"]
    FIX_PROXY --> PROXY_CHECK

    CERT_CHECK -->|Evet| API_CHECK{"API erişimi<br/>var mı?"}
    CERT_CHECK -->|Hayır| FIX_CERT["NODE_EXTRA_CA_CERTS<br/>ayarlayın"]
    FIX_CERT --> CERT_CHECK

    API_CHECK -->|Evet| SUCCESS["✅ Claude Code<br/>Hazır"]
    API_CHECK -->|Hayır| DEBUG["Debug:<br/>curl -v api.anthropic.com"]
    DEBUG --> FIX_PROXY

    style SUCCESS fill:#27AE60,color:#fff
    style FIX_PROXY fill:#E74C3C,color:#fff
    style FIX_CERT fill:#E74C3C,color:#fff
```

### Test Komutları

```bash
# 1. Proxy bağlantısını test et
curl -v -x $HTTPS_PROXY https://api.anthropic.com

# 2. CA sertifikası ile test et
curl -v --cacert $NODE_EXTRA_CA_CERTS https://api.anthropic.com

# 3. Claude Code ile tam test
claude --version
claude "Merhaba, bağlantı testi"
```

---

## Sorun Giderme

| Sorun | Olası Neden | Çözüm |
|-------|-------------|-------|
| `ECONNREFUSED` | Proxy erişimi yok | `HTTPS_PROXY` değerini kontrol edin |
| `UNABLE_TO_VERIFY_LEAF_SIGNATURE` | CA sertifikası eksik | `NODE_EXTRA_CA_CERTS` ayarlayın |
| `SELF_SIGNED_CERT_IN_CHAIN` | Self-signed sertifika | Kurumsal CA'yı `NODE_EXTRA_CA_CERTS`'e ekleyin |
| `ECONNRESET` | Firewall engeli | IT ekibinden `api.anthropic.com:443` açılmasını isteyin |
| `407 Proxy Authentication Required` | Proxy credential yanlış | Kullanıcı/şifre bilgisini kontrol edin |
| Yavaş yanıtlar | Proxy yükü | Proxy admin ile iletişime geçin |

---

## Sık Yapılan Hatalar

| Hata | Çözüm |
|------|-------|
| `NODE_TLS_REJECT_UNAUTHORIZED=0` üretimde kullanmak | Hiçbir zaman üretimde kullanmayın; CA sertifikasını doğru ayarlayın |
| NO_PROXY'yi unutmak | İç ağ adresleri NO_PROXY'ye ekleyin |
| Sertifika süresini kontrol etmemek | Sertifika süresi dolabilir, periyodik kontrol yapın |
| Proxy credential'ı koda gömmek | Ortam değişkeni veya secret manager kullanın |

---

## Özet

| Konu | Anahtar Konfigürasyon |
|------|----------------------|
| Proxy | `HTTPS_PROXY`, `NO_PROXY` |
| CA Sertifikası | `NODE_EXTRA_CA_CERTS` |
| mTLS | LLM Gateway üzerinden |
| Firewall | `api.anthropic.com:443` açık olmalı |
| Güvenlik | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true` |

---

## Sonraki Adım

Proxy yerine veya proxy ile birlikte LLM Gateway kullanarak API trafiğini yönetmek için:

→ [LLM Gateway](./05-llm-gateway.md)
