# Hook Örnekleri ve Tarifler

Bu bölüm, gerçek dünya senaryolarına yönelik hazır hook tarifleri (recipes) sunar. Her tarif, problemi, çözümü, tam JSON konfigürasyonunu ve çalışma akışını içerir.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Hook kavramı | [Hooks Nedir?](./01-hooks-nedir.md) |
| Hook olayları | [Hook Olayları](./02-hook-olaylari.md) |
| Hook tipleri | [Hook Tipleri](./03-hook-tipleri.md) |
| Hook konfigürasyonu | [Hook Konfigürasyonu](./04-hook-konfigurasyonu.md) |

---

## Tarif Kataloğu

```mermaid
flowchart TD
    subgraph recipes ["Hook Tarifleri"]
        direction TB
        R1["🎨 Tarif 1\nOtomatik Formatlama"]
        R2["🛡️ Tarif 2\nTehlikeli Komut Engelleme"]
        R3["💬 Tarif 3\nSlack Bildirimi"]
        R4["✅ Tarif 4\nOtomatik İzin Onaylama"]
        R5["📏 Tarif 5\nProje Kuralı Zorlama"]
        R6["📝 Tarif 6\nAraç Kullanım Logu"]
        R7["🧪 Tarif 7\nOtomatik Test Çalıştırma"]
        R8["🔍 Tarif 8\nOrtam Doğrulama"]
        R9["📊 Tarif 9\nMetrik ve Telemetri"]
        R10["🔐 Tarif 10\nGizli Bilgi Sızıntı Koruması"]
    end

    style R1 fill:#3498DB,color:#fff
    style R2 fill:#E74C3C,color:#fff
    style R3 fill:#1ABC9C,color:#fff
    style R4 fill:#27AE60,color:#fff
    style R5 fill:#9B59B6,color:#fff
    style R6 fill:#F39C12,color:#fff
    style R7 fill:#2ECC71,color:#fff
    style R8 fill:#E67E22,color:#fff
    style R9 fill:#3498DB,color:#fff
    style R10 fill:#E74C3C,color:#fff
```

---

## Tarif 1: Otomatik Dosya Formatlama

**Problem:** Claude her dosya düzenlediğinde veya oluşturduğunda, projenin kod stili bozulabiliyor. Her seferinde manuel formatlama hatırlanması gerekiyor.

**Çözüm:** `PostToolUse` olayında `Edit` ve `Write` araçlarını yakalayarak otomatik formatlama.

### Akış

```mermaid
flowchart LR
    A["Claude dosya\ndüzenler"] --> B["PostToolUse\ntetiklenir"]
    B --> C{"Dosya uzantısı?"}
    C -->|".ts/.tsx/.js"| D["Prettier"]
    C -->|".py"| E["Black"]
    C -->|".go"| F["gofmt"]
    C -->|Diğer| G["Atla"]
    D --> H["✅ Formatlandı"]
    E --> H
    F --> H

    style H fill:#27AE60,color:#fff
```

### Konfigürasyon

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "EXT=\"${CLAUDE_FILE_PATH##*.}\"; case \"$EXT\" in ts|tsx|js|jsx|json|css|scss|md) npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; py) black \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; go) gofmt -w \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; rs) rustfmt \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; esac",
            "timeout_ms": 15000
          }
        ]
      },
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "EXT=\"${CLAUDE_FILE_PATH##*.}\"; case \"$EXT\" in ts|tsx|js|jsx|json|css|scss|md) npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; py) black \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; go) gofmt -w \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; rs) rustfmt \"$CLAUDE_FILE_PATH\" 2>/dev/null ;; esac",
            "timeout_ms": 15000
          }
        ]
      }
    ]
  }
}
```

### Basit Versiyon (Sadece Prettier)

Projeniz yalnızca JavaScript/TypeScript kullanıyorsa:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## Tarif 2: Tehlikeli Komut Engelleme

**Problem:** Claude, `rm -rf /`, `DROP TABLE`, `git push --force` gibi tehlikeli komutları çalıştırabilir.

**Çözüm:** `PreToolUse` olayında `Bash` aracını yakalayarak komut içeriğini analiz etme ve tehlikeli komutları engelleme.

### Akış

```mermaid
flowchart TD
    A["Claude Bash komutu\nçalıştırmak istiyor"] --> B["PreToolUse\ntetiklenir"]
    B --> C["Komut içeriği\nanaliz edilir"]
    C --> D{"Tehlikeli\nkalıp var mı?"}
    D -->|"rm -rf /, DROP TABLE,\ngit push --force, mkfs..."| E["❌ Exit 1\nKomut engellendi"]
    D -->|"Güvenli"| F["✅ Exit 0\nKomut çalışabilir"]

    style E fill:#E74C3C,color:#fff
    style F fill:#27AE60,color:#fff
