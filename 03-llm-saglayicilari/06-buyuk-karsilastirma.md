# Büyük Karşılaştırma: Tüm LLM Modelleri

Bu bölüm, önceki dosyalarda incelediğimiz tüm büyük dil modellerini benchmark (kıyaslama) sonuçları, fiyatlandırma, context window ve kullanım senaryoları açısından kapsamlı şekilde karşılaştırmaktadır. Mart 2026 verileri baz alınmıştır.

## Ön Koşullar

- [LLM Değerlendirme Kriterleri](../02-buyuk-dil-modelleri/05-llm-degerlendirme-kriterleri.md)
- Bu bölümdeki önceki dosyalar (01-05)

---

## Amiral Gemisi Modeller — Genel Bakış

```mermaid
flowchart TD
    subgraph kapali ["Kapalı Kaynak"]
        GPT5["GPT-5.4<br/>(OpenAI)"]
        CLAUDE["Claude 4.6 Opus<br/>(Anthropic)"]
        GEM["Gemini 3.1 Pro<br/>(Google)"]
    end

    subgraph acik ["Açık Kaynak / Ağırlıklar"]
        LLAMA["Llama 4 Maverick<br/>(Meta)"]
        DS["DeepSeek-R1<br/>(DeepSeek)"]
        QW["Qwen 3.5-122B<br/>(Alibaba)"]
        MI["Mistral Large 2<br/>(Mistral)"]
    end

    subgraph kucuk ["Küçük / Verimli"]
        PHI["Phi-4 (14B)<br/>(Microsoft)"]
        FLASH["Gemini 3.1 Flash<br/>(Google)"]
        HAIKU["Claude 4 Haiku<br/>(Anthropic)"]
        MINI["GPT-4o-mini<br/>(OpenAI)"]
    end
```

---

## Benchmark Karşılaştırmaları

### MMLU (Massive Multitask Language Understanding)

MMLU, 57 farklı akademik alanda (matematik, tarih, hukuk, tıp vb.) genel bilgiyi ölçen bir benchmark'tır.

| Model | MMLU Skoru | Kategori |
|-------|-----------|----------|
| GPT-5.4 | %91.2 | Kapalı kaynak |
| Claude 4.6 Opus | %90.8 | Kapalı kaynak |
| Gemini 3.1 Pro | %90.5 | Kapalı kaynak |
| DeepSeek-R1 | %90.2 | Açık kaynak |
| Qwen 3.5-122B | %88.5 | Açık kaynak |
| Llama 4 Maverick | %88.1 | Açık kaynak |
| Mistral Large 2 | %86.3 | Kapalı kaynak |
| Claude 4.6 Sonnet | %88.7 | Kapalı kaynak |
| Phi-4 (14B) | %82.5 | Açık kaynak (küçük) |

```mermaid
xychart-beta
    title "MMLU Benchmark Sonuçları (%)"
    x-axis ["GPT-5.4", "Claude Opus", "Gemini Pro", "DeepSeek R1", "Qwen 122B", "Llama Mav.", "Mistral L2", "Phi-4"]
    y-axis "Skor (%)" 75 --> 95
    bar [91.2, 90.8, 90.5, 90.2, 88.5, 88.1, 86.3, 82.5]
```

### HumanEval (Kod Üretme)

HumanEval, modelin Python programlama problemlerini çözme yeteneğini ölçer. Yazılım geliştiriciler için en kritik benchmark'tır.

| Model | HumanEval (pass@1) | Not |
|-------|---------------------|-----|
| Claude 4.6 Opus | %95.2 | Kodlama lideri |
| Claude 4.6 Sonnet | %94.1 | Fiyat/performans şampiyonu |
| GPT-5.4 | %93.8 | Güçlü rakip |
| DeepSeek-R1 | %92.5 | Açık kaynak lideri |
| Gemini 3.1 Pro | %93.1 | Yakın takip |
| o3 (reasoning) | %96.2 | Reasoning ile en yüksek |
| Qwen2.5-Coder 32B | %91.8 | Açık kaynak kod uzmanı |
| Llama 4 Maverick | %89.5 | Açık kaynak |
| Mistral Large 2 | %87.2 | Orta segment |
| Phi-4 (14B) | %80.3 | Boyutuna göre etkileyici |

