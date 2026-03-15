# Oturum Yönetimi (Session Management)

Claude Code her çalıştırıldığında bir **session** (oturum) oluşturur. Oturumlar arasında geçiş yapmak, önceki oturuma devam etmek ve oturum geçmişini yönetmek günlük iş akışınızın önemli bir parçasıdır. Bu bölümde `--continue`, `--resume`, `--session-id` ve `/resume` mekanizmalarını öğreneceksiniz.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Context window yönetimi | [Context Window Yönetimi](./05-context-window-yonetimi.md) |
| Claude Code temel kullanım | [Bölüm 06](../06-claude-code-tanitim/README.md) |

---

## Oturum Yaşam Döngüsü

Her Claude Code oturumu bir yaşam döngüsüne sahiptir:

```mermaid
flowchart TD
    START(["claude komutu<br/>çalıştırılır"]) --> INIT["Oturum Başlatma"]
    INIT --> LOAD["Bağlam Yükleme<br/>- CLAUDE.md<br/>- Auto Memory<br/>- Rules"]
    LOAD --> WORK["Çalışma<br/>(Görevler yürütülür)"]
    WORK --> CHECK{"Oturum<br/>bitiyor mu?"}
    CHECK -->|"Hayır"| WORK
    CHECK -->|"Evet"| SAVE["Oturum Kaydedilir<br/>(Geçmiş + Durum)"]
    SAVE --> END(["Oturum Sona Erer"])

    END -.->|"--continue"| RESUME_LATEST["En son oturuma<br/>devam et"]
    END -.->|"--resume"| RESUME_PICK["Belirli oturuma<br/>devam et"]
    END -.->|"Yeni oturum"| START

    style START fill:#4A90D9,color:#fff
    style END fill:#95A5A6,color:#fff
    style RESUME_LATEST fill:#27AE60,color:#fff
    style RESUME_PICK fill:#F39C12,color:#fff
```

---

## Cold Start Tax (Soğuk Başlangıç Maliyeti)

Her yeni oturum başlatıldığında, Claude Code'un bağlamı yeniden oluşturması gerekir. Bu, token harcaması anlamına gelir:

```mermaid
flowchart LR
    subgraph cold ["Yeni Oturum (Cold Start)"]
        direction TB
        C1["CLAUDE.md oku (~2-5K token)"]
        C2["Auto memory oku (~1-2K)"]
        C3["Proje yapısını tara (~3-5K)"]
        C4["Git durumunu kontrol et (~1K)"]
        C5["Toplam: ~7-13K token<br/>harcanır"]
    end

    subgraph warm ["Devam Eden Oturum (Warm)"]
        direction TB
        W1["Mevcut bağlam korunur"]
        W2["Ek yükleme yok"]
        W3["Hemen çalışmaya<br/>devam edilir"]
    end

    cold -.->|"--continue ile<br/>önlenebilir"| warm

    style C5 fill:#E74C3C,color:#fff
    style W3 fill:#27AE60,color:#fff
```

| Senaryo | Token Maliyeti | Açıklama |
|---------|---------------|----------|
| **Yeni oturum** | ~7-13K token | CLAUDE.md, memory, proje taraması |
| **--continue** | ~2-3K token | Sadece oturum özeti yüklenir |
| **--resume** | ~2-3K token | Belirli oturum bağlamı yüklenir |
| **Aynı oturumda devam** | 0 token | Bağlam zaten mevcut |

---

## Oturum Devam Yöntemleri

### 1. --continue / -c (En Son Oturuma Devam)

En son oturumdan kaldığınız yerden devam etmek için:

```bash
# En son oturuma devam et
$ claude --continue

# Kısa form
$ claude -c

# Devam ederken yeni bir görev verin
$ claude -c "Dün kaldığım auth modülündeki rate limiting'i tamamla"
```

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant CC as Claude Code
    participant H as Oturum Geçmişi

    Note over U: Dün akşam oturumu kapattı
    U->>CC: claude --continue
    CC->>H: En son oturum ID'sini bul
    H-->>CC: session_abc123 (dün 18:45)
    CC->>H: Oturum özetini yükle
    H-->>CC: "Auth modülünde rate limiting üzerinde çalışıldı..."
    CC-->>U: "Önceki oturumdan devam ediyorum.<br/>Auth modülünde rate limiting üzerinde çalışıyorduk."
    U->>CC: "Rate limiting'i tamamla"
    CC->>CC: Bağlamı kullanarak devam eder
```

### 2. --resume / -r (Belirli Oturumu Seç)

Birden fazla oturum arasından seçim yapmak için:

```bash
# Oturum listesinden seçim yap (interaktif)
$ claude --resume

# Kısa form
$ claude -r
```

Komut çalıştırıldığında oturum listesi gösterilir:

```
Recent sessions:
  1. [2 saat önce] Auth modülü rate limiting
  2. [5 saat önce] Product sayfa tasarımı
  3. [dün] Veritabanı migration'ları
  4. [2 gün önce] CI/CD pipeline kurulumu

Select a session (1-4):
```

### 3. --session-id (Script'lerde Kullanım)

Otomasyon ve script'lerde belirli bir oturum ID'si ile devam etmek için:

```bash
# Belirli bir oturum ID'si ile devam et
$ claude --session-id "abc123def456"

# CI/CD pipeline'ında kullanım
$ claude --session-id "$SESSION_ID" "Pipeline hatalarını kontrol et"
```

### 4. /resume (REPL İçinde)

Aktif bir oturumdayken başka bir oturuma geçmek için:

```bash
$ claude
> /resume
# Oturum listesi gösterilir ve seçim yapılır
```

---

## Yöntem Karşılaştırması

```mermaid
flowchart TD
    NEED["Oturuma devam<br/>etmem gerekiyor"] --> Q1{"Hangi oturum?"}

    Q1 -->|"En son"| CONTINUE["--continue / -c<br/>En hızlı yol"]
    Q1 -->|"Belirli bir oturum"| Q2{"Nasıl seçeceğim?"}
    Q1 -->|"Script'te kullanıyorum"| SESSION_ID["--session-id<br/>Programatik erişim"]

    Q2 -->|"Listeden seçeceğim"| RESUME["--resume / -r<br/>İnteraktif seçim"]
    Q2 -->|"ID biliyorum"| SESSION_ID

    Q2 -->|"Zaten oturumdayım"| REPL_RESUME["/resume<br/>REPL içi geçiş"]

    style CONTINUE fill:#27AE60,color:#fff
    style RESUME fill:#4A90D9,color:#fff
    style SESSION_ID fill:#9B59B6,color:#fff
    style REPL_RESUME fill:#F39C12,color:#fff
```

| Yöntem | Kullanım | Senaryo |
|--------|----------|---------|
| `--continue` / `-c` | `claude -c` | Günlük kullanım, en son kaldığınız yer |
| `--resume` / `-r` | `claude -r` | Birkaç gün önceki oturuma dönmek |
| `--session-id` | `claude --session-id "id"` | CI/CD, otomasyon, script'ler |
| `/resume` | REPL içinde `/resume` | Oturum içinde başka oturuma geçiş |

---

## Pratik Örnek 1: Günlük İş Akışı

```bash
# Sabah: Dünkü oturuma devam et
$ claude -c
> Dün nerede kalmıştık?

# Claude önceki bağlamı hatırlar:
# "Dün auth modülünde rate limiting middleware'i üzerinde
#  çalışıyorduk. Redis cache entegrasyonu kalmıştı."

> Redis cache entegrasyonunu tamamla

# ... çalışma ...

# Öğleden sonra: Farklı bir görev
$ claude
> Product listesi için sayfalama ekle