```

### Konfigürasyon

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"\nimport sys, json\n\ncmd = json.load(sys.stdin).get('command', '')\n\ndangerous_patterns = [\n    'rm -rf /',\n    'rm -rf ~',\n    'rm -rf .',\n    'rm -rf *',\n    'mkfs.',\n    'dd if=',\n    ':(){:|:&};:',\n    'chmod -R 777 /',\n    'chown -R',\n    '> /dev/sda',\n    'DROP TABLE',\n    'DROP DATABASE',\n    'TRUNCATE TABLE',\n    'DELETE FROM',\n    'git push --force origin main',\n    'git push -f origin main',\n    'git reset --hard origin',\n    'shutdown',\n    'reboot',\n    'init 0',\n    'FORMAT C:'\n]\n\ncmd_upper = cmd.upper()\nfor pattern in dangerous_patterns:\n    if pattern.upper() in cmd_upper:\n        print(f'ENGELLENDI: Tehlikeli komut kalıbı tespit edildi: {pattern}')\n        print(f'Komut: {cmd[:200]}')\n        sys.exit(1)\n\nprint('Komut güvenli görünüyor.')\nsys.exit(0)\n\""
          }
        ]
      }
    ]
  }
}
```

### Hafif Versiyon (Bash ile)

Python yüklü olmayan ortamlar için bash-only versiyon:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "CMD=$(echo \"$CLAUDE_TOOL_INPUT\" | grep -oP '\"command\"\\s*:\\s*\"\\K[^\"]+'); for PATTERN in 'rm -rf /' 'rm -rf ~' 'DROP TABLE' 'DROP DATABASE' 'git push --force' 'mkfs.' 'dd if='; do if echo \"$CMD\" | grep -qi \"$PATTERN\"; then echo \"ENGELLENDI: $PATTERN tespit edildi\"; exit 1; fi; done; echo 'Güvenli'; exit 0"
          }
        ]
      }
    ]
  }
}
```

---

## Tarif 3: Slack Bildirimi Gönderme

**Problem:** Claude Code oturumu bittiğinde ekip üyelerinin haberdar olması gerekiyor.

**Çözüm:** `SessionEnd` olayında Slack webhook'una HTTP isteği gönderme.

### Akış

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant CC as Claude Code
    participant HK as Hook System
    participant SL as Slack API

    U->>CC: /exit
    CC->>HK: SessionEnd olayı
    HK->>SL: POST /services/webhook
    SL-->>HK: 200 OK
    Note over SL: #dev-notifications kanalına<br/>mesaj gönderildi
    HK-->>CC: Başarılı
    CC-->>U: Oturum kapatıldı
```

### Konfigürasyon (HTTP Hook)

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "https://hooks.slack.com/services/T00000000/B00000000/xxxxxxxxxxxxxxxxxxxxxxxx",
            "method": "POST",
            "headers": {
              "Content-Type": "application/json"
            },
            "body": "{\"blocks\":[{\"type\":\"section\",\"text\":{\"type\":\"mrkdwn\",\"text\":\"*Claude Code Oturum Özeti*\\n• Proje: `{{working_directory}}`\\n• Süre: {{duration_seconds}} saniye\\n• Zaman: {{timestamp}}\"}}]}",
            "async": true
          }
        ]
      }
    ]
  }
}
```

### Konfigürasyon (Command Hook ile curl)

Slack webhook URL'sini ortam değişkeninden alan versiyon:

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "curl -s -X POST \"$SLACK_WEBHOOK_URL\" -H 'Content-Type: application/json' -d \"{\\\"text\\\": \\\"🤖 Claude Code oturumu sona erdi.\\\\nProje: $(basename $(pwd))\\\\nSüre: ${CLAUDE_SESSION_DURATION:-bilinmiyor}s\\\\nZaman: $(date '+%Y-%m-%d %H:%M:%S')\\\"}\"",
            "timeout_ms": 10000,
            "async": true
          }
        ]
      }
    ]
  }
}
```