```mermaid
xychart-beta
    title "HumanEval Kod Üretme Başarısı (%)"
    x-axis ["o3", "Claude Opus", "Claude Son.", "GPT-5.4", "Gemini Pro", "DS-R1", "Qwen Cod.", "Llama Mav.", "Phi-4"]
    y-axis "pass@1 (%)" 75 --> 100
    bar [96.2, 95.2, 94.1, 93.8, 93.1, 92.5, 91.8, 89.5, 80.3]
```

### SWE-bench (Gerçek Dünya Yazılım Mühendisliği)

SWE-bench, gerçek GitHub issue'larını çözme yeteneğini ölçer. En pratik kodlama benchmark'ıdır.

| Model | SWE-bench Verified | Not |
|-------|---------------------|-----|
| Claude 4.6 Sonnet | %72.5 | Claude Code'un motoru |
| Claude 4.6 Opus | %70.8 | Derin analiz |
| o3 | %69.3 | Reasoning avantajı |
| GPT-5.4 | %67.1 | Güçlü genel yetenek |
| DeepSeek-R1 | %65.8 | Açık kaynak en iyi |
| Gemini 3.1 Pro | %64.2 | Gelişen performans |
| Llama 4 Maverick | %58.5 | Açık kaynak |

> **Neden Claude Sonnet, Opus'tan yüksek?** SWE-bench'te Sonnet'in daha iyi performans göstermesi, bu modelin özellikle kodlama görevleri için optimize edilmesiyle açıklanır. Opus daha geniş bir yetenek yelpazesine sahiptir.

### MATH-500 (Matematik)

MATH-500, çok adımlı matematik problemlerini çözme yeteneğini ölçer.

| Model | MATH-500 (%) | Not |
|-------|-------------|-----|
| o3 | %97.8 | Reasoning modeli |
| DeepSeek-R1 | %96.3 | Reasoning modeli |
| o4-mini | %94.5 | Hızlı reasoning |
| GPT-5.4 | %92.1 | Genel model |
| Claude 4.6 Opus | %91.5 | Genel model |
| Gemini 3.1 Pro | %91.2 | Genel model |
| Qwen 3.5-122B | %88.7 | Açık kaynak |
| Phi-4 (14B) | %86.2 | Boyutuna göre çok iyi |

```mermaid
xychart-beta
    title "MATH-500 Başarı Oranları (%)"
    x-axis ["o3", "DS-R1", "o4-mini", "GPT-5.4", "Claude Opus", "Gemini Pro", "Qwen 122B", "Phi-4"]
    y-axis "Skor (%)" 80 --> 100
    bar [97.8, 96.3, 94.5, 92.1, 91.5, 91.2, 88.7, 86.2]
```

---

## Fiyatlandırma Karşılaştırması

### Amiral Gemisi Modeller (Output Fiyatı / 1M Token)

| Model | Input | Output | Output/Input Oranı |
|-------|-------|--------|---------------------|
| Claude 4.6 Opus | $15.00 | $75.00 | 5x |
| o3 | $10.00 | $40.00 | 4x |
| GPT-5 | $10.00 | $30.00 | 3x |
| Claude 4.6 Sonnet | $3.00 | $15.00 | 5x |
| Gemini 3.1 Pro | $3.50 | $10.50 | 3x |
| Mistral Large 2 | $2.00 | $6.00 | 3x |
| DeepSeek-R1 | $0.55 | $2.19 | 4x |
| Qwen 3.5-122B | $0.50 | $1.50 | 3x |

```mermaid
xychart-beta
    title "Output Fiyatı Karşılaştırması ($/1M token)"
    x-axis ["Claude Opus", "o3", "GPT-5", "Claude Son.", "Gemini Pro", "Mistral L2", "DS-R1", "Qwen 122B"]
    y-axis "Fiyat ($)" 0 --> 80
    bar [75, 40, 30, 15, 10.5, 6, 2.19, 1.5]
```

### Hızlı / Hafif Modeller (Output Fiyatı / 1M Token)

| Model | Input | Output |
|-------|-------|--------|
| Claude 4 Haiku | $0.80 | $4.00 |
| GPT-4o-mini | $0.15 | $0.60 |
| Gemini 3.1 Flash | $0.075 | $0.30 |
| DeepSeek-V3.2 | $0.27 | $1.10 |
| o4-mini | $1.10 | $4.40 |
| Mistral Small | $0.20 | $0.60 |

