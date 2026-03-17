# AI Agent ve Agentic Workflow

AI Agent (yapay zeka aracısı), bir görevi otonom olarak planlayabilen, uygulayabilen, sonuçları gözlemleyebilen ve gerektiğinde tekrar deneyebilen yapay zeka sistemidir. Agentic Workflow (aracı tabanlı iş akışı), bu otonom döngünün yapılandırılmış halidir ve modern AI kodlama araçlarının temelini oluşturur.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| AI destekli geliştirme kavramı | [AI Destekli Geliştirme Nedir?](./01-ai-destekli-gelistirme-nedir.md) |
| LLM nasıl çalışır | [02 - Büyük Dil Modelleri](../02-buyuk-dil-modelleri/README.md) |

---

## AI Agent Nedir?

Geleneksel bir LLM'ye soru sorduğunuzda tek bir yanıt alırsınız. Bir AI Agent ise bu yanıtla yetinmez: planlar yapar, araçlar kullanır, sonuçları değerlendirir ve hedefe ulaşana kadar iterasyon yapar.

### Chatbot vs AI Agent

```mermaid
flowchart LR
    subgraph chatbot ["Chatbot (Sohbet Botu)"]
        direction TB
        C1["Kullanıcı sorar"] --> C2["Model yanıtlar"]
        C2 --> C3["Bitti"]
    end

    subgraph agent ["AI Agent (Yapay Zeka Aracısı)"]
        direction TB
        A1["Kullanıcı görev verir"] --> A2["Agent planlar"]
        A2 --> A3["Araç kullanır"]
        A3 --> A4["Sonucu gözlemler"]
        A4 --> A5{"Hedef<br/>tamam mı?"}
        A5 -->|"Hayır"| A2
        A5 -->|"Evet"| A6["Sonuç sunar"]
    end

```

| Özellik | Chatbot | AI Agent |
|---------|---------|----------|
| **Etkileşim** | Soru → Yanıt | Görev → Otonom çalışma |
| **Araç kullanımı** | Yok | Dosya, terminal, web, API |
| **Bellek** | Oturum içi sınırlı | Kalıcı bellek (CLAUDE.md vb.) |
| **Planlama** | Yok | Çok adımlı plan oluşturma |
| **Hata düzeltme** | "Şunu dene" der | Kendisi dener ve düzeltir |
| **İterasyon** | Manuel | Otomatik |
| **Örnek** | ChatGPT web, Claude.ai | Claude Code, Codex CLI |

---

## AI Agent'ın 4 Temel Yeteneği

Bir AI Agent'ı chatbot'tan ayıran dört temel yetenek vardır:

```mermaid
flowchart LR
    AG((AI Agent))

    AG --> R["Reasoning<br/>(Akıl Yürütme)"]
    AG --> TU["Tool Use<br/>(Araç Kullanımı)"]
    AG --> M["Memory<br/>(Bellek)"]
    AG --> AU["Autonomy<br/>(Özerklik)"]

    R --> R1["Görevi anlama"]
    R --> R2["Plan oluşturma"]
    R --> R3["Karar verme"]
    R --> R4["Hata analizi"]

    TU --> T1["Dosya okuma/yazma"]
    TU --> T2["Terminal komutları"]
    TU --> T3["Web arama"]
    TU --> T4["API çağrıları"]

    M --> M1["Kısa süreli bellek"]
    M --> M2["Uzun süreli bellek"]
    M1 --> M1A["Context window"]
    M2 --> M2A["CLAUDE.md"]
    M2 --> M2B["Session history"]

    AU --> A1["Kendi kararını verme"]
    AU --> A2["İterasyon yapma"]
    AU --> A3["Hata düzeltme"]
    AU --> A4["İzin isteme"]
```

### 1. Reasoning (Akıl Yürütme)

Agent, görevi anlamak ve plan oluşturmak için akıl yürütme yeteneğine sahiptir:

```
Görev: "Kullanıcı profil sayfasına avatar yükleme özelliği ekle"

Agent'ın düşünce süreci:
├── 1. Mevcut User modelini incele (avatar alanı var mı?)
├── 2. Dosya yükleme altyapısını kontrol et (multer, S3 vb.)
├── 3. Gerekli değişiklikleri planla:
│   ├── User modeline avatar alanı ekle
│   ├── Upload endpoint'i oluştur
│   ├── Frontend'de yükleme bileşeni ekle
│   └── Testleri yaz
└── 4. Uygulama sırasını belirle
```

### 2. Tool Use (Araç Kullanımı)

Agent, çeşitli araçları kullanarak gerçek dünyada değişiklik yapabilir:

```
Agent'ın kullanabildiği araçlar:
├── 📁 Dosya İşlemleri
│   ├── Dosya okuma (Read)
│   ├── Dosya yazma (Write)
│   ├── Dosya arama (Glob, Grep)
│   └── Dosya düzenleme (Edit)
├── 💻 Terminal
│   ├── Komut çalıştırma (Bash)
│   ├── Test koşturma (npm test, pytest)
│   └── Build işlemleri
├── 🌐 Web
│   ├── Dokümantasyon okuma
│   └── API referansı arama
└── 🔧 Diğer
    ├── LSP (dil sunucusu)
    ├── Git işlemleri
    └── MCP sunucuları
```

### 3. Memory (Bellek)

Agent, farklı bellek katmanları sayesinde bağlamı korur:

| Bellek Türü | Kapsam | Süre | Örnek |
|-------------|--------|------|-------|
| **Context Window** | Mevcut oturum | Oturum boyunca | Son konuşma, dosya içerikleri |
| **CLAUDE.md** | Proje düzeyinde | Kalıcı | Proje kuralları, mimari kararlar |
| **Session Memory** | Oturum geçmişi | Oturumlar arası | Önceki oturumların özeti |
| **Auto Memory** | Öğrenilmiş bilgi | Kalıcı | Kullanıcı tercihleri |

### 4. Autonomy (Özerklik)

Agent, insan müdahalesi olmadan kararlar alıp uygulayabilir:

```mermaid
sequenceDiagram
    participant İ as İnsan
    participant A as AI Agent
    participant S as Sistem

    İ->>A: "Login sayfasındaki bug'ı düzelt"
    
    Note over A: Akıl yürütme başlıyor
    A->>S: Dosya oku: auth/login.tsx
    S-->>A: Dosya içeriği
    A->>S: Dosya oku: auth/login.test.tsx
    S-->>A: Test içeriği
    
    Note over A: Hata tespit edildi
    A->>S: Dosya düzenle: auth/login.tsx
    S-->>A: Düzenleme başarılı
    A->>S: Test çalıştır: npm test
    S-->>A: 2 test hâlâ başarısız
    
    Note over A: İkinci iterasyon
    A->>S: Dosya düzenle: auth/login.tsx
    S-->>A: Düzenleme başarılı
    A->>S: Test çalıştır: npm test
    S-->>A: Tüm testler geçti ✅
    
    A-->>İ: "Bug düzeltildi. Sorun state<br/>yönetimindeki race condition'dı.<br/>12/12 test geçiyor."
```

---

## Agentic Loop (Aracı Döngüsü)

AI Agent'ların çalışma prensibi, sürekli tekrarlanan bir döngüye dayanır. Bu döngüye Agentic Loop denir.

### Temel Döngü: Plan → Execute → Observe → Iterate

```mermaid
flowchart TD
    A["🎯 PLAN<br/>(Planla)"] --> B["⚡ EXECUTE<br/>(Uygula)"]
    B --> C["👁️ OBSERVE<br/>(Gözlemle)"]
    C --> D{"✅ Hedef<br/>tamam mı?"}
    D -->|"Hayır"| E["🔄 ITERATE<br/>(Tekrarla)"]
    E --> A
    D -->|"Evet"| F["📦 RESULT<br/>(Sonuç)"]

```

### Döngünün Her Adımı

**1. Plan (Planla)**
- Görevi alt görevlere böl
- Hangi araçlara ihtiyaç olduğunu belirle
- Uygulama sırasını oluştur

**2. Execute (Uygula)**
- Plana göre araçları çalıştır
- Dosya oku, düzenle, oluştur
- Terminal komutları çalıştır

**3. Observe (Gözlemle)**
- Araç çıktılarını oku
- Hata mesajlarını analiz et
- Test sonuçlarını değerlendir

**4. Iterate (Tekrarla)**
- Başarısız adımları yeniden planla
- Hataları düzelt
- Yaklaşımı değiştir

### Pratik Örnek: Agentic Loop Uygulamada

```
Görev: "users tablosuna email validasyonu ekle"

─── İterasyon 1 ────────────────────────────
PLAN:   User modelini bul ve incele
EXEC:   grep -r "class User" src/
OBSERVE: src/models/user.ts bulundu

─── İterasyon 2 ────────────────────────────
PLAN:   Mevcut validasyonu incele
EXEC:   read src/models/user.ts
OBSERVE: Email alanı var, validasyon yok

─── İterasyon 3 ────────────────────────────
PLAN:   Email validasyonu ekle
EXEC:   edit src/models/user.ts → regex validasyon eklendi
OBSERVE: Dosya güncellendi

─── İterasyon 4 ────────────────────────────
PLAN:   Test yaz ve çalıştır
EXEC:   write src/models/user.test.ts → 5 test senaryosu
        run npm test
OBSERVE: 4/5 test geçti, 1 test başarısız
         (uluslararası domain uzantıları)

─── İterasyon 5 ────────────────────────────
PLAN:   Regex'i düzelt (uluslararası domain)
EXEC:   edit src/models/user.ts → regex güncellendi
        run npm test
OBSERVE: 5/5 test geçti ✅

RESULT: Email validasyonu eklendi, 5 test yazıldı ve geçiyor.
```