---

## Tarif 4: Güvenli İşlemleri Otomatik Onaylama

**Problem:** Claude her okuma veya güvenli komut için izin soruyor, bu iş akışını yavaşlatıyor.

**Çözüm:** `PermissionRequest` olayında güvenli işlemleri otomatik onaylama, tehlikeli işlemleri otomatik reddetme.

### Akış

```mermaid
flowchart TD
    A["Claude bir araç\nkullanmak istiyor"] --> B["PermissionRequest\ntetiklenir"]
    B --> C{"Araç ve komut\nanalizi"}
    
    C -->|"Read, Glob, Grep"| D["Exit 0\n✅ Otomatik onayla"]
    C -->|"Bash: ls, pwd, git status,\necho, cat, head"| E["Exit 0\n✅ Otomatik onayla"]
    C -->|"Bash: rm, mv, chmod,\ngit push, npm publish"| F["Exit 1\n❌ Otomatik reddet"]
    C -->|"Diğer"| G["Exit 2\n❓ Kullanıcıya sor"]

    style D fill:#27AE60,color:#fff
    style E fill:#27AE60,color:#fff
    style F fill:#E74C3C,color:#fff
    style G fill:#F39C12,color:#fff
```

### Konfigürasyon

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "exit 0"
          }
        ]
      },
      {
        "matcher": "Glob",
        "hooks": [
          {
            "type": "command",
            "command": "exit 0"
          }
        ]
      },
      {
        "matcher": "Grep",
        "hooks": [
          {
            "type": "command",
            "command": "exit 0"
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"\nimport sys, json\n\ncmd = json.load(sys.stdin).get('command', '')\n\nsafe_prefixes = [\n    'ls', 'pwd', 'echo', 'cat ', 'head ', 'tail ',\n    'wc ', 'grep ', 'find ', 'which ', 'whoami',\n    'date', 'uname', 'env', 'printenv',\n    'git status', 'git log', 'git diff', 'git branch',\n    'node --version', 'npm --version', 'python --version',\n    'npm list', 'npm outdated', 'npm audit'\n]\n\ndangerous_patterns = [\n    'rm ', 'mv ', 'chmod ', 'chown ',\n    'git push', 'git reset', 'npm publish',\n    'sudo ', 'curl.*|.*sh', 'wget.*|.*sh'\n]\n\nif any(cmd.strip().startswith(p) for p in safe_prefixes):\n    sys.exit(0)\n\nif any(p in cmd for p in dangerous_patterns):\n    sys.exit(1)\n\nsys.exit(2)\n\""
          }
        ]
      }
    ]
  }
}
```

---

## Tarif 5: Proje Kurallarını Zorlama

**Problem:** Kullanıcılar proje standartlarına uymayan isteklerde bulunabiliyor (ör: "JavaScript ile yaz" ama proje TypeScript).

**Çözüm:** `UserPromptSubmit` olayında prompt'u analiz ederek proje kurallarına uygunluk kontrolü.

### Konfigürasyon

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_PROMPT\" | python3 -c \"\nimport sys\n\nprompt = sys.stdin.read().lower()\n\nrules = {\n    'javascript': 'Bu proje TypeScript kullanıyor. JavaScript yerine TypeScript tercih edin.',\n    '.js dosya': 'Bu proje TypeScript kullanıyor. .js yerine .ts/.tsx uzantısı kullanın.',\n    'var ': 'Bu projede var kullanılmaz. const veya let tercih edin.',\n    'class component': 'Bu projede class component kullanılmaz. Function component tercih edin.',\n    'jquery': 'Bu projede jQuery kullanılmaz. React kullanın.',\n    'css module': 'Bu projede CSS Modules kullanılmaz. Tailwind CSS kullanın.',\n    'axios': 'Bu projede axios kullanılmaz. fetch API kullanın.'\n}\n\nwarnings = []\nfor keyword, message in rules.items():\n    if keyword in prompt:\n        warnings.append(f'⚠️ {message}')\n\nif warnings:\n    print('PROJE KURALI UYARISI:')\n    for w in warnings:\n        print(w)\n    print('\\nDevam etmek istiyorsanız kuralları dikkate alarak ilerleyin.')\n\nsys.exit(0)\n\""
          }
        ]
      }
    ]
  }
}
```