### Fiyat/Performans Analizi

```mermaid
quadrantChart
    title Fiyat vs Performans (MMLU)
    x-axis "Düşük Fiyat" --> "Yüksek Fiyat"
    y-axis "Düşük Performans" --> "Yüksek Performans"
    quadrant-1 "Premium"
    quadrant-2 "En İyi Değer"
    quadrant-3 "Bütçe"
    quadrant-4 "Pahalı & Zayıf"
    DeepSeek-R1: [0.10, 0.88]
    Qwen-122B: [0.08, 0.82]
    Claude-Sonnet: [0.35, 0.85]
    Gemini-Pro: [0.30, 0.87]
    GPT-5: [0.55, 0.90]
    Claude-Opus: [0.85, 0.92]
    o3: [0.70, 0.93]
    Mistral-L2: [0.25, 0.78]
    Phi-4: [0.02, 0.65]
```

> **En iyi fiyat/performans:** DeepSeek-R1 ve Claude 4.6 Sonnet, farklı bütçe segmentlerinde en iyi fiyat/performans oranını sunar.

---

## Context Window Karşılaştırması

| Model | Context Window | Yaklaşık Sayfa | Pratik Kullanım |
|-------|----------------|----------------|-----------------|
| Llama 4 Scout | **10.000.000** | ~15.000 sayfa | Tüm kod tabanı analizi (deneysel) |
| Gemini 3.1 Pro | **2.000.000** | ~3.000 sayfa | Kitap boyutunda doküman işleme |
| Llama 4 Maverick | **1.000.000** | ~1.500 sayfa | Büyük proje analizi |
| GPT-5 / GPT-5.4 | 256.000 | ~385 sayfa | Geniş bağlam, çoklu dosya |
| Claude 4.6 Opus | 200.000 | ~300 sayfa | Proje bazlı analiz |
| Claude 4.6 Sonnet | 200.000 | ~300 sayfa | Kodlama ve analiz |
| DeepSeek-R1 | 128.000 | ~190 sayfa | Standart kullanım |
| Qwen 3.5-122B | 128.000 | ~190 sayfa | Standart kullanım |
| Mistral Large 2 | 128.000 | ~190 sayfa | Standart kullanım |
| Phi-4 | 16.000 | ~24 sayfa | Kısa bağlam görevleri |

```mermaid
xychart-beta
    title "Context Window Karşılaştırması (K token, logaritmik değil)"
    x-axis ["Phi-4", "DS/Qwen/Mistral", "Claude 4.6", "GPT-5", "Llama Mav.", "Gemini Pro", "Llama Scout"]
    y-axis "Token (K)" 0 --> 10100
    bar [16, 128, 200, 256, 1000, 2000, 10000]
```

---

## Kullanım Senaryosuna Göre Model Önerileri

### Yazılım Geliştirme

```mermaid
flowchart TD
    Q["Yazılım geliştirme<br/>için model seçimi"]

    Q --> Q1{"Bütçe<br/>durumu?"}

    Q1 -->|"Sınırsız"| A1["Claude 4.6 Opus<br/>+ Claude Code<br/>En iyi kodlama deneyimi"]
    Q1 -->|"Orta"| A2["Claude 4.6 Sonnet<br/>En iyi fiyat/performans"]
    Q1 -->|"Kısıtlı"| Q2{"Gizlilik<br/>gereksinimi?"}

    Q2 -->|"Veri dışarı çıkamaz"| A3["Llama 4 Maverick<br/>(lokal) veya<br/>Qwen2.5-Coder"]
    Q2 -->|"Sorun değil"| A4["DeepSeek-R1<br/>En ucuz reasoning"]

```

### Genel Amaçlı Kullanım

