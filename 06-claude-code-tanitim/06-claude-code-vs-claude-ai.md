# Claude Code vs Claude AI

Claude ailesi iki farklı kullanım şekli sunar: **Claude AI** (claude.ai web arayüzü) ve **Claude Code** (terminal tabanlı kodlama ajanı). Bu bölümde her ikisinin yeteneklerini, farklarını ve ne zaman hangisini kullanmanız gerektiğini öğreneceksiniz.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code nedir | [Claude Code Nedir?](./01-claude-code-nedir.md) |
| İlk oturum deneyimi | [İlk Oturum](./05-ilk-oturum.md) |

---

## Genel Bakış

```mermaid
flowchart LR
    subgraph claude_ai ["Claude AI (claude.ai)"]
        direction TB
        A1["Web tarayıcısında çalışır"]
        A2["Genel amaçlı sohbet"]
        A3["Dosya yükleme"]
        A4["Görsel analiz"]
        A5["Artifacts ile kod/doküman"]
    end

    subgraph claude_code ["Claude Code (Terminal)"]
        direction TB
        B1["Terminalinizde çalışır"]
        B2["Kodlama odaklı ajan"]
        B3["Dosya sistemi erişimi"]
        B4["Komut çalıştırma"]
        B5["Git entegrasyonu"]
    end

    MODEL["Claude Opus 4.6\n(Aynı model)"] --> claude_ai
    MODEL --> claude_code
```

Her iki ürün de **aynı Claude modeli** üzerine inşa edilmiştir. Temel fark, modelin erişebildiği **araçlar** ve **çalışma ortamıdır**.

---

## Detaylı Karşılaştırma Tablosu

| Özellik | Claude AI (claude.ai) | Claude Code (Terminal) |
|---------|----------------------|------------------------|
| **Arayüz** | Web tarayıcısı | Terminal / CLI |
| **Erişim** | claude.ai | `npm install -g @anthropic-ai/claude-code` |
| **Temel kullanım** | Genel sohbet, yazma, analiz | Kod okuma, yazma, çalıştırma |
| **Model** | Claude Opus 4.6 | Claude Opus 4.6 |
| **Context window** | 200K token | 200K token |
| **Dosya sistemi erişimi** | ❌ Hayır (yükleme ile sınırlı) | ✅ Tam erişim |
| **Komut çalıştırma** | ❌ Hayır | ✅ Shell komutları |
| **Git entegrasyonu** | ❌ Hayır | ✅ Commit, PR, log |
| **Web erişimi** | ✅ Sınırlı | ✅ WebFetch, WebSearch |
| **Görsel analiz** | ✅ Evet | ⚠️ Sınırlı |
| **Dosya yükleme** | ✅ PDF, görsel, metin | ❌ Gerek yok (direkt okur) |
| **Artifacts** | ✅ Kod, doküman, diyagram | ❌ Hayır (dosyaya yazar) |
| **Çoklu konuşma** | ✅ Evet | ✅ Evet (oturumlar) |
| **MCP desteği** | ❌ Hayır | ✅ Evet |
| **Subagent'lar** | ❌ Hayır | ✅ Evet |
| **İzin sistemi** | Yok (gerek yok) | ✅ Var (güvenlik için) |
| **Otonom çalışma** | ❌ Hayır | ✅ Evet (agentic loop) |
| **IDE entegrasyonu** | ❌ Hayır | ✅ VS Code, JetBrains |
| **CI/CD entegrasyonu** | ❌ Hayır | ✅ GitHub Actions vb. |
| **Fiyat** | Pro $20/ay | Pro $20/ay (aynı abonelik) |

---

## Kullanım Senaryoları

### Claude AI En İyi Ne İçin?