> **Not:** Bu tarif `exit 0` döndürür ve engelleme yapmaz — sadece uyarı verir. Zorlama yapmak isterseniz `sys.exit(1)` kullanabilirsiniz.

---

## Tarif 6: Tüm Araç Kullanımını Loglama

**Problem:** Claude'un oturum boyunca hangi araçları, ne sıklıkla ve hangi dosyalar üzerinde kullandığını takip etmek istiyorsunuz.

**Çözüm:** `PostToolUse` ve `PostToolUseFailure` olaylarında kapsamlı loglama.

### Akış

```mermaid
flowchart LR
    A["Araç kullanıldı"] --> B{"Başarılı mı?"}
    B -->|Evet| C["PostToolUse\n→ changes.log"]
    B -->|Hayır| D["PostToolUseFailure\n→ errors.log"]
    C --> E["📊 Analiz için\nhazır loglar"]
    D --> E

    style C fill:#27AE60,color:#fff
    style D fill:#E74C3C,color:#fff
```

### Konfigürasyon

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "LOG_DIR=\"$HOME/.claude/logs\"; mkdir -p \"$LOG_DIR\"; echo \"$(date -u '+%Y-%m-%dT%H:%M:%SZ') | OK | $CLAUDE_TOOL_NAME | ${CLAUDE_FILE_PATH:-N/A} | $(echo $CLAUDE_TOOL_INPUT | head -c 200)\" >> \"$LOG_DIR/tool-usage.log\"",
            "async": true
          }
        ]
      }
    ],
    "PostToolUseFailure": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "LOG_DIR=\"$HOME/.claude/logs\"; mkdir -p \"$LOG_DIR\"; echo \"$(date -u '+%Y-%m-%dT%H:%M:%SZ') | FAIL | $CLAUDE_TOOL_NAME | ${CLAUDE_FILE_PATH:-N/A} | $(echo $CLAUDE_TOOL_INPUT | head -c 200)\" >> \"$LOG_DIR/tool-usage.log\"",
            "async": true
          }
        ]
      }
    ]
  }
}
```

### Log Çıktı Örneği

```
2026-03-15T10:05:00Z | OK   | Read  | src/app.ts        | {"file_path":"src/app.ts"}
2026-03-15T10:05:15Z | OK   | Edit  | src/app.ts        | {"file_path":"src/app.ts","old_string":"...
2026-03-15T10:05:20Z | OK   | Edit  | src/app.ts        | {"file_path":"src/app.ts","old_string":"...
2026-03-15T10:05:30Z | FAIL | Bash  | N/A               | {"command":"npm test"}
2026-03-15T10:06:00Z | OK   | Edit  | src/app.test.ts   | {"file_path":"src/app.test.ts","old_stri...
2026-03-15T10:06:15Z | OK   | Bash  | N/A               | {"command":"npm test"}
```

### Gelişmiş: JSON Formatında Loglama

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "LOG_DIR=\"$HOME/.claude/logs\"; mkdir -p \"$LOG_DIR\"; echo \"{\\\"timestamp\\\":\\\"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\\\",\\\"status\\\":\\\"success\\\",\\\"tool\\\":\\\"$CLAUDE_TOOL_NAME\\\",\\\"file\\\":\\\"${CLAUDE_FILE_PATH:-null}\\\",\\\"session\\\":\\\"$CLAUDE_SESSION_ID\\\"}\" >> \"$LOG_DIR/tool-usage.jsonl\"",
            "async": true
          }
        ]
      }
    ]
  }
}
```

---

## Tarif 7: Kod Değişikliğinden Sonra Otomatik Test

**Problem:** Claude bir dosyayı düzenledikten sonra ilgili testlerin çalıştırılmasını unutabiliyor veya siz hatırlatmayı unutuyorsunuz.

**Çözüm:** `PostToolUse` olayında `Edit` aracını yakalayarak ilgili test dosyasını otomatik çalıştırma.

### Akış

```mermaid
flowchart TD
    A["Claude\nsrc/utils/math.ts\ndosyasını düzenler"] --> B["PostToolUse tetiklenir"]
    B --> C{"Test dosyası\nvar mı?"}
    C -->|"src/utils/math.test.ts\nmevcut"| D["npm test -- math.test.ts"]
    C -->|"Test dosyası\nbulunamadı"| E["Atla"]
    D --> F{"Test\nsonucu?"}
    F -->|"✅ Geçti"| G["Claude'a bildir:\n'Testler geçti'"]
    F -->|"❌ Kaldı"| H["Claude'a bildir:\n'Testler kaldı, düzelt'"]

    style G fill:#27AE60,color:#fff
    style H fill:#E74C3C,color:#fff
```

### Konfigürasyon

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATH\" == *.test.* ]] || [[ \"$CLAUDE_FILE_PATH\" == *__test__* ]] || [[ \"$CLAUDE_FILE_PATH\" == *.spec.* ]]; then exit 0; fi; BASENAME=$(basename \"$CLAUDE_FILE_PATH\" | sed 's/\\.[^.]*$//'); EXT=\"${CLAUDE_FILE_PATH##*.}\"; TEST_FILE=\"\"; for SUFFIX in \".test.$EXT\" \".spec.$EXT\" \"_test.$EXT\"; do DIR=$(dirname \"$CLAUDE_FILE_PATH\"); CANDIDATE=\"$DIR/${BASENAME}${SUFFIX}\"; if [ -f \"$CANDIDATE\" ]; then TEST_FILE=\"$CANDIDATE\"; break; fi; CANDIDATE=\"$DIR/__tests__/${BASENAME}${SUFFIX}\"; if [ -f \"$CANDIDATE\" ]; then TEST_FILE=\"$CANDIDATE\"; break; fi; done; if [ -n \"$TEST_FILE\" ]; then echo \"İlgili test çalıştırılıyor: $TEST_FILE\"; npx jest \"$TEST_FILE\" --no-coverage 2>&1 | tail -20; else echo 'İlgili test dosyası bulunamadı.'; fi",
            "timeout_ms": 60000
          }
        ]
      }
    ]
  }
}
```

### Basit Versiyon (Tüm Testleri Çalıştır)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATH\" == *.ts || \"$CLAUDE_FILE_PATH\" == *.tsx ]]; then npx jest --bail --no-coverage 2>&1 | tail -10; fi",
            "timeout_ms": 120000
          }
        ]
      }
    ]
  }
}
```