| Senaryo | Birincil Öneri | Alternatif | Neden? |
|---------|---------------|------------|--------|
| **Günlük soru-cevap** | ChatGPT (GPT-5) | Claude.ai | Geniş bilgi, hızlı yanıt |
| **Akademik araştırma** | Claude 4.6 Opus | o3 | Derin analiz, uzun yanıtlar |
| **Kod yazma (profesyonel)** | Claude 4.6 Sonnet + Claude Code | Codex CLI | SWE-bench lideri |
| **Kod yazma (hobi/öğrenme)** | DeepSeek-R1 | Qwen2.5-Coder | Ücretsiz/çok ucuz |
| **Matematik / bilim** | o3 | DeepSeek-R1 | Reasoning modelleri üstün |
| **Uzun doküman analizi** | Gemini 3.1 Pro | Llama 4 Scout | 2M context window |
| **Çok dilli içerik** | Qwen 3.5-122B | Gemini 3.1 Pro | 29+ dil desteği |
| **Görsel analiz** | Gemini 3.1 Pro | GPT-5.4 | Doğuştan multimodal |
| **Video analiz** | Gemini 3.1 Pro | — | Tek gerçek seçenek |
| **GDPR uyumlu kurumsal** | Mistral Large 2 | Cohere Command R+ | Avrupa'da veri barındırma |
| **Edge / mobil** | Phi-4 (14B) | Gemini Nano | Küçük boyut, lokal çalışma |
| **RAG uygulamaları** | Cohere Command R+ | Claude Sonnet | RAG optimize |
| **Maliyet optimizasyonu** | DeepSeek-V3.2 | Gemini Flash | En düşük token fiyatı |

### Kurumsal Kullanım

```mermaid
flowchart TD
    K["Kurumsal Model Seçimi"]

    K --> K1{"Veri gizliliği<br/>önceliği?"}

    K1 -->|"Maksimum"| K2["Lokal Deployment"]
    K1 -->|"Standart"| K3{"Konum?"}

    K2 --> K2A["Llama 4 Maverick<br/>(kendi sunucunuz)"]
    K2 --> K2B["Qwen 3.5-122B<br/>(açık kaynak)"]

    K3 -->|"Avrupa"| K3A["Mistral Large 2<br/>(GDPR uyumlu)"]
    K3 -->|"Global"| K4{"Kullanım hacmi?"}

    K4 -->|"Yüksek"| K4A["Claude Sonnet<br/>(API)"]
    K4 -->|"Düşük/Orta"| K4B["GPT-5<br/>(Azure OpenAI)"]

```

---

## Çok Boyutlu Karşılaştırma

### Yetenek Radar Analizi

Aşağıdaki tablo, her modelin farklı alanlardaki göreceli gücünü 10 üzerinden göstermektedir:

| Yetenek | GPT-5.4 | Claude 4.6 Opus | Gemini 3.1 Pro | DeepSeek-R1 | Llama 4 Mav. |
|---------|---------|-----------------|----------------|-------------|-------------|
| **Kodlama** | 9.3 | **9.7** | 9.0 | 9.1 | 8.5 |
| **Reasoning** | 9.0 | 9.2 | 9.0 | **9.5** | 8.0 |
| **Yaratıcı yazım** | **9.5** | 9.4 | 8.5 | 7.5 | 8.0 |
| **Çok dilli** | 9.0 | 8.5 | **9.5** | 8.0 | 8.5 |
| **Multimodal** | 9.0 | 8.0 | **9.8** | 7.0 | 7.5 |
| **Context uzunluğu** | 7.5 | 7.0 | **9.5** | 6.5 | 8.5 |
| **Fiyat/performans** | 6.0 | 5.5 | 8.0 | **9.5** | 8.5 |
| **Güvenlik** | 8.5 | **9.5** | 8.0 | 7.5 | 7.0 |
| **Ekosistem** | **9.5** | 8.5 | 9.0 | 6.0 | 8.0 |

### Sonuç Yorumları

```mermaid
flowchart TD
    subgraph kazananlar ["Alan Şampiyonları"]
        K1["🏆 Kodlama<br/>Claude 4.6 Opus/Sonnet"]
        K2["🏆 Reasoning<br/>DeepSeek-R1 / o3"]
        K3["🏆 Multimodal<br/>Gemini 3.1 Pro"]
        K4["🏆 Fiyat/Performans<br/>DeepSeek-R1"]
        K5["🏆 Context Window<br/>Gemini 3.1 Pro (2M)"]
        K6["🏆 Güvenlik<br/>Claude (Constitutional AI)"]
        K7["🏆 Açık Kaynak<br/>Llama 4 / DeepSeek"]
        K8["🏆 Ekosistem<br/>OpenAI (ChatGPT)"]
    end
```

---

## Model Seçim Rehberi — Özet Akış

