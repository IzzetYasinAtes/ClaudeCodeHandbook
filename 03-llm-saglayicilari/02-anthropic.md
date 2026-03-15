# Anthropic

Anthropic, AI Safety (yapay zeka güvenliği) odaklı bir araştırma şirketidir. OpenAI'nin eski araştırmacıları tarafından kurulan şirket, Constitutional AI (anayasal yapay zeka) yaklaşımıyla güvenli ve yararlı AI sistemleri geliştirmeyi hedefler. Bu rehberin odak noktası olan **Claude** model ailesi Anthropic tarafından geliştirilmektedir.

## Ön Koşullar

- [LLM Nedir?](../02-buyuk-dil-modelleri/01-llm-nedir.md)
- [OpenAI](./01-openai.md) (karşılaştırma için önerilir)

---

## Şirket Tarihi ve Felsefesi

### Kuruluş Hikayesi

Anthropic, 2021 yılında **Dario Amodei** (CEO) ve **Daniela Amodei** (Başkan) kardeşler tarafından kuruldu. Her ikisi de OpenAI'nin üst düzey yöneticileriydi:

- **Dario Amodei:** OpenAI'de Araştırma Başkan Yardımcısı olarak GPT-2 ve GPT-3 projelerini yönetti
- **Daniela Amodei:** OpenAI'de Operasyon Başkan Yardımcısıydı

OpenAI'den ayrılma nedeni, yapay zeka güvenliği konusundaki **felsefe farklılığıydı**. Amodei kardeşler, AI geliştirmede güvenliğin hız ve kârdan önce gelmesi gerektiğine inanıyorlardı.

```mermaid
timeline
    title Anthropic Tarihçesi
    2021 : Kuruluş (Dario & Daniela Amodei)
         : OpenAI'den ayrılan 7 araştırmacı
    2022 : Constitutional AI makalesi yayınlandı
         : Claude 1.0 (sınırlı erişim)
    2023 : Claude 2, Amazon ortaklığı ($4B yatırım)
         : Claude 2.1 (200K context window)
    2024 : Claude 3 ailesi (Haiku, Sonnet, Opus)
         : Claude 3.5 Sonnet — kodlamada en iyi model
    2025 : Claude 4 serisi, MCP protokolü
         : Claude Code lansmanı
    2026 : Claude 4.5, Claude 4.6 Opus
         : Toplam yatırım $15B+ ulaştı
```

---

## AI Safety (Yapay Zeka Güvenliği) Yaklaşımı

Anthropic'in diğer şirketlerden en büyük farkı, güvenliği bir "ek özellik" değil **temel tasarım ilkesi** olarak görmesidir.

### Üç Temel İlke

```mermaid
flowchart TD
    A["Anthropic'in AI Güvenlik Yaklaşımı"]

    A --> B["Helpful<br/>(Yararlı)"]
    A --> C["Harmless<br/>(Zararsız)"]
    A --> D["Honest<br/>(Dürüst)"]

    B --> B1["Kullanıcının ihtiyacını<br/>en iyi şekilde karşıla"]
    C --> C1["Zararlı, tehlikeli veya<br/>ayrımcı içerik üretme"]
    D --> D1["Bilmediğini kabul et<br/>Yanıltıcı bilgi verme"]

    style B fill:#4a9eff,color:#fff
    style C fill:#ff6b6b,color:#fff
    style D fill:#51cf66,color:#fff
```

Bu üç ilke birbiriyle bazen çelişebilir. Örneğin, "yararlı" olmak bazen "zararsız" olmakla çatışabilir. Constitutional AI, bu dengeyi sağlamak için geliştirilmiştir.

---

## Constitutional AI (Anayasal Yapay Zeka)

Constitutional AI (CAI), Anthropic'in geliştirdiği özgün bir eğitim metodolojisidir. Modelin kendi çıktılarını bir "anayasa" (kurallar seti) temelinde değerlendirmesine dayanır.

### CAI Süreci

```mermaid
flowchart TD
    subgraph asama1 ["Aşama 1: Supervised Learning (Denetimli Öğrenme)"]
        A["İnsan yazarlar örnek<br/>soru-cevap çiftleri oluşturur"] --> B["Model bu örneklerle<br/>ince ayar (fine-tune) yapılır"]
    end

    subgraph asama2 ["Aşama 2: Kendi Kendini Eleştirme"]
        C["Modelden bir yanıt<br/>üretmesi istenir"] --> D["Aynı modelden yanıtı<br/>anayasaya göre eleştirmesi istenir"]
        D --> E["Model eleştiri doğrultusunda<br/>yanıtı revize eder"]
    end

    subgraph asama3 ["Aşama 3: RLAIF"]
        F["Revize edilmiş yanıtlar<br/>ile orijinaller karşılaştırılır"] --> G["Model hangisinin anayasaya<br/>daha uygun olduğunu seçer"]
        G --> H["Bu tercihler ile<br/>RL eğitimi yapılır"]
    end

    B --> C
    E --> F

    style asama2 fill:#f0f4ff,stroke:#4a9eff
```