---

## Tarif 8: Oturum Başlangıcında Ortam Doğrulama

**Problem:** Claude Code oturumu başlatıldığında gerekli araçlar eksik olabilir, yanlış dizinde olunabilir veya servisler çalışmıyor olabilir.

**Çözüm:** `SessionStart` olayında kapsamlı ortam kontrolü.

### Akış

```mermaid
flowchart TD
    A["Oturum\nbaşlatıldı"] --> B["SessionStart\ntetiklenir"]
    B --> C["Araç kontrolleri"]
    C --> D{"Node.js?"}
    D -->|✅| E{"npm?"}
    D -->|❌| WARN["⚠️ Uyarı"]
    E -->|✅| F{"Git?"}
    E -->|❌| WARN
    F -->|✅| G{"package.json?"}
    F -->|❌| WARN
    G -->|✅| H{"node_modules?"}
    G -->|❌| WARN
    H -->|✅| OK["✅ Ortam hazır"]
    H -->|❌| INSTALL["npm install\nçalıştır"]
    INSTALL --> OK

    style OK fill:#27AE60,color:#fff
    style WARN fill:#F39C12,color:#fff
```

### Konfigürasyon

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\nERRORS=0\nWARNINGS=0\necho '=== Ortam Doğrulama ==='\n\n# Node.js kontrolü\nif command -v node &>/dev/null; then\n  NODE_VER=$(node --version)\n  echo \"✅ Node.js: $NODE_VER\"\n  NODE_MAJOR=$(echo $NODE_VER | cut -d. -f1 | tr -d 'v')\n  if [ \"$NODE_MAJOR\" -lt 18 ]; then\n    echo \"⚠️ Node.js 18+ önerilir (mevcut: $NODE_VER)\"\n    WARNINGS=$((WARNINGS+1))\n  fi\nelse\n  echo '❌ Node.js bulunamadı!'\n  ERRORS=$((ERRORS+1))\nfi\n\n# npm kontrolü\nif command -v npm &>/dev/null; then\n  echo \"✅ npm: $(npm --version)\"\nelse\n  echo '❌ npm bulunamadı!'\n  ERRORS=$((ERRORS+1))\nfi\n\n# Git kontrolü\nif command -v git &>/dev/null; then\n  echo \"✅ Git: $(git --version | cut -d' ' -f3)\"\nelse\n  echo '❌ Git bulunamadı!'\n  ERRORS=$((ERRORS+1))\nfi\n\n# package.json kontrolü\nif [ -f 'package.json' ]; then\n  echo '✅ package.json mevcut'\nelse\n  echo '⚠️ package.json bulunamadı'\n  WARNINGS=$((WARNINGS+1))\nfi\n\n# node_modules kontrolü\nif [ -d 'node_modules' ]; then\n  echo '✅ node_modules mevcut'\nelse\n  echo '⚠️ node_modules bulunamadı — npm install çalıştırılıyor...'\n  npm install --silent 2>/dev/null\n  WARNINGS=$((WARNINGS+1))\nfi\n\n# .env kontrolü\nif [ -f '.env' ]; then\n  echo '✅ .env dosyası mevcut'\nelse\n  if [ -f '.env.example' ]; then\n    echo '⚠️ .env dosyası yok ama .env.example mevcut'\n    WARNINGS=$((WARNINGS+1))\n  fi\nfi\n\necho \"\\n=== Sonuç: $ERRORS hata, $WARNINGS uyarı ===\"\nif [ $ERRORS -gt 0 ]; then\n  echo '⛔ Kritik eksiklikler var!'\nelse\n  echo '✅ Ortam kullanıma hazır.'\nfi",
            "timeout_ms": 60000
          }
        ]
      }
    ]
  }
}
```

### Minimal Versiyon

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Node: $(node -v 2>/dev/null || echo 'YOK')\" && echo \"npm: $(npm -v 2>/dev/null || echo 'YOK')\" && echo \"Git: $(git --version 2>/dev/null | cut -d' ' -f3 || echo 'YOK')\" && echo \"Dizin: $(pwd)\" && echo \"Branch: $(git branch --show-current 2>/dev/null || echo 'N/A')\""
          }
        ]
      }
    ]
  }
}
```