---

## Agentic Workflow'un 5 Bileşeni

Tam bir Agentic Workflow, aşağıdaki beş bileşenden oluşur:

```mermaid
flowchart LR
    A["1️⃣<br/>Task Input<br/>(Görev Girdisi)"] --> B["2️⃣<br/>Context Assembly<br/>(Bağlam Derleme)"]
    B --> C["3️⃣<br/>Sandbox Execution<br/>(Korumalı Çalışma)"]
    C --> D["4️⃣<br/>PR Output<br/>(Çıktı Üretme)"]
    D --> E["5️⃣<br/>Human Review<br/>(İnsan İncelemesi)"]
    
    E -->|"Revizyon gerekli"| B
    
```

### 1. Task Input (Görev Girdisi)

Kullanıcının agent'a verdiği görev tanımıdır. Ne kadar spesifik olursa, sonuç o kadar iyi olur.

| Girdi Kalitesi | Örnek | Sonuç |
|----------------|-------|-------|
| **Kötü** | "Bir şeyleri düzelt" | Agent ne yapacağını bilemez |
| **Orta** | "Login'deki bug'ı düzelt" | Agent arar, birden fazla sorun bulabilir |
| **İyi** | "Login sayfasında email validasyonu eksik, kullanıcı @ işareti olmadan giriş yapabiliyor" | Agent tam olarak sorunu anlar |
| **Mükemmel** | "src/auth/login.tsx dosyasında email input'unda validasyon yok. RFC 5322 standardına uygun regex ekle ve 5 edge-case testi yaz" | Agent spesifik talimatla çalışır |

### 2. Context Assembly (Bağlam Derleme)

Agent, görevi anlamak için gerekli bağlamı derler:

```mermaid
flowchart TD
    subgraph sources ["Bağlam Kaynakları"]
        S1["CLAUDE.md<br/>Proje kuralları"]
        S2["Dosya sistemi<br/>Kod tabanı"]
        S3["Git geçmişi<br/>Değişiklik kaydı"]
        S4["Test sonuçları<br/>Mevcut durum"]
        S5["Hata logları<br/>Stack trace"]
    end

    subgraph assembly ["Bağlam Derleme"]
        A["Agent ilgili bilgileri<br/>seçer ve birleştirir"]
    end

    S1 --> A
    S2 --> A
    S3 --> A
    S4 --> A
    S5 --> A

    A --> R["Zenginleştirilmiş<br/>Bağlam"]
```

**Bağlam derleme süreci:**
1. CLAUDE.md dosyasını oku → Proje kurallarını öğren
2. İlgili dosyaları bul → Kod tabanını tara
3. Git geçmişine bak → Son değişiklikleri incele
4. Mevcut testleri çalıştır → Başlangıç durumunu belirle
5. Hata loglarını oku → Sorunun kaynağını anla

### 3. Sandbox Execution (Korumalı Çalışma)

Agent, değişikliklerini güvenli bir ortamda yapar:

```
Güvenlik Katmanları:
├── 🔒 İzin Sistemi
│   ├── Dosya yazma izni gerekli
│   ├── Terminal komutu izni gerekli
│   └── Ağ erişimi izni gerekli
├── 🏖️ Sandbox
│   ├── İzole çalışma ortamı
│   ├── Git branch üzerinde çalışma
│   └── Geri alınabilir değişiklikler
└── ✅ Doğrulama
    ├── Lint kontrolü
    ├── Type checking
    └── Test çalıştırma
```

### 4. PR Output (Çıktı Üretme)

Agent, çalışmasının sonucunu yapılandırılmış bir çıktı olarak sunar:

```
Çıktı Türleri:
├── 📝 Kod Değişiklikleri
│   ├── Yeni dosyalar
│   ├── Düzenlenmiş dosyalar
│   └── Silinen dosyalar
├── 🧪 Test Sonuçları
│   ├── Kaç test yazıldı
│   ├── Kaçı geçti/kaldı
│   └── Coverage raporu
├── 📋 Özet
│   ├── Yapılan değişiklikler listesi
│   ├── Tasarım kararları açıklaması
│   └── Bilinen sınırlamalar
└── 🔀 PR / Commit
    ├── Anlamlı commit mesajı
    ├── PR açıklaması
    └── Reviewer ataması
```

### 5. Human Review (İnsan İncelemesi)