> **RLAIF** = Reinforcement Learning from AI Feedback (yapay zeka geri bildiriminden pekiştirmeli öğrenme). Geleneksel RLHF'den (insan geri bildirimi) farklı olarak, modelin kendisi geri bildirim verir.

### Anayasa Örneği

CAI'deki "anayasa" şu tür kurallardan oluşur:

1. Yanıtın zararlı, etik dışı veya yasadışı faaliyetleri teşvik edip etmediğini kontrol et
2. Yanıtın doğru ve kanıta dayalı olup olmadığını değerlendir
3. Yanıtın ayrımcı, önyargılı veya kırıcı olup olmadığını kontrol et
4. Yanıtın kullanıcının sorusuna gerçekten yardımcı olup olmadığını değerlendir
5. Yanıtın belirsizlik durumunda bunu açıkça ifade edip etmediğini kontrol et

---

## Responsible Scaling Policy (Sorumlu Ölçekleme Politikası)

Anthropic, modellerin yetenekleri arttıkça güvenlik önlemlerinin de artması gerektiğini savunur. Bu yaklaşım **AI Safety Level (ASL)** sistemiyle tanımlanır:

```mermaid
flowchart LR
    ASL1["ASL-1<br/>Temel sohbet<br/>Düşük risk"]
    ASL2["ASL-2<br/>Gelişmiş yetenekler<br/>Orta risk"]
    ASL3["ASL-3<br/>Uzman düzey bilgi<br/>Yüksek risk"]
    ASL4["ASL-4<br/>Otonom hareket<br/>Çok yüksek risk"]

    ASL1 -->|"Daha fazla<br/>güvenlik önlemi"| ASL2
    ASL2 -->|"Red teaming<br/>Ağır testler"| ASL3
    ASL3 -->|"Harici denetim<br/>Hükümet işbirliği"| ASL4

    style ASL1 fill:#51cf66,color:#fff
    style ASL2 fill:#ffd43b,color:#333
    style ASL3 fill:#ff922b,color:#fff
    style ASL4 fill:#ff6b6b,color:#fff
```

| Seviye | Yetenek | Güvenlik Gereksinimi |
|--------|---------|----------------------|
| ASL-1 | Basit sohbet, sınırlı bilgi | Standart test |
| ASL-2 | Geniş bilgi, kod üretme | Kırmızı takım testleri, kullanım politikaları |
| ASL-3 | Uzman düzey biyoloji/siber bilgi | Bağımsız güvenlik denetimi, erişim kontrolü |
| ASL-4 | Otonom görev yürütme | Hükümet düzeyinde denetim |

> **Mart 2026:** Claude 4.6 modelleri ASL-3 kategorisinde değerlendirilmektedir.

---

## Claude Model Ailesi

Claude, Anthropic'in amiral gemisi model ailesidir. Her nesil üç katmanda sunulur:

```mermaid
flowchart TD
    subgraph claude3 ["Claude 3 (Mart 2024)"]
        C3H["Haiku<br/>(Hızlı & Hafif)"]
        C3S["Sonnet<br/>(Dengeli)"]
        C3O["Opus<br/>(En Yetenekli)"]
    end

    subgraph claude35 ["Claude 3.5 (2024)"]
        C35H["Haiku"]
        C35S["Sonnet ⭐<br/>(Kodlamada lider)"]
    end

    subgraph claude4 ["Claude 4 (2025)"]
        C4H["Haiku"]
        C4S["Sonnet"]
        C4O["Opus"]
    end

    subgraph claude45_46 ["Claude 4.5 / 4.6 (2026)"]
        C45S["4.5 Sonnet"]
        C46S["4.6 Sonnet"]
        C46O["4.6 Opus ⭐<br/>(En güncel & en güçlü)"]
    end

    claude3 --> claude35
    claude35 --> claude4
    claude4 --> claude45_46
```

### Katman Yapısı

| Katman | Özellik | Kullanım Alanı |
|--------|---------|----------------|
| **Haiku** | En hızlı, en düşük maliyet | Sınıflandırma, basit sorular, yüksek hacim |
| **Sonnet** | Hız ve yetenek dengesi | Kodlama, analiz, günlük kullanım |
| **Opus** | En yetenekli, en derin akıl yürütme | Karmaşık araştırma, yaratıcı yazım, uzman görevler |

### Mart 2026 Güncel Claude Modelleri

| Model | Context Window | Öne Çıkan Özellik |
|-------|----------------|---------------------|
| Claude 4.6 Opus | 200K token | En güçlü model, derin reasoning |
| Claude 4.6 Sonnet | 200K token | En iyi fiyat/performans oranı |
| Claude 4.5 Sonnet | 200K token | Güvenilir ve stabil |
| Claude 4 Haiku | 200K token | Ultra hızlı yanıtlar |