---

## Tarif 9: Metrik ve Telemetri Toplama

**Problem:** Claude Code kullanımını analiz etmek, hangi araçların ne sıklıkla kullanıldığını ve oturum istatistiklerini görmek istiyorsunuz.

**Çözüm:** Birden fazla olay noktasında veri toplayarak JSON formatında metrik dosyası oluşturma.

### Konfigürasyon

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "METRICS_DIR=\"$HOME/.claude/metrics\"; mkdir -p \"$METRICS_DIR\"; echo \"{\\\"event\\\":\\\"session_start\\\",\\\"session\\\":\\\"$CLAUDE_SESSION_ID\\\",\\\"cwd\\\":\\\"$(pwd)\\\",\\\"timestamp\\\":\\\"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\\\"}\" >> \"$METRICS_DIR/$(date '+%Y-%m-%d').jsonl\"",
            "async": true
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "METRICS_DIR=\"$HOME/.claude/metrics\"; echo \"{\\\"event\\\":\\\"tool_use\\\",\\\"tool\\\":\\\"$CLAUDE_TOOL_NAME\\\",\\\"status\\\":\\\"success\\\",\\\"file\\\":\\\"${CLAUDE_FILE_PATH:-null}\\\",\\\"session\\\":\\\"$CLAUDE_SESSION_ID\\\",\\\"timestamp\\\":\\\"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\\\"}\" >> \"$METRICS_DIR/$(date '+%Y-%m-%d').jsonl\"",
            "async": true
          }
        ]
      }
    ],
    "PostToolUseFailure": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "METRICS_DIR=\"$HOME/.claude/metrics\"; echo \"{\\\"event\\\":\\\"tool_use\\\",\\\"tool\\\":\\\"$CLAUDE_TOOL_NAME\\\",\\\"status\\\":\\\"failure\\\",\\\"file\\\":\\\"${CLAUDE_FILE_PATH:-null}\\\",\\\"session\\\":\\\"$CLAUDE_SESSION_ID\\\",\\\"timestamp\\\":\\\"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\\\"}\" >> \"$METRICS_DIR/$(date '+%Y-%m-%d').jsonl\"",
            "async": true
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "METRICS_DIR=\"$HOME/.claude/metrics\"; echo \"{\\\"event\\\":\\\"session_end\\\",\\\"session\\\":\\\"$CLAUDE_SESSION_ID\\\",\\\"duration\\\":\\\"${CLAUDE_SESSION_DURATION:-0}\\\",\\\"timestamp\\\":\\\"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\\\"}\" >> \"$METRICS_DIR/$(date '+%Y-%m-%d').jsonl\"",
            "async": true
          }
        ]
      }
    ]
  }
}
```

### Metrikleri Analiz Etme

Toplanan metrikleri analiz etmek için basit bir komut:

```bash
# Bugünkü araç kullanım sayıları
cat ~/.claude/metrics/$(date '+%Y-%m-%d').jsonl | \
  python3 -c "