```mermaid
flowchart TD
    subgraph ai_kullanim ["Claude AI Kullanım Alanları"]
        direction TB
        AI1["📝 Metin yazma ve düzenleme\nBlog, e-posta, rapor"]
        AI2["🔍 Araştırma ve analiz\nKonu özetleme, karşılaştırma"]
        AI3["🖼️ Görsel analiz\nEkran görüntüsü, diyagram okuma"]
        AI4["📄 Doküman inceleme\nPDF analizi, sözleşme okuma"]
        AI5["💡 Beyin fırtınası\nFikir üretme, planlama"]
        AI6["📊 Veri analizi\nTablo, grafik yorumlama"]
    end
```

**Claude AI tercih edin:**
- Genel sohbet ve soru-cevap
- Metin yazma, düzenleme, çeviri
- PDF veya görsel dosya analizi
- Teknik olmayan iş görevleri
- Hızlı bilgi arama
- Beyin fırtınası ve fikir üretme

### Claude Code En İyi Ne İçin?

```mermaid
flowchart TD
    subgraph code_kullanim ["Claude Code Kullanım Alanları"]
        direction TB
        CC1["💻 Kod geliştirme\nYeni özellik, refactoring"]
        CC2["🐛 Bug düzeltme\nHata bulma ve onarma"]
        CC3["🧪 Test yazma\nUnit test, integration test"]
        CC4["📦 Proje yönetimi\nGit, build, deploy"]
        CC5["🔧 DevOps görevleri\nCI/CD, otomasyon"]
        CC6["📖 Kod dokümantasyonu\nREADME, API docs"]
    end
```

**Claude Code tercih edin:**
- Proje çapında kod okuma ve anlama
- Yeni özellik geliştirme
- Bug bulma ve düzeltme
- Test yazma ve çalıştırma
- Git işlemleri (commit, branch, PR)
- Büyük ölçekli refactoring
- CI/CD otomasyon görevleri
- Proje genelinde arama ve analiz

---

## Karar Akış Şeması

Hangi aracı kullanmanız gerektiğini belirlemek için bu akış şemasını takip edin:

```mermaid
flowchart TD
    START(["Ne yapmak\nistiyorsunuz?"]) --> Q1{"Kod veya\nproje ile mi ilgili?"}

    Q1 -->|"Hayır"| Q2{"Dosya analizi\ngerekiyor mu?"}
    Q1 -->|"Evet"| Q3{"Dosya sistemi\nerişimi lazım mı?"}

    Q2 -->|"PDF/Görsel var"| CLAUDE_AI_1["Claude AI\n(Dosya yükleme ile)"]
    Q2 -->|"Hayır"| CLAUDE_AI_2["Claude AI\n(Genel sohbet)"]

    Q3 -->|"Hayır, tek dosya"| Q4{"Otonom çalışma\ngerekiyor mu?"}
    Q3 -->|"Evet, proje geneli"| CLAUDE_CODE_1["Claude Code\n(Proje erişimi)"]

    Q4 -->|"Sadece soru-cevap"| CLAUDE_AI_3["Claude AI veya\nClaude Code\n(ikisi de olur)"]
    Q4 -->|"Okuma + Yazma + Çalıştırma"| CLAUDE_CODE_2["Claude Code\n(Otonom ajan)"]

    style CLAUDE_AI_1 fill:#3498DB,color:#fff
    style CLAUDE_AI_2 fill:#3498DB,color:#fff
    style CLAUDE_AI_3 fill:#9B59B6,color:#fff
    style CLAUDE_CODE_1 fill:#27AE60,color:#fff
    style CLAUDE_CODE_2 fill:#27AE60,color:#fff
```

---

## Pratik Karşılaştırma Örnekleri

### Senaryo 1: "Bu hatayı düzelt"

**Claude AI ile:**
```
Kullanıcı: Bu hata mesajını alıyorum: "TypeError: Cannot read property 
'map' of undefined". Kod şu: [kodu yapıştırır]

Claude AI: Bu hata, data değişkeninin undefined olmasından kaynaklanıyor.
Şu şekilde düzeltebilirsiniz: [düzeltilmiş kodu gösterir]

Kullanıcı: [kodu elle kopyalayıp yapıştırır, dosyayı kaydeder]
```