Son karar her zaman insandadır:

```mermaid
flowchart TD
    A["Agent çıktıyı sunar"] --> B{"İnsan inceler"}
    B -->|"✅ Onay"| C["Merge / Deploy"]
    B -->|"📝 Revizyon"| D["Geri bildirim"]
    B -->|"❌ Red"| E["İptal"]
    D --> F["Agent revizyonu<br/>uygular"]
    F --> B
```

**İnceleme kontrol listesi:**
- [ ] Kod değişiklikleri mantıklı mı?
- [ ] Güvenlik açığı var mı?
- [ ] Performans sorunları olabilir mi?
- [ ] Testler yeterli mi?
- [ ] Mimari kararlara uygun mu?
- [ ] Edge case'ler düşünülmüş mü?

---

## Agentic Workflow Tam Akış Örneği

Gerçek bir görevin tüm aşamalarından geçişi:

```mermaid
sequenceDiagram
    participant U as Kullanıcı
    participant A as AI Agent
    participant D as Dosya Sistemi
    participant T as Terminal
    participant G as Git

    rect rgb(227, 242, 253)
    Note over U,G: 1. Task Input
    U->>A: "Ürün listesine sayfalama ekle"
    end

    rect rgb(255, 243, 224)
    Note over U,G: 2. Context Assembly
    A->>D: Dosya oku: src/api/products.ts
    D-->>A: Mevcut endpoint kodu
    A->>D: Dosya oku: CLAUDE.md
    D-->>A: Proje kuralları
    A->>T: npm test -- products
    T-->>A: 8/8 test geçiyor
    end

    rect rgb(232, 245, 233)
    Note over U,G: 3. Sandbox Execution
    A->>D: Düzenle: src/api/products.ts
    Note over A: offset/limit parametreleri eklendi
    A->>D: Yaz: src/api/products.test.ts
    Note over A: 6 yeni test eklendi
    A->>T: npm test -- products
    T-->>A: 14/14 test geçiyor ✅
    end

    rect rgb(243, 229, 245)
    Note over U,G: 4. PR Output
    A->>G: git add && git commit
    A->>G: git push && gh pr create
    G-->>A: PR #42 oluşturuldu
    A-->>U: "PR #42 hazır. 6 yeni test eklendi."
    end

    rect rgb(255, 235, 238)
    Note over U,G: 5. Human Review
    U->>A: "Default limit 50 yerine 20 olsun"
    A->>D: Düzenle: DEFAULT_LIMIT = 20
    A->>T: npm test -- products
    T-->>A: 14/14 ✅
    A->>G: Commit: "chore: default limit 20"
    end
```

---

## Otonom Kodlama: Fırsatlar ve Sınırlar

### Agent Ne Kadar Otonom Olmalı?

```mermaid
flowchart LR
    subgraph spectrum ["Özerklik Spektrumu"]
        A["Tamamen Manuel<br/>(İnsan yapar)"] ---- B["Copilot Mode<br/>(AI öneri yapar)"] ---- C["Supervised Agent<br/>(AI yapar,<br/>insan onaylar)"] ---- D["Autonomous Agent<br/>(AI yapar,<br/>insan inceler)"]
    end

```

| Seviye | Açıklama | Risk | Kullanım Alanı |
|--------|----------|------|----------------|
| **Manuel** | İnsan her şeyi yapar | Yavaş ama güvenli | Kritik sistemler |
| **Copilot** | AI öneri sunar, insan karar verir | Düşük risk | Günlük geliştirme |
| **Supervised** | AI yapar, her adımda onay ister | Orta risk | Çoğu görev için ideal |
| **Autonomous** | AI yapar, sonunda insan inceler | Yüksek risk | Rutin görevler |

> **Tavsiye:** Claude Code varsayılan olarak "Supervised Agent" modunda çalışır. Dosya yazma ve terminal komutları için izin ister. Bu denge, çoğu senaryo için idealdir.

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **AI Agent** | Otonom planlama, uygulama ve iterasyon yapabilen AI sistemi |
| **Agentic Loop** | Plan → Execute → Observe → Iterate döngüsü |
| **Tool Use** | Agent'ın dosya, terminal, web gibi araçları kullanabilmesi |
| **Context Assembly** | Görev için gerekli bağlam bilgisinin derlenmesi |
| **Sandbox Execution** | Korumalı ortamda güvenli çalışma |
| **Human Review** | Son kararın insanda kalması ilkesi |

---

## Sonraki Adım

AI Agent'ların nasıl çalıştığını anladık. Şimdi bu kavramlarla doğrudan bağlantılı olan "Vibe Coding" yaklaşımını inceleyelim:

→ [Vibe Coding](./03-vibe-coding.md)