import sys, json
from collections import Counter
tools = Counter()
for line in sys.stdin:
    data = json.loads(line)
    if data.get('event') == 'tool_use':
        tools[data['tool']] += 1
for tool, count in tools.most_common():
    print(f'{tool}: {count}')
"
```

---

## Tarif 10: Gizli Bilgi Sızıntı Koruması

**Problem:** Claude, API anahtarları, şifreler veya gizli bilgiler içeren dosyaları düzenleyebilir veya bu bilgileri commit edebilir.

**Çözüm:** `PreToolUse` olayında yazma işlemlerini kontrol ederek gizli bilgi kalıplarını tespit etme.

### Konfigürasyon

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"\nimport sys, json, re\n\ndata = json.load(sys.stdin)\nfile_path = data.get('file_path', '')\nnew_string = data.get('new_string', '')\n\n# Hassas dosya kontrolü\nprotected_files = ['.env', '.env.local', '.env.production', 'credentials.json', 'secrets.yaml', '.npmrc', '.pypirc']\nfor pf in protected_files:\n    if file_path.endswith(pf):\n        print(f'ENGELLENDI: {pf} dosyası korumalıdır ve düzenlenemez.')\n        sys.exit(1)\n\n# Gizli bilgi kalıpları\nsecret_patterns = [\n    (r'(?:api[_-]?key|apikey)\\s*[=:]\\s*[\\'\\\"][A-Za-z0-9_\\-]{20,}', 'API Key'),\n    (r'(?:secret|password|passwd|pwd)\\s*[=:]\\s*[\\'\\\"][^\\s]{8,}', 'Password/Secret'),\n    (r'(?:aws_access_key_id|aws_secret)\\s*[=:]\\s*[A-Za-z0-9/+=]{20,}', 'AWS Credential'),\n    (r'ghp_[A-Za-z0-9]{36}', 'GitHub Token'),\n    (r'sk-[A-Za-z0-9]{48}', 'OpenAI API Key'),\n    (r'-----BEGIN (?:RSA )?PRIVATE KEY-----', 'Private Key'),\n    (r'(?:Bearer|token)\\s+[A-Za-z0-9_\\-\\.]{30,}', 'Bearer Token')\n]\n\nfor pattern, name in secret_patterns:\n    if re.search(pattern, new_string, re.IGNORECASE):\n        print(f'ENGELLENDI: Olası {name} tespit edildi. Gizli bilgileri koda eklemeyin.')\n        print('Öneriler:')\n        print('  - Ortam değişkeni kullanın')\n        print('  - .env dosyasına ekleyin')\n        print('  - Secret manager kullanın')\n        sys.exit(1)\n\nsys.exit(0)\n\""
          }
        ]
      },
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "PROTECTED=('.env' '.env.local' '.env.production' 'credentials.json' 'secrets.yaml'); for P in \"${PROTECTED[@]}\"; do if [[ \"$CLAUDE_FILE_PATH\" == *\"$P\" ]]; then echo \"ENGELLENDI: $P dosyası korumalıdır.\"; exit 1; fi; done; exit 0"
          }
        ]
      }
    ]
  }
}
```

---

## Tüm Tarifleri Birleştiren Kapsamlı Konfigürasyon

