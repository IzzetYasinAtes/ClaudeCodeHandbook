# Diğer Önemli LLM Sağlayıcıları

OpenAI, Anthropic, Google ve Meta dışında, büyük dil modelleri alanında güçlü rakipler bulunmaktadır. Bu bölümde DeepSeek, Alibaba Qwen, Mistral, Microsoft Phi, Cohere ve 01.AI Yi modellerini inceliyoruz.

## Ön Koşullar

- [LLM Nedir?](../02-buyuk-dil-modelleri/01-llm-nedir.md)
- [Açık Kaynak vs Kapalı Kaynak](../02-buyuk-dil-modelleri/04-acik-kaynak-vs-kapali-kaynak.md)

---

## Sağlayıcı Haritası

```mermaid
flowchart TD
    D["Diğer LLM Sağlayıcıları"]

    D --> DS["DeepSeek 🇨🇳\nAçık kaynak reasoning"]
    D --> QW["Qwen (Alibaba) 🇨🇳\nÇok dilli, açık kaynak"]
    D --> MI["Mistral 🇫🇷\nAvrupa uyumlu"]
    D --> PH["Microsoft Phi 🇺🇸\nKüçük & güçlü"]
    D --> CO["Cohere 🇨🇦\nKurumsal RAG"]
    D --> YI["01.AI Yi 🇨🇳\nMultimodal"]

    style DS fill:#4ecdc4,color:#fff
    style QW fill:#ff6b35,color:#fff
    style MI fill:#7c3aed,color:#fff
    style PH fill:#0078d4,color:#fff
    style CO fill:#39d353,color:#fff
    style YI fill:#e63946,color:#fff
```

---

## DeepSeek

### Genel Bakış

DeepSeek, Çin merkezli bir AI laboratuvarıdır. 2024 yılında DeepSeek-R1 modeli ile açık kaynak reasoning (akıl yürütme) alanında büyük ses getirmiştir. Modellerin **MIT lisansı** ile yayınlanması, sektörde geniş yankı uyandırmıştır.

| Özellik | Detay |
|---------|-------|
| **Merkez** | Hangzhou, Çin |
| **Kuruluş** | 2023 (High-Flyer hedge fund tarafından finanse ediliyor) |
| **Strateji** | Açık kaynak (MIT lisansı) |
| **Öne çıkan** | Düşük maliyetli eğitim, MoE mimarisi |

### Model Ailesi

```mermaid
flowchart TD
    DS["DeepSeek"]

    DS --> R["DeepSeek-R1\n(Reasoning)"]
    DS --> V["DeepSeek-V3.2\n(Genel Amaçlı)"]
    DS --> CODER["DeepSeek-Coder\n(Kodlama)"]

    R --> R1["R1\n671B (MoE)\n37B aktif"]
    R --> R1D["R1-Distill\n1.5B - 70B\n(Damıtılmış)"]

    V --> V32["V3.2\n685B (MoE)\n37B aktif"]

    CODER --> COD["Coder V3\n(Kod uzmanı)"]

    style R1 fill:#4ecdc4,color:#fff
    style V32 fill:#4ecdc4,color:#fff
```

### DeepSeek-R1: Reasoning Devrimi

DeepSeek-R1, o1 ve o3 modelleriyle rekabet eden açık kaynak bir reasoning modelidir:

| Özellik | Değer |
|---------|-------|
| **Toplam parametre** | 671 milyar (MoE) |
| **Aktif parametre** | ~37B (her token için) |
| **Expert sayısı** | 256 expert, 8 aktif |
| **Context window** | 128K token |
| **Lisans** | **MIT** (en serbest açık kaynak lisanslarından) |
| **Eğitim maliyeti** | ~$5.6M (GPT-4'ün tahmini maliyetinin onda biri) |

### DeepSeek-V3.2

| Özellik | Değer |
|---------|-------|
| **Toplam parametre** | 685B (MoE) |
| **Aktif parametre** | ~37B |
| **Context window** | 128K token |
| **Öne çıkan** | Genel amaçlı, çok dilli, kodlama |

### MoE Verimliliği

DeepSeek'in MoE yaklaşımının dikkat çekici yanı, **devasa modelleri çok düşük maliyetle** çıkarım (inference) yapabilmesidir:

```mermaid
flowchart LR
    subgraph geleneksel ["Geleneksel Dense Model"]
        T1["GPT-5\nTüm parametreler aktif\nYüksek hesaplama maliyeti"]
    end

    subgraph moe ["DeepSeek MoE"]
        T2["671B toplam parametre\nAma sadece 37B aktif\nDüşük hesaplama maliyeti"]
    end

    T1 -.->|"~10x daha\npahalı çıkarım"| T2
```

### API Fiyatlandırması

| Model | Input | Output |
|-------|-------|--------|
| DeepSeek-R1 | $0.55 / 1M token | $2.19 / 1M token |
| DeepSeek-V3.2 | $0.27 / 1M token | $1.10 / 1M token |

> **Fiyat avantajı:** DeepSeek, GPT-5'ten yaklaşık **40x daha ucuzdur** ve Claude 4.6 Sonnet'ten yaklaşık **10x daha ucuzdur**.

---

## Alibaba Qwen (通义千问)

### Genel Bakış

Qwen (Tongyi Qianwen), Alibaba Cloud'un geliştirdiği açık kaynaklı büyük dil modeli ailesidir. Çok dilli yetenekleri ve kod üretme kapasitesiyle öne çıkar.

| Özellik | Detay |
|---------|-------|
| **Merkez** | Hangzhou, Çin (Alibaba Group) |
| **Strateji** | Açık kaynak (Apache 2.0) |
| **Öne çıkan** | 29+ dil desteği, güçlü kodlama, Alibaba Cloud entegrasyonu |

### Model Ailesi

```mermaid
flowchart TD
    QW["Qwen"]

    QW --> Q35["Qwen 3.5\n(En Güncel)"]
    QW --> Q25["Qwen 2.5\n(Stabil)"]
    QW --> QC["Qwen2.5-Coder\n(Kod Uzmanı)"]
    QW --> QV["Qwen-VL\n(Vision-Language)"]

    Q35 --> Q35_72["72B"]
    Q35 --> Q35_122["122B ⭐\n(En büyük)"]

    Q25 --> Q25_7["7B"]
    Q25 --> Q25_32["32B"]
    Q25 --> Q25_72["72B"]

    QC --> QC_7["Coder 7B"]
    QC --> QC_32["Coder 32B ⭐\n(Kod benchmark lideri)"]

    style Q35_122 fill:#ff6b35,color:#fff
    style QC_32 fill:#ff6b35,color:#fff
```

### Qwen 3.5-122B

| Özellik | Değer |
|---------|-------|
| **Parametre** | 122 milyar |
| **Context window** | 128K token |
| **Dil desteği** | **29+ dil** (Çince, İngilizce, Türkçe dahil) |
| **Lisans** | Apache 2.0 (tam özgür) |

### Qwen2.5-Coder

Qwen2.5-Coder, özellikle kodlama için optimize edilmiş bir modeldir:

- **92+ programlama dili** desteği
- HumanEval benchmark'ında GPT-4o seviyesinde performans
- 32B versiyonu ile lokal çalıştırılabilir

### Pratik Kullanım

```python
# Ollama ile lokal Qwen kullanımı
# Terminal: ollama pull qwen2.5-coder:32b

from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"
)

response = client.chat.completions.create(
    model="qwen2.5-coder:32b",
    messages=[
        {"role": "user", "content": "React ile drag-and-drop kanban board oluştur."}
    ]
)
```

---

## Mistral

### Genel Bakış

Mistral AI, Fransa merkezli bir AI şirketidir. **Avrupa veri düzenlemelerine uyumluluk** (GDPR) konusunda öncü konumdadır ve Avrupa'nın en değerli AI girişimidir.

| Özellik | Detay |
|---------|-------|
| **Merkez** | Paris, Fransa |
| **Kuruluş** | 2023 (DeepMind ve Meta alumni tarafından) |
| **Strateji** | Karma (açık kaynak + kapalı kaynak modeller) |
| **Öne çıkan** | GDPR uyumluluk, Avrupa barındırma, MoE öncüsü |

### Model Ailesi

```mermaid
flowchart TD
    MI["Mistral AI"]

    MI --> ACIK["Açık Kaynak"]
    MI --> KAPALI["Kapalı Kaynak"]

    ACIK --> M7["Mistral 7B\n(İlk model)"]
    ACIK --> MIX["Mixtral 8x22B\n(MoE öncüsü)"]
    ACIK --> MS["Mistral Small\n(24B)"]

    KAPALI --> ML["Mistral Large 2\n(123B) ⭐"]
    KAPALI --> MC["Codestral\n(Kod uzmanı)"]
    KAPALI --> MN["Mistral NeMo\n(NVIDIA işbirliği)"]

    style ML fill:#7c3aed,color:#fff
```

### Mistral Large 2

| Özellik | Değer |
|---------|-------|
| **Parametre** | 123 milyar |
| **Context window** | 128K token |
| **Dil desteği** | İngilizce, Fransızca, Almanca, İspanyolca, İtalyanca + daha fazlası |
| **Öne çıkan** | GDPR uyumlu, Avrupa'da barındırma seçeneği |
| **Function calling** | Gelişmiş araç kullanımı |

### Codestral

Mistral'in kodlamaya özel modeli:

- 32K context window, 80+ programlama dili
- Fill-in-the-middle (FIM) desteği — IDE'ler için optimize
- VS Code ve JetBrains entegrasyonu

### Avrupa Uyumluluğu

Mistral'in en büyük farklılaştırıcısı, **Avrupa düzenlemelerine tam uyumluluk**tur:

| Düzenleme | Mistral Uyumluluğu |
|-----------|---------------------|
| **GDPR** | Veri Avrupa'da işlenir ve saklanır |
| **EU AI Act** | Uyumluluk çalışmaları devam ediyor |
| **Data residency** | Avrupa sınırları içinde kalma garantisi |
| **Veri gizliliği** | API verileri eğitim için kullanılmaz |

### API Fiyatlandırması

| Model | Input | Output |
|-------|-------|--------|
| Mistral Large 2 | $2.00 / 1M token | $6.00 / 1M token |
| Mistral Small | $0.20 / 1M token | $0.60 / 1M token |
| Codestral | $0.30 / 1M token | $0.90 / 1M token |

---

## Microsoft Phi

### Genel Bakış

Microsoft Research'ün geliştirdiği Phi serisi, **küçük ama güçlü** (Small Language Model / SLM) modelleriyle tanınır. "Büyük her zaman daha iyi değildir" felsefesiyle hareket eder.

| Özellik | Detay |
|---------|-------|
| **Merkez** | Redmond, ABD (Microsoft Research) |
| **Strateji** | Açık kaynak (MIT lisansı) |
| **Öne çıkan** | Küçük boyut, yüksek performans, reasoning |

### Model Ailesi

```mermaid
flowchart LR
    P1["Phi-1\n(1.3B)\n2023"]
    P2["Phi-2\n(2.7B)\n2023"]
    P3["Phi-3\n(3.8B-14B)\n2024"]
    P4["Phi-4\n(14B) ⭐\n2025"]

    P1 --> P2 --> P3 --> P4

    style P4 fill:#0078d4,color:#fff
```

### Phi-4

| Özellik | Değer |
|---------|-------|
| **Parametre** | 14 milyar |
| **Context window** | 16K token |
| **Lisans** | MIT |
| **Öne çıkan** | Reasoning, matematik, bilim |
| **Boyut** | ~8GB (quantized) |

Phi-4'ün dikkat çekici yanı, **14B parametre** ile bazı 70B+ modellere yakın matematik ve reasoning performansı göstermesidir.

### Kullanım Alanları

```mermaid
flowchart TD
    PHI["Phi-4 (14B)"]

    PHI --> EDGE["Edge Computing\n(Uç cihazlar)"]
    PHI --> MOB["Mobil Uygulamalar"]
    PHI --> EMB["Embedded Systems\n(Gömülü sistemler)"]
    PHI --> ED["Eğitim & Araştırma\n(Düşük donanım)"]

    EDGE --> EDGE1["Lokal çalıştırma\nİnternet gerektirmez"]
    MOB --> MOB1["Telefonda AI\nDüşük gecikme"]
```

---

## Cohere

### Genel Bakış

Cohere, Kanada merkezli bir AI şirketidir. **Kurumsal RAG (Retrieval-Augmented Generation)** ve metin analizi konusunda uzmanlaşmıştır.

| Özellik | Detay |
|---------|-------|
| **Merkez** | Toronto, Kanada |
| **Kuruluş** | 2019 (Google Brain alumni tarafından) |
| **Strateji** | Kurumsal odaklı, API tabanlı |
| **Öne çıkan** | RAG, Embedding, çok dilli arama |

### Ürünler

| Ürün | Açıklama |
|------|----------|
| **Command R+** | Ana dil modeli, 128K context, RAG optimize |
| **Embed v4** | Metin embedding modeli, 100+ dil |
| **Rerank** | Arama sonuçlarını sıralama modeli |

### Command R+ Özellikleri

- **128K token** context window
- **Grounded generation:** Yanıtlar kaynak belgelere referans verir
- **RAG native:** Harici belgeleri doğrudan işleyebilir
- **Çok dilli:** 10+ dilde güçlü performans

---

## 01.AI Yi

### Genel Bakış

01.AI, eski Google China başkanı **Kai-Fu Lee** tarafından kurulan Çin merkezli AI şirketidir.

| Özellik | Detay |
|---------|-------|
| **Merkez** | Pekin, Çin |
| **Kuruluş** | 2023 |
| **Strateji** | Açık kaynak (Apache 2.0) |
| **Öne çıkan** | Multimodal, çok dilli |

### Yi Modelleri

| Model | Parametre | Özellik |
|-------|-----------|---------|
| Yi-1.5-34B | 34B | Çok dilli, uzun bağlam |
| Yi-Lightning | Açıklanmadı | Hızlı çıkarım |
| Yi-Vision | 34B+ | Görsel anlama |

---

## Karşılaştırma Tablosu

```mermaid
flowchart LR
    subgraph acik ["Açık Kaynak"]
        DS2["DeepSeek\nMIT"]
        QW2["Qwen\nApache 2.0"]
        PH2["Phi\nMIT"]
        YI2["Yi\nApache 2.0"]
    end

    subgraph karma ["Karma"]
        MI2["Mistral\nAçık + Kapalı"]
    end

    subgraph kapali ["Kapalı / Kurumsal"]
        CO2["Cohere\nAPI only"]
    end
```

| Sağlayıcı | En Güçlü Model | Parametre | Context | Lisans | Fiyat (Output/1M) |
|-----------|----------------|-----------|---------|--------|---------------------|
| **DeepSeek** | R1 | 671B (MoE) | 128K | MIT | $2.19 |
| **Qwen** | 3.5-122B | 122B | 128K | Apache 2.0 | $1.50 |
| **Mistral** | Large 2 | 123B | 128K | Kapalı | $6.00 |
| **Microsoft** | Phi-4 | 14B | 16K | MIT | Ücretsiz (lokal) |
| **Cohere** | Command R+ | Açıklanmadı | 128K | Kapalı | $15.00 |
| **01.AI** | Yi-1.5-34B | 34B | 200K | Apache 2.0 | $0.80 |

---

## Hangi Durumda Hangisi?

| Senaryo | Önerilen Model | Neden? |
|---------|---------------|--------|
| **Bütçe kısıtlı reasoning** | DeepSeek-R1 | MIT lisans, çok düşük maliyet |
| **Çok dilli uygulama** | Qwen 3.5-122B | 29+ dil desteği |
| **Avrupa GDPR uyumluluğu** | Mistral Large 2 | Avrupa'da veri barındırma |
| **Mobil / edge cihaz** | Microsoft Phi-4 | 14B parametre, küçük boyut |
| **Kurumsal RAG** | Cohere Command R+ | RAG optimize, grounding |
| **Lokal kodlama asistanı** | Qwen2.5-Coder 32B | Açık kaynak, güçlü kodlama |
| **Araştırma & deneysel** | DeepSeek-V3.2 | Düşük maliyet, geniş yetenek |

---

## Özet

Bu bölümde incelediğimiz sağlayıcılar, büyük dil modelleri ekosisteminin çeşitliliğini göstermektedir:

- **DeepSeek:** Açık kaynak reasoning'de devrim — MIT lisansı ile en serbest model
- **Qwen:** Çok dilli yeteneklerde lider — 29+ dil, güçlü kodlama
- **Mistral:** Avrupa'nın AI şampiyonu — GDPR uyumlu, kurumsal odaklı
- **Microsoft Phi:** "Küçük ama güçlü" felsefesi — edge ve mobil uygulamalar için
- **Cohere:** Kurumsal RAG uzmanı — belge tabanlı AI uygulamaları için
- **01.AI Yi:** Çin'den çok dilli ve multimodal açık kaynak seçenek

---

## Sonraki Adım

Tüm sağlayıcıları tanıdık. Şimdi hepsini bir arada karşılaştıran detaylı benchmark tabloları ve kullanım senaryolarını inceleyelim:

→ [Büyük Karşılaştırma](./06-buyuk-karsilastirma.md)
