# Bölüm 03: LLM Sağlayıcıları ve Karşılaştırma

Büyük dil modellerini geliştiren şirketleri, stratejilerini, model ailelerini ve birbirleriyle karşılaştırmalarını inceliyoruz. Bu bölüm, doğru model seçiminde bilinçli kararlar vermenizi sağlayacak.

## Bu Bölümde Neler Öğreneceksiniz?

```mermaid
flowchart TD
    A["OpenAI<br/>GPT & o-serisi"] --> B["Anthropic<br/>Claude & AI Safety"]
    B --> C["Google DeepMind<br/>Gemini"]
    C --> D["Meta & Llama<br/>Açık Kaynak"]
    D --> E["Diğer Sağlayıcılar<br/>DeepSeek, Mistral, Qwen..."]
    E --> F["Büyük Karşılaştırma<br/>Benchmark & Fiyat"]
```

## İçerik

| # | Dosya | Konu | Süre |
|---|-------|------|------|
| 01 | [OpenAI](./01-openai.md) | Şirket tarihi, GPT-5.x serisi, o-serisi reasoning modeller, API, fiyatlandırma | ~15 dk |
| 02 | [Anthropic](./02-anthropic.md) | AI Safety felsefesi, Constitutional AI, Claude model serisi, sorumlu ölçekleme | ~15 dk |
| 03 | [Google DeepMind](./03-google-deepmind.md) | DeepMind birleşmesi, Gemini serisi, 2M context window, multimodal | ~12 dk |
| 04 | [Meta ve Llama](./04-meta-ve-llama.md) | Açık kaynak stratejisi, Llama evrimi, Scout & Maverick, ekosistem | ~12 dk |
| 05 | [Diğer Sağlayıcılar](./05-diger-saglayicilar.md) | DeepSeek, Qwen, Mistral, Microsoft Phi, Cohere, 01.AI Yi | ~15 dk |
| 06 | [Büyük Karşılaştırma](./06-buyuk-karsilastirma.md) | Benchmark tabloları, fiyat karşılaştırma, kullanım senaryoları | ~20 dk |

## Sağlayıcı Haritası

```mermaid
quadrantChart
    title LLM Sağlayıcıları - Strateji Haritası
    x-axis "Kapalı Kaynak" --> "Açık Kaynak"
    y-axis "Niş Odak" --> "Geniş Platform"
    quadrant-1 "Açık Platform"
    quadrant-2 "Kapalı Platform"
    quadrant-3 "Kapalı & Özelleşmiş"
    quadrant-4 "Açık & Özelleşmiş"
    OpenAI: [0.15, 0.85]
    Anthropic: [0.2, 0.6]
    Google: [0.25, 0.95]
    Meta: [0.9, 0.8]
    DeepSeek: [0.95, 0.45]
    Mistral: [0.7, 0.5]
    Qwen: [0.8, 0.65]
    Microsoft: [0.55, 0.9]
```

## Ön Koşullar

- [Bölüm 02 - Büyük Dil Modelleri](../02-buyuk-dil-modelleri/README.md)

## Sonraki Adım

→ [Bölüm 04 - Yapay Zeka Destekli Yazılım Geliştirme](../04-ai-destekli-gelistirme/README.md)
