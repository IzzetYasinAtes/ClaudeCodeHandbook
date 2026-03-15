# Hooks Nedir?

**Hooks** (kancalar), Claude Code'un yaşam döngüsündeki belirli anlarda otomatik olarak tetiklenen, kullanıcı tanımlı eylemlerdir. Event-driven automation (olay tabanlı otomasyon) prensibiyle çalışarak tekrarlayan görevleri ortadan kaldırır, kuralları zorunlu kılar ve dış sistemlerle entegrasyon sağlar.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code araçları | [Araçlara Genel Bakış](../08-araclar/01-araclara-genel-bakis.md) |
| İzin sistemi | [İzin Sistemi](../10-izinler-ve-guvenlik/01-izin-sistemi.md) |
| JSON temel bilgisi | Harici kaynak |

---

## Hook Kavramı

Hook, "bir olay gerçekleştiğinde otomatik olarak bir şey yap" demektir. Claude Code'da hook'lar üç temel bileşenden oluşur:

```mermaid
flowchart LR
    subgraph hook ["Hook Bileşenleri"]
        direction TB
        EVENT["🎯 Event (Olay)\nNe zaman tetiklensin?"]
        MATCHER["🔍 Matcher (Eşleştirici)\nHangi araç için? (opsiyonel)"]
        ACTION["⚡ Hook Action (Eylem)\nNe yapılsın?"]
    end

    EVENT --> MATCHER --> ACTION

    subgraph ornekler ["Örnek"]
        direction TB
        E1["PostToolUse\n(Araç kullanıldıktan sonra)"]
        M1["tool_name == 'Edit'\n(Sadece Edit aracı için)"]
        A1["prettier --write $file\n(Dosyayı formatla)"]
    end

    E1 --> M1 --> A1

    style EVENT fill:#3498DB,color:#fff
    style MATCHER fill:#F39C12,color:#fff
    style ACTION fill:#27AE60,color:#fff
```

### Üç Bileşen Detaylı

| Bileşen | Zorunlu mu? | Açıklama | Örnek |
|---------|-------------|----------|-------|
| **Event** (Olay) | ✅ Evet | Hook'un ne zaman tetikleneceğini belirler | `PreToolUse`, `SessionStart` |
| **Matcher** (Eşleştirici) | ❌ Hayır | Hangi araç veya koşul için geçerli olduğunu filtreler | `tool_name: "Bash"` |
| **Hook Action** (Eylem) | ✅ Evet | Tetiklendiğinde ne yapılacağını tanımlar | Shell komutu, HTTP çağrısı |

---

## Neden Hook Kullanmalı?

Hooks, Claude Code deneyiminizi dört ana alanda güçlendirir:

```mermaid
mindmap
  root(("Hooks\nKullanım\nAlanları"))
    Otomasyon
      Dosya formatlama
      Test çalıştırma
      Lint kontrolü
      Dependency güncelleme
    Kural Zorlama
      Tehlikeli komut engelleme
      Kod standartları
      Branch koruma
      Dosya boyut limiti
    Entegrasyon
      Slack bildirimi
      Jira güncelleme
      Webhook tetikleme
      Loglama
    İş Akışı
      Otomatik onay
      Ortam doğrulama
      Checkpoint oluşturma
      Metrik toplama
```

### Somut Faydalar

| Senaryo | Hook Olmadan | Hook İle |
|---------|-------------|----------|
| Dosya formatı | Her düzenlemeden sonra `prettier` çalıştır demeniz gerekir | Otomatik formatlanır |
| Tehlikeli komut | Claude `rm -rf /` çalıştırabilir | Hook engeller, uyarı verir |
| Bildirim | Oturum sonunu manuel takip edersiniz | Slack'e otomatik mesaj gider |
| Test | "Testleri çalıştır" demeniz gerekir | Her kod değişikliğinde otomatik çalışır |

---

## Hook Yaşam Döngüsü

Bir hook tetiklendiğinde şu adımlar sırayla gerçekleşir:

```mermaid
sequenceDiagram
    participant CC as Claude Code
    participant HS as Hook System
    participant HK as Hook Action
    participant EXT as Dış Sistem

    CC->>HS: Olay tetiklendi (ör: PostToolUse)
    HS->>HS: Matcher kontrolü (araç adı eşleşiyor mu?)
    
    alt Matcher eşleşmedi
        HS-->>CC: Hook atlandı
    else Matcher eşleşti veya yok
        HS->>HK: Hook action çalıştır
        
        alt Command Hook
            HK->>HK: Shell komutu çalıştır
            HK-->>HS: Exit code döndür
        else HTTP Hook
            HK->>EXT: HTTP isteği gönder
            EXT-->>HK: Yanıt döndür
            HK-->>HS: Sonuç döndür
        end
        
        alt Exit code == 0
            HS-->>CC: ✅ Başarılı, devam et
        else Exit code != 0
            HS-->>CC: ❌ Engellendi, işlemi durdur
        end
    end
```

### Yaşam Döngüsü Adımları