Yukarıdaki tariflerin en önemli parçalarını bir araya getiren gerçek dünya konfigürasyonu:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Node: $(node -v 2>/dev/null || echo 'YOK')\" && echo \"Dizin: $(pwd)\" && echo \"Branch: $(git branch --show-current 2>/dev/null || echo 'N/A')\" && if [ ! -d 'node_modules' ] && [ -f 'package.json' ]; then echo '⚠️ node_modules eksik, npm install çalıştırılıyor...' && npm install --silent; fi"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"import sys,json; cmd=json.load(sys.stdin).get('command',''); dangerous=['rm -rf /','rm -rf ~','DROP TABLE','DROP DATABASE','git push --force','mkfs.']; sys.exit(1 if any(d in cmd for d in dangerous) else 0)\""
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATH\" == *.ts || \"$CLAUDE_FILE_PATH\" == *.tsx || \"$CLAUDE_FILE_PATH\" == *.js || \"$CLAUDE_FILE_PATH\" == *.jsx ]]; then npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null; fi",
            "timeout_ms": 15000
          },
          {
            "type": "command",
            "command": "LOG_DIR=\"$HOME/.claude/logs\"; mkdir -p \"$LOG_DIR\"; echo \"$(date -u '+%Y-%m-%dT%H:%M:%SZ') | EDIT | $CLAUDE_FILE_PATH\" >> \"$LOG_DIR/changes.log\"",
            "async": true
          }
        ]
      },
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATH\" == *.ts || \"$CLAUDE_FILE_PATH\" == *.tsx || \"$CLAUDE_FILE_PATH\" == *.js || \"$CLAUDE_FILE_PATH\" == *.jsx ]]; then npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null; fi",
            "timeout_ms": 15000
          }
        ]
      }
    ],
    "PermissionRequest": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "exit 0"
          }
        ]
      },
      {
        "matcher": "Glob",
        "hooks": [
          {
            "type": "command",
            "command": "exit 0"
          }
        ]
      },
      {
        "matcher": "Grep",
        "hooks": [
          {
            "type": "command",
            "command": "exit 0"
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date -u '+%Y-%m-%dT%H:%M:%SZ') | Oturum sona erdi | $(pwd)\" >> \"$HOME/.claude/logs/sessions.log\"",
            "async": true
          }
        ]
      }
    ]
  }
}
```

---

## Tarif Seçim Rehberi

```mermaid
flowchart TD
    START["Ne yapmak\nistiyorsunuz?"] --> Q1{"Kod kalitesi\nmi?"}
    Q1 -->|Evet| R1["Tarif 1: Formatlama\nTarif 7: Test"]
    Q1 -->|Hayır| Q2{"Güvenlik\nmi?"}
    Q2 -->|Evet| R2["Tarif 2: Komut engelleme\nTarif 10: Sızıntı koruması"]
    Q2 -->|Hayır| Q3{"Bildirim\nmi?"}
    Q3 -->|Evet| R3["Tarif 3: Slack\nTarif 6: Loglama"]
    Q3 -->|Hayır| Q4{"İş akışı\noptimizasyonu mu?"}
    Q4 -->|Evet| R4["Tarif 4: Otomatik onay\nTarif 5: Kural zorlama\nTarif 8: Ortam doğrulama"]
    Q4 -->|Hayır| R5["Tarif 9: Metrik\nve telemetri"]

    style R1 fill:#3498DB,color:#fff
    style R2 fill:#E74C3C,color:#fff
    style R3 fill:#1ABC9C,color:#fff
    style R4 fill:#27AE60,color:#fff
    style R5 fill:#9B59B6,color:#fff
```

---

## Özet

| Tarif | Olay | Amaç | Zorluk |
|-------|------|------|--------|
| **1. Otomatik Formatlama** | PostToolUse | Kod stili tutarlılığı | Kolay |
| **2. Tehlikeli Komut Engelleme** | PreToolUse | Güvenlik | Orta |
| **3. Slack Bildirimi** | SessionEnd | İletişim | Kolay |
| **4. Otomatik İzin Onaylama** | PermissionRequest | İş akışı hızlandırma | Orta |
| **5. Proje Kuralı Zorlama** | UserPromptSubmit | Standart uyumu | Orta |
| **6. Araç Kullanım Logu** | PostToolUse/Failure | İzleme ve denetim | Kolay |
| **7. Otomatik Test** | PostToolUse | Kod kalitesi | Zor |
| **8. Ortam Doğrulama** | SessionStart | Hazırlık kontrolü | Orta |
| **9. Metrik Toplama** | Çoklu | Analiz ve raporlama | Orta |
| **10. Sızıntı Koruması** | PreToolUse | Güvenlik | Zor |

---

## Sonraki Adım

Hook'ları ve otomasyonu kapsamlı şekilde öğrendiniz. Şimdi Claude Code'u farklı IDE ve platformlarla nasıl entegre edeceğinizi inceleyelim:

→ [Bölüm 15: IDE ve Platform Entegrasyonları](../15-entegrasyonlar/README.md)
