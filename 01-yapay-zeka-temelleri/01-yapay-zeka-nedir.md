# Yapay Zeka Nedir?

Artificial Intelligence (yapay zeka), makinelerin insan zekasını taklit ederek öğrenme, problem çözme, karar verme ve dil anlama gibi görevleri yerine getirmesini sağlayan bir bilim dalıdır.

## Ön Koşullar

Yok. Bu, rehberin başlangıç noktasıdır.

---

## Yapay Zeka Günlük Hayatta

Yapay zekayı muhtemelen her gün kullanıyorsunuz, farkında olmasanız bile:

| Uygulama | Ne Yapıyor? | Kullandığı AI Türü |
|----------|-------------|---------------------|
| **Google Haritalar** | Trafik tahmini, en hızlı rota | Prediction (tahmin) |
| **Netflix / Spotify** | Size özel içerik önerisi | Recommendation (öneri sistemi) |
| **Siri / Google Asistan** | Sesli komut anlama | Speech Recognition (konuşma tanıma) |
| **Gmail** | Spam filtreleme, otomatik yanıt önerileri | Classification (sınıflandırma) |
| **ChatGPT / Claude** | Metin oluşturma, soru cevaplama | Natural Language Generation (doğal dil üretimi) |
| **Tesla Autopilot** | Otonom sürüş | Computer Vision (bilgisayarlı görü) |

---

## AI'nin Kısa Tarihçesi

```mermaid
timeline
    title Artificial Intelligence Tarihçesi
    1950 : Alan Turing - "Turing Test"
         : Makine düşünebilir mi sorusu
    1956 : Dartmouth Konferansı
         : "Artificial Intelligence" terimi ilk kez kullanıldı
    1997 : IBM Deep Blue
         : Satranç şampiyonu Kasparov'u yendi
    2011 : IBM Watson
         : Jeopardy yarışmasını kazandı
    2012 : Deep Learning devrimi
         : ImageNet yarışmasında büyük başarı
    2017 : Transformer mimarisi
         : "Attention Is All You Need" makalesi
    2022 : ChatGPT
         : AI herkesin gündemine girdi
    2024 : AI Agent'lar
         : Otonom kodlama araçları yaygınlaştı
    2026 : Günümüz
         : GPT-5.4, Claude 4.6, Gemini 3.1
```

---

## AI'nin Alt Dalları

Yapay zeka geniş bir şemsiye kavramdır. Altında birçok alt dal bulunur:

```mermaid
mindmap
  root((Artificial Intelligence))
    Machine Learning
      Supervised Learning
      Unsupervised Learning
      Reinforcement Learning
      Deep Learning
        CNN
        RNN
        Transformer
    Natural Language Processing
      Text Classification
      Sentiment Analysis
      Machine Translation
      Text Generation
    Computer Vision
      Image Recognition
      Object Detection
      Face Recognition
    Robotics
      Motion Planning
      Manipulation
    Expert Systems
      Rule-based Systems
      Knowledge Graphs
```

### Önemli alt dallar:

- **Machine Learning (makine öğrenimi):** Makinelerin veriden öğrenmesi
- **Deep Learning (derin öğrenme):** Çok katmanlı sinir ağları ile öğrenme
- **Natural Language Processing / NLP (doğal dil işleme):** İnsan dilini anlama ve üretme
- **Computer Vision (bilgisayarlı görü):** Görüntü ve video analizi
- **Robotics (robotik):** Fiziksel dünyada hareket ve etkileşim

> **Bu rehberin odağı:** NLP ve Deep Learning kesişiminde yer alan **Large Language Model (büyük dil modelleri)** konusu olacaktır. Çünkü Claude Code, bir LLM (Claude) üzerine inşa edilmiş bir araçtır.

---

## AI Ne Yapabilir, Ne Yapamaz?

### Yapabilir

- Büyük veri setlerinde örüntü (pattern) bulma
- Metin, kod, görsel üretme
- Dil çevirisi yapma
- Tahmin ve sınıflandırma
- İnsan dilini anlama ve yanıtlama
- Tekrarlayan görevleri otomatikleştirme

### Yapamaz (Henüz)

- Gerçek anlamda "anlamak" veya "düşünmek"
- Eğitim verisi dışında yeni bilgi "bilmek"
- %100 doğruluk garantisi vermek
- Yaratıcı bilinç veya duygusal zeka göstermek
- Etik kararları bağımsız olarak almak

> **Önemli:** AI modelleri, eğitim verilerindeki kalıpları istatistiksel olarak modeller. "Bilmek" veya "anlamak" yerine "tahmin etmek" daha doğru bir ifadedir.

---

## Neden Yazılımcılar İçin Önemli?

Yapay zeka, yazılım geliştirme sürecini kökten değiştiriyor:

```mermaid
flowchart LR
    subgraph once ["Geleneksel Geliştirme"]
        A1["Gereksinim\nAnalizi"] --> A2["Tasarım"]
        A2 --> A3["Kodlama\n(elle)"]
        A3 --> A4["Test\n(elle)"]
        A4 --> A5["Hata Ayıklama\n(elle)"]
    end

    subgraph simdi ["AI Destekli Geliştirme"]
        B1["Gereksinim\nAnalizi"] --> B2["AI ile\nTasarım"]
        B2 --> B3["AI ile\nKodlama"]
        B3 --> B4["AI ile\nTest Üretme"]
        B4 --> B5["AI ile\nHata Tespiti"]
    end

    once -.->|"Evrim"| simdi
```

**Somut etkiler:**

- **Kod yazma hızı:** AI destekli araçlarla 2-5x daha hızlı prototipleme
- **Bug tespiti:** Kod inceleme sürecinde AI ile otomatik hata yakalama
- **Dokümantasyon:** Otomatik API dokümantasyonu ve README üretimi
- **Test:** AI ile test senaryosu ve test kodu üretimi
- **Refactoring:** Büyük ölçekli kod yeniden yapılandırma önerileri

---

## Örnek: AI Bir Soruyu Nasıl Yanıtlar?

Basitleştirilmiş bir akış:

```mermaid
sequenceDiagram
    participant K as Kullanıcı
    participant AI as AI Modeli
    participant V as Veri/Bilgi

    K->>AI: "Türkiye'nin başkenti neresidir?"
    AI->>V: Eğitim verisinde arama
    V-->>AI: İstatistiksel olarak en olası yanıt
    AI-->>K: "Türkiye'nin başkenti Ankara'dır."
    
    Note over AI: Model gerçekten "bilmiyor".<br/>Eğitim verisindeki kalıplara<br/>göre en olası yanıtı üretiyor.
```

Bu basit örnek bile önemli bir gerçeği gösterir: AI modelleri bir veritabanından bilgi çekmez. Eğitim sırasında öğrendiği istatistiksel kalıplara göre en olası devam eden metni üretir.

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Artificial Intelligence** | Makinelerin insan zekasını taklit etmesi |
| **Machine Learning** | Veriden öğrenme yeteneği |
| **Deep Learning** | Çok katmanlı sinir ağları |
| **NLP** | İnsan dilini anlama ve üretme |
| **Model** | Veriden öğrenilmiş kalıpları saklayan matematiksel yapı |

---

## Sonraki Adım

Yapay zekanın ne olduğunu anladık. Şimdi Machine Learning ve Deep Learning kavramlarını daha derinlemesine inceleyelim:

→ [Machine Learning ve Deep Learning](./02-makine-ogrenimi-ve-derin-ogrenme.md)