1. **Olay Tetiklenir** — Claude Code belirli bir yaşam döngüsü noktasına ulaşır
2. **Matcher Kontrolü** — Tanımlı matcher varsa, koşulun sağlanıp sağlanmadığı kontrol edilir
3. **Hook Çalıştırılır** — Koşullar sağlanırsa, hook action yürütülür
4. **Sonuç Değerlendirilir** — Exit code'a göre işlem devam eder veya engellenir
5. **Çıktı İşlenir** — Hook'un stdout çıktısı Claude'un bağlamına eklenebilir

---

## Temel Hook Yapısı

Hook'lar `settings.json` dosyasında tanımlanır. İşte en basit yapı:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

Bu örnekte:
- **Event:** `PostToolUse` — Bir araç kullanıldıktan sonra
- **Matcher:** `"Edit"` — Sadece Edit aracı için
- **Action:** `prettier --write` komutu — Düzenlenen dosyayı formatla

---

## Pratik Örnek 1: İlk Hook'unuz

Dosya düzenlemelerinden sonra otomatik formatlama yapan basit bir hook:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

Bu hook şu şekilde çalışır:

```mermaid
flowchart LR
    A["Claude Edit aracını\nkullanır"] --> B["PostToolUse\nolayı tetiklenir"]
    B --> C{"Matcher:\nAraç == Edit?"}
    C -->|Evet| D["prettier --write\nçalıştırılır"]
    C -->|Hayır| E["Hook atlanır"]
    D --> F["Dosya\nformatlandı ✅"]

    style A fill:#3498DB,color:#fff
    style D fill:#27AE60,color:#fff
    style F fill:#27AE60,color:#fff
```

---

## Pratik Örnek 2: Tehlikeli Komut Engelleme

`rm -rf /` gibi tehlikeli komutları engelleyen bir güvenlik hook'u:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python -c \"import sys,json; cmd=json.load(sys.stdin).get('command',''); sys.exit(1 if any(d in cmd for d in ['rm -rf /','rm -rf ~','mkfs.','dd if=']) else 0)\""
          }
        ]
      }
    ]
  }
}
```

Bu hook'ta exit code davranışı kritiktir:
- **Exit code 0:** Komut güvenli, devam et
- **Exit code 1:** Tehlikeli komut tespit edildi, engelle!

---

## Pratik Örnek 3: Oturum Başlangıç Doğrulaması

Oturum başladığında gerekli araçların kurulu olduğunu doğrulayan bir hook:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node --version && npm --version && git --version && echo 'Tüm araçlar mevcut' || echo 'UYARI: Bazı araçlar eksik!'"
          }
        ]
      }
    ]
  }
}
```

---

## Hook vs Manuel Yaklaşım Karşılaştırması

```mermaid
flowchart TB
    subgraph manuel ["❌ Manuel Yaklaşım"]
        direction TB
        M1["Claude dosya düzenler"]
        M2["'Şimdi prettier çalıştır' dersiniz"]
        M3["Claude prettier çalıştırır"]
        M4["'Şimdi testleri çalıştır' dersiniz"]
        M5["Claude testleri çalıştırır"]
        M1 --> M2 --> M3 --> M4 --> M5
    end

    subgraph hooks ["✅ Hook Yaklaşımı"]
        direction TB
        H1["Claude dosya düzenler"]
        H2["PostToolUse hook:\nprettier otomatik çalışır"]
        H3["PostToolUse hook:\ntestler otomatik çalışır"]
        H1 --> H2
        H1 --> H3
    end

    style manuel fill:#FADBD8,stroke:#E74C3C
    style hooks fill:#D5F5E3,stroke:#27AE60
```

---

## Sık Yapılan Hatalar

| Hata | Doğrusu |
|------|---------|
| Matcher'ı büyük/küçük harf yanlış yazmak | Araç adları büyük harfle başlar: `"Edit"`, `"Bash"` |
| Exit code davranışını anlamamak | 0 = başarı/izin ver, non-zero = engelle |
| Hook'ta sonsuz döngü oluşturmak | Hook'un tetiklediği işlem yeni hook tetiklemesin |
| Asenkron hook'u senkron gibi beklemek | `async: true` ayarını kontrol edin |
| Tüm hook'ları tek bir event'e yığmak | İlgili event'lere dağıtın |

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Hook** | Yaşam döngüsü olaylarında tetiklenen kullanıcı tanımlı eylem |
| **Event** | Hook'un ne zaman tetikleneceğini belirler |
| **Matcher** | Hangi araç/koşul için geçerli olduğunu filtreler |
| **Hook Action** | Tetiklendiğinde çalıştırılacak komut veya istek |
| **Exit Code** | 0 = başarı, non-zero = engelleme |
| **Konfigürasyon** | `settings.json` dosyasında `hooks` objesi altında |

---

## Sonraki Adım

Hook'ların hangi olaylarda tetiklenebileceğini ve her olayda hangi verilere erişilebildiğini detaylı inceleyelim:

→ [Hook Olayları](./02-hook-olaylari.md)