```mermaid
flowchart TD
    START["Model Seçimi<br/>Başla"]

    START --> Q1{"Birincil<br/>kullanım alanı?"}

    Q1 -->|"Kodlama"| CODE["Claude 4.6 Sonnet<br/>+ Claude Code"]
    Q1 -->|"Genel amaçlı"| GEN{"Bütçe?"}
    Q1 -->|"Araştırma/Analiz"| RES{"Reasoning<br/>gerekli mi?"}
    Q1 -->|"Multimodal"| MULTI["Gemini 3.1 Pro"]
    Q1 -->|"Lokal/Gizlilik"| LOCAL["Llama 4 veya<br/>Qwen 3.5"]

    GEN -->|"Yüksek"| GPT["GPT-5.4 veya<br/>Claude Opus"]
    GEN -->|"Orta"| SONNET["Claude Sonnet veya<br/>Gemini Pro"]
    GEN -->|"Düşük"| CHEAP["DeepSeek-V3.2 veya<br/>Gemini Flash"]

    RES -->|"Evet"| REASON["o3 veya<br/>DeepSeek-R1"]
    RES -->|"Hayır"| OPUS["Claude 4.6 Opus"]

```

---

## Önemli Uyarılar

1. **Benchmark ≠ Gerçek Dünya:** Benchmark sonuçları kontrollü ortamlarda elde edilir. Gerçek kullanımda performans farklılık gösterebilir.

2. **Fiyatlar değişir:** Token fiyatları sık güncellenir. Güncel fiyatlar için her sağlayıcının resmi sitesini kontrol edin.

3. **Bir model hepsinde en iyi olamaz:** Her model farklı alanlarda öne çıkar. İhtiyacınıza göre seçim yapın.

4. **Açık kaynak ≠ Ücretsiz çalıştırma:** Açık kaynak modeller ücretsiz indirilebilir ancak çalıştırmak için GPU donanımı gerekir. Alternatif olarak, API sağlayıcıları (Together AI, Groq, AWS Bedrock) üzerinden kullanılabilir.

5. **Context window ≠ Kaliteli context kullanımı:** Büyük context window'a sahip olmak, modelin o bağlamdaki tüm bilgiyi eşit kalitede işleyeceği anlamına gelmez.

---

## Özet Tablosu

| Model | Şirket | Parametre | Context | MMLU | HumanEval | Output Fiyat |
|-------|--------|-----------|---------|------|-----------|-------------|
| GPT-5.4 | OpenAI | Gizli | 256K | %91.2 | %93.8 | $30 |
| o3 | OpenAI | Gizli | 200K | — | %96.2 | $40 |
| Claude 4.6 Opus | Anthropic | Gizli | 200K | %90.8 | %95.2 | $75 |
| Claude 4.6 Sonnet | Anthropic | Gizli | 200K | %88.7 | %94.1 | $15 |
| Gemini 3.1 Pro | Google | Gizli | 2M | %90.5 | %93.1 | $10.50 |
| Llama 4 Maverick | Meta | 400B | 1M | %88.1 | %89.5 | Ücretsiz* |
| DeepSeek-R1 | DeepSeek | 671B | 128K | %90.2 | %92.5 | $2.19 |
| Qwen 3.5-122B | Alibaba | 122B | 128K | %88.5 | %91.8 | $1.50 |
| Mistral Large 2 | Mistral | 123B | 128K | %86.3 | %87.2 | $6.00 |
| Phi-4 | Microsoft | 14B | 16K | %82.5 | %80.3 | Ücretsiz* |

*\* Açık kaynak modeller lokal çalıştırıldığında token başına ücret yoktur ancak donanım maliyeti vardır.*

---

## Bu Rehberin Odağı

Bu rehber bir **Claude Code el kitabı** olduğu için, bundan sonraki bölümlerde **Claude 4.6 Sonnet** (Claude Code'un varsayılan motoru) ve **Anthropic ekosistemi** üzerine yoğunlaşılacaktır.

Ancak Claude Code, model seçimini destekler — Gemini, GPT veya diğer modeller de kullanılabilir.

---

## Sonraki Adım

Tüm sağlayıcıları ve modellerini karşılaştırdık. Artık yapay zeka destekli yazılım geliştirme dünyasına adım atabiliriz:

→ [Bölüm 04 - Yapay Zeka Destekli Yazılım Geliştirme](../04-ai-destekli-gelistirme/README.md)

---

**← Önceki Bölüm:** [Bölüm 02 - Büyük Dil Modelleri](../02-buyuk-dil-modelleri/README.md)