# ... çalışma ...

# Akşam: Sabahki auth oturumuna dön
$ claude -r
# Listeden "Auth modülü rate limiting" oturumunu seç
```

---

## Pratik Örnek 2: Script'te Oturum Yönetimi

```bash
#!/bin/bash
# deploy-check.sh — Deployment öncesi kontrol script'i

SESSION_ID="deploy-check-$(date +%Y%m%d)"

# İlk çalıştırma veya devam
claude --session-id "$SESSION_ID" "
  1. Tüm testleri çalıştır
  2. Lint hatalarını kontrol et
  3. Build'in başarılı olduğunu doğrula
  4. Sonuçları DEPLOY-CHECK.md'ye yaz
"

# Sonucu kontrol et
if [ -f "DEPLOY-CHECK.md" ]; then
  echo "Kontrol tamamlandı, rapor hazır."
else
  echo "Kontrol başarısız."
  exit 1
fi
```

---

## Pratik Örnek 3: Oturum Arası Bağlam Aktarımı

Oturumlar arası bağlam aktarmak için dosya kullanma stratejisi:

```bash
# Oturum 1: Plan oluştur
$ claude
> Bu projenin refactoring planını oluştur ve REFACTORING-PLAN.md'ye kaydet
> /exit

# Oturum 2: Planı uygula (temiz context window ile)
$ claude
> REFACTORING-PLAN.md'yi oku ve Adım 1'i uygula
> Tamamladığın adımları REFACTORING-PLAN.md'de işaretle
> /exit

# Oturum 3: Devam et
$ claude
> REFACTORING-PLAN.md'yi oku ve bir sonraki tamamlanmamış adımı uygula
```

```mermaid
flowchart LR
    S1["Oturum 1<br/>(Plan oluştur)"] -->|"PLAN.md<br/>yazılır"| FILE["📄 PLAN.md<br/>(Oturumlar arası<br/>bağlam köprüsü)"]
    FILE -->|"PLAN.md<br/>okunur"| S2["Oturum 2<br/>(Adım 1 uygula)"]
    S2 -->|"PLAN.md<br/>güncellenir"| FILE
    FILE -->|"PLAN.md<br/>okunur"| S3["Oturum 3<br/>(Adım 2 uygula)"]

    style FILE fill:#F39C12,color:#fff
```

---

## Oturum Geçmişi

Claude Code tüm oturum geçmişini yerel olarak saklar:

```bash
# Oturum geçmişi konumu
~/.claude/sessions/

# Son oturumları listele (--resume ile)
$ claude --resume
```

### Oturum Geçmişinde Saklanan Bilgiler

| Bilgi | Açıklama |
|-------|----------|
| **Oturum ID** | Benzersiz tanımlayıcı |
| **Başlangıç zamanı** | Oturumun ne zaman başladığı |
| **Konuşma geçmişi** | Tüm mesajlar ve araç çıktıları |
| **Çalışma dizini** | Oturumun hangi projede çalıştığı |
| **Oturum özeti** | Compact sonrası oluşan özet |

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Session** | Claude Code'un tek bir çalışma birimi |
| **Cold start tax** | Yeni oturumda bağlam yeniden oluşturma maliyeti (~7-13K token) |
| **--continue / -c** | En son oturuma devam etmenin en hızlı yolu |
| **--resume / -r** | Belirli bir oturumu seçerek devam etme |
| **--session-id** | Programatik erişim için oturum ID'si belirtme |
| **/resume** | REPL içinde oturum geçişi |
| **Dosya ile bağlam aktarımı** | Plan dosyaları aracılığıyla oturumlar arası bağlam köprüsü |

---

## Sonraki Adım

Oturumlar arası geçişin ötesinde, bir oturum içindeki değişiklikleri izlemek ve geri almak için checkpointing mekanizmasını inceleyelim:

→ [Checkpointing](./07-checkpointing.md)
