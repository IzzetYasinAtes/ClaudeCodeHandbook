# Bölüm 05: Claude Ekosistemi

Bu bölüm, Anthropic'in geliştirdiği Claude AI'ı tanıtır. Claude'un ne olduğunu, model ailesini, API/SDK kullanımını ve rakiplerinden ayıran özel yeteneklerini detaylı olarak inceler.

## Bu Bölümde Neler Öğreneceksiniz?

```mermaid
flowchart TD
    A["Claude Nedir?"] --> B["Claude Model Ailesi<br/>(Haiku / Sonnet / Opus)"]
    B --> C["API ve SDK<br/>Kullanımı"]
    C --> D["Claude'un Özel<br/>Yetenekleri"]
```

## İçerik

| # | Dosya | Konu | Süre |
|---|-------|------|------|
| 01 | [Claude Nedir?](./01-claude-nedir.md) | Claude AI tanımı, temel yetenekler, güvenlik felsefesi, Constitutional AI | ~12 dk |
| 02 | [Claude Model Ailesi](./02-claude-model-ailesi.md) | Haiku, Sonnet, Opus modelleri, sürümler, Extended Thinking, fiyatlandırma | ~15 dk |
| 03 | [API ve SDK Kullanımı](./03-claude-api-ve-sdk.md) | Messages API, Python SDK, TypeScript SDK, streaming, system prompt | ~20 dk |
| 04 | [Claude'un Özel Yetenekleri](./04-claude-ozel-yetenekleri.md) | Düşük hallucination, kodlama liderliği, 200K context, vision, artifacts, tool use | ~15 dk |

## Ön Koşullar

- [Bölüm 02 - Büyük Dil Modelleri](../02-buyuk-dil-modelleri/README.md)
- [Bölüm 04 - AI Destekli Yazılım Geliştirme](../04-ai-destekli-gelistirme/README.md)

## Bu Bölüm Kimler İçin?

```mermaid
flowchart LR
    subgraph hedef ["Hedef Okuyucular"]
        A["LLM kavramını<br/>bilen herkes"]
        B["Claude kullanmak<br/>isteyen geliştiriciler"]
        C["AI araçlarını<br/>karşılaştıranlar"]
    end
```

| Birim | Bu Bölümden Ne Öğrenir? |
|-------|------------------------|
| **Teknik** | Claude'un kodlama yetenekleri, API entegrasyonu, hangi model ne zaman kullanılır |
| **Ürün & Analiz** | Claude'un analiz ve metin yetenekleri, model seçimi, rakiplerle karşılaştırma |
| **Ticari** | Claude'un doğal dil yetenekleri, içerik üretimi ve veri analizi kapasitesi |
| **Operasyon** | Claude modelleri arasındaki farklar, maliyet optimizasyonu, kurumsal kullanım |

## Sonraki Adım

Bu bölümü tamamladıktan sonra → [06 - Claude Code: Tanıtım ve Kurulum](../06-claude-code-tanitim/README.md)