---

## Claude'un Öne Çıkan Özellikleri

### 1. Extended Thinking (Genişletilmiş Düşünme)

Claude, karmaşık problemlerde "düşünme süreci"ni kullanıcıya gösterebilir. Bu, OpenAI'nin o-serisine benzer bir yaklaşımdır ancak kullanıcıya şeffaf biçimde sunulur.

### 2. Tool Use (Araç Kullanımı)

Claude, API üzerinden harici araçlara (fonksiyonlara) erişebilir. Bu, Claude Code'un temelini oluşturan mekanizmadır.

### 3. Vision (Görsel Anlama)

Tüm Claude 3+ modelleri görselleri anlayabilir: diyagramlar, ekran görüntüleri, el yazıları, grafikler.

### 4. Model Context Protocol (MCP)

Anthropic tarafından geliştirilen açık standart protokol. AI modellerinin harici veri kaynaklarına ve araçlara standart bir şekilde bağlanmasını sağlar. Detaylar için → [Bölüm 11](../11-mcp/README.md)

---

## API Fiyatlandırması (Mart 2026)

| Model | Input (Giriş) | Output (Çıkış) |
|-------|----------------|-----------------|
| Claude 4.6 Opus | $15.00 / 1M token | $75.00 / 1M token |
| Claude 4.6 Sonnet | $3.00 / 1M token | $15.00 / 1M token |
| Claude 4.5 Sonnet | $3.00 / 1M token | $15.00 / 1M token |
| Claude 4 Haiku | $0.80 / 1M token | $4.00 / 1M token |

---

## Güçlü ve Zayıf Yanlar

| Güçlü Yanlar | Zayıf Yanlar |
|-------------|-------------|
| AI Safety konusunda sektör lideri | Model çeşitliliği sınırlı (sadece Claude) |
| Kodlama benchmark'larında en üst sıralarda | Multimodal yetenekler OpenAI'nin gerisinde (video, ses) |
| 200K token context window | Görsel üretim yeteneği yok |
| MCP açık standart protokolü | Tüketici ürünü (ChatGPT benzeri) görece geç geldi |
| Claude Code ile güçlü geliştirici deneyimi | Fiyatlandırma GPT-5'e kıyasla daha yüksek (Opus) |
| Uzun, tutarlı metin üretiminde çok başarılı | Bazı ülkelerde erişim kısıtlı |

---

## Pratik Örnek: Claude API Kullanımı

```python
import anthropic

client = anthropic.Anthropic(api_key="YOUR_API_KEY_HERE")

# Claude 4.6 Sonnet ile kod analizi
message = client.messages.create(
    model="claude-sonnet-4-6-20260301",
    max_tokens=2000,
    messages=[
        {
            "role": "user",
            "content": """Bu Python kodundaki güvenlik açıklarını bul ve düzelt:

import sqlite3

def get_user(username):
    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()
    cursor.execute(f"SELECT * FROM users WHERE name = '{username}'")
    return cursor.fetchone()
"""
        }
    ]
)

print(message.content[0].text)
```

```python
# Extended Thinking ile karmaşık problem çözme
message = client.messages.create(
    model="claude-sonnet-4-6-20260301",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000
    },
    messages=[
        {
            "role": "user",
            "content": "Dağıtık bir e-ticaret sistemi için event-driven mimari tasarla."
        }
    ]
)

for block in message.content:
    if block.type == "thinking":
        print("Düşünce süreci:", block.thinking)
    elif block.type == "text":
        print("Yanıt:", block.text)
```

---

## Anthropic vs OpenAI: Felsefe Farkı

```mermaid
flowchart LR
    subgraph openai ["OpenAI Yaklaşımı"]
        OA1["Hızlı ürün geliştirme"] --> OA2["Geniş ekosistem"]
        OA2 --> OA3["Güvenlik sonradan<br/>entegre edilir"]
    end

    subgraph anthropic ["Anthropic Yaklaşımı"]
        AN1["Güvenlik öncelikli<br/>araştırma"] --> AN2["Odaklı ürün seti"]
        AN2 --> AN3["Güvenlik tasarımın<br/>parçası"]
    end
```

---

## Özet

| Özellik | Detay |
|---------|-------|
| **Kuruluş** | 2021, San Francisco (Dario & Daniela Amodei) |
| **Felsefe** | AI Safety öncelikli, Constitutional AI |
| **Amiral gemisi** | Claude 4.6 Opus / Sonnet |
| **Öne çıkan** | Kodlama, güvenlik, MCP protokolü, Claude Code |
| **Fiyat aralığı** | $0.80 — $75 / 1M token |

---

## Sonraki Adım

Anthropic ve Claude'u tanıdık. Şimdi en büyük context window ve multimodal yetenekleriyle öne çıkan Google DeepMind'ı inceleyelim:

→ [Google DeepMind](./03-google-deepmind.md)