**Claude Code ile:**
```
> src/UserList.tsx'deki "Cannot read property 'map' of undefined" hatasını düzelt

⏺ Read src/UserList.tsx
⏺ data undefined olabilir. Optional chaining ve varsayılan değer ekliyorum.
⏺ Edit src/UserList.tsx
  ✓ Applied fix
⏺ Running: npm test
  ✓ All tests passed

Düzeltme tamamlandı. data?.users ?? [] ile null safety eklendi.
```

### Senaryo 2: "README yaz"

**Claude AI ile:**
```
Kullanıcı: [package.json ve src dizin yapısını yapıştırır]
Bu proje için README yaz.

Claude AI: [README metnini gösterir]

Kullanıcı: [çıktıyı kopyalar, README.md oluşturur, yapıştırır]
```

**Claude Code ile:**
```
> Bu proje için kapsamlı bir README.md oluştur

⏺ Read package.json
⏺ Glob src/**/*
⏺ Read src/App.tsx
⏺ Read src/routes/index.ts
⏺ Write README.md
  ✓ Created README.md (142 lines)

README.md oluşturuldu. İçerik: proje tanımı, kurulum adımları,
API referansı, katkıda bulunma rehberi.
```

### Fark Özeti

```mermaid
flowchart LR
    subgraph ai_flow ["Claude AI Akışı"]
        direction TB
        A1["Kodu kopyala"] --> A2["Claude AI'a yapıştır"]
        A2 --> A3["Yanıtı oku"]
        A3 --> A4["Çözümü elle uygula"]
    end

    subgraph code_flow ["Claude Code Akışı"]
        direction TB
        B1["Görevi tanımla"] --> B2["Claude Code çalışır"]
        B2 --> B3["İzinleri onayla"]
        B3 --> B4["Sonuç otomatik uygulanır"]
    end

    ai_flow -.->|"Manuel süreç"| MANUAL["⏱️ Daha fazla zaman"]
    code_flow -.->|"Otomatik süreç"| AUTO["⚡ Daha az zaman"]
```

---

## Birlikte Kullanım

Claude AI ve Claude Code birbirinin rakibi değil, **tamamlayıcısıdır**. Tipik bir iş günü:

```mermaid
flowchart TD
    SABAH(["Sabah"]) --> AI1["Claude AI:\nGünlük e-postaları özetle"]
    AI1 --> CODE1["Claude Code:\nDünkü PR'leri incele"]
    CODE1 --> CODE2["Claude Code:\nBug #342'yi düzelt"]
    CODE2 --> AI2["Claude AI:\nMimari tasarım tartışması"]
    AI2 --> CODE3["Claude Code:\nYeni API endpoint'i oluştur"]
    CODE3 --> CODE4["Claude Code:\nTestleri yaz ve çalıştır"]
    CODE4 --> AI3["Claude AI:\nSprint raporu yaz"]
    AI3 --> AKSAM(["Akşam"])

    style SABAH fill:#4A90D9,color:#fff
    style AKSAM fill:#27AE60,color:#fff
```

---

## Özet

| Kriter | Claude AI | Claude Code |
|--------|-----------|-------------|
| **Ne zaman** | Genel görevler, yazı, analiz | Kodlama, proje yönetimi |
| **Nerede** | Web tarayıcısı | Terminal |
| **Nasıl** | Sohbet, dosya yükleme | Doğal dil komutları, otonom |
| **Avantaj** | Kolay erişim, görsel destek | Dosya/komut erişimi, otomasyon |
| **Sınırlama** | Dosya sistemi yok | GUI yok, terminal gerekli |

> **Kural:** Elinizde kod varsa ve üzerinde çalışmanız gerekiyorsa → **Claude Code**. Geri kalan her şey için → **Claude AI**.

---

## Sonraki Adım

Claude Code'un ne olduğunu, nasıl çalıştığını, nasıl kurulacağını ve Claude AI'dan farkını öğrendiniz. Şimdi arayüz detaylarına ve komut referansına geçelim:

→ [Bölüm 07 — Arayüz ve Komutlar](../07-arayuz-ve-komutlar/README.md)
