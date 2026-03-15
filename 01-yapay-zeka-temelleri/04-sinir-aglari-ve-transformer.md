# Sinir Ağları ve Transformer Mimarisi

Artificial Neural Network (yapay sinir ağı), insan beynindeki nöronlardan esinlenerek tasarlanmış matematiksel bir modeldir. Transformer ise 2017'de ortaya çıkan ve günümüz LLM'lerinin temelini oluşturan devrim niteliğinde bir sinir ağı mimarisidir.

## Ön Koşullar

- [Yapay Zeka Nedir?](./01-yapay-zeka-nedir.md)
- [Machine Learning ve Deep Learning](./02-makine-ogrenimi-ve-derin-ogrenme.md)
- [Natural Language Processing](./03-dogal-dil-isleme.md)

---

## Neural Network (Sinir Ağı) Nedir?

Bir sinir ağı, birbirine bağlı "nöron" adı verilen hesaplama birimlerinden oluşur. Her nöron girdileri alır, bir işlem uygular ve çıktı üretir.

### Tek Bir Nöron

```mermaid
flowchart LR
    X1["Girdi 1\n(x₁)"] -->|"Ağırlık (w₁)"| N["Nöron\nΣ + Aktivasyon"]
    X2["Girdi 2\n(x₂)"] -->|"Ağırlık (w₂)"| N
    X3["Girdi 3\n(x₃)"] -->|"Ağırlık (w₃)"| N
    B["Bias\n(b)"] --> N
    N --> Y["Çıktı\n(y)"]
```

**Formül:** `y = aktivasyon(w₁·x₁ + w₂·x₂ + w₃·x₃ + b)`

- **Weight (ağırlık):** Her girdinin ne kadar önemli olduğunu belirler
- **Bias (önyargı):** Modelin esnekliğini artıran sabit değer
- **Activation Function (aktivasyon fonksiyonu):** Çıktıyı belirli bir aralığa sıkıştırır (ReLU, Sigmoid, Tanh)

### Çok Katmanlı Sinir Ağı

```mermaid
flowchart LR
    subgraph input ["Input Layer\n(Girdi Katmanı)"]
        I1["x₁"]
        I2["x₂"]
        I3["x₃"]
    end

    subgraph hidden1 ["Hidden Layer 1\n(Gizli Katman 1)"]
        H1["n₁"]
        H2["n₂"]
        H3["n₃"]
        H4["n₄"]
    end

    subgraph hidden2 ["Hidden Layer 2\n(Gizli Katman 2)"]
        H5["n₅"]
        H6["n₆"]
        H7["n₇"]
    end

    subgraph output ["Output Layer\n(Çıktı Katmanı)"]
        O1["y₁"]
        O2["y₂"]
    end

    I1 --> H1 & H2 & H3 & H4
    I2 --> H1 & H2 & H3 & H4
    I3 --> H1 & H2 & H3 & H4

    H1 --> H5 & H6 & H7
    H2 --> H5 & H6 & H7
    H3 --> H5 & H6 & H7
    H4 --> H5 & H6 & H7

    H5 --> O1 & O2
    H6 --> O1 & O2
    H7 --> O1 & O2
```

### Eğitim Süreci

```mermaid
flowchart TD
    A["1. Forward Pass\n(İleri Geçiş)"] --> B["2. Loss Hesaplama\n(Hata Ölçümü)"]
    B --> C["3. Backpropagation\n(Geri Yayılım)"]
    C --> D["4. Weight Güncelleme\n(Ağırlık Ayarı)"]
    D --> A

    A -.- A1["Girdiyi ağ boyunca ilerlet,\nçıktı üret"]
    B -.- B1["Üretilen çıktı ile beklenen\nçıktı arasındaki farkı hesapla"]
    C -.- C1["Hatanın her ağırlığa\nne kadar bağlı olduğunu hesapla"]
    D -.- D1["Hatayı azaltacak şekilde\nağırlıkları güncelle"]
```

Bu döngü, model yeterli doğruluğa ulaşana kadar milyonlarca kez tekrarlanır.

---

## Transformer Öncesi: RNN ve LSTM

Transformer'dan önce, metin işleme için RNN (Recurrent Neural Network) ve LSTM (Long Short-Term Memory) kullanılıyordu.

### RNN'in Sorunu

```mermaid
flowchart LR
    W1["Bugün"] --> N1["RNN"] --> W2["hava"] --> N2["RNN"] --> W3["çok"] --> N3["RNN"] --> W4["güzel"] --> N4["RNN"]
    
    N1 -->|"durum"| N2 -->|"durum"| N3 -->|"durum"| N4
```

**Sorunlar:**
- **Sıralı işlem:** Kelimeler birer birer işlenir → yavaş
- **Vanishing Gradient (kaybolan gradyan):** Uzun cümlelerde başlangıçtaki kelimelerin etkisi kaybolur
- **Paralelleştirilemez:** GPU'lardan tam verim alınamaz

**Örnek sorun:**
```
"İstanbul'da doğdum, 20 yıl yaşadım, sonra Ankara'ya taşındım,
 orada 5 yıl kaldım, şimdi tekrar ___ dönmek istiyorum."

RNN, "İstanbul" kelimesini bu kadar uzak mesafede hatırlayamayabilir.
```

---

## Transformer Mimarisi

2017'de Google araştırmacıları tarafından yayınlanan **"Attention Is All You Need"** makalesi, NLP alanını kökten değiştirdi.

### Transformer'ın Temel Bileşenleri

```mermaid
flowchart TD
    subgraph transformer ["Transformer Mimarisi"]
        subgraph encoder ["Encoder (Kodlayıcı)"]
            E1["Input Embedding\n+ Positional Encoding"]
            E2["Multi-Head\nSelf-Attention"]
            E3["Feed Forward\nNeural Network"]
            E1 --> E2 --> E3
        end
        
        subgraph decoder ["Decoder (Kod Çözücü)"]
            D1["Output Embedding\n+ Positional Encoding"]
            D2["Masked Multi-Head\nSelf-Attention"]
            D3["Cross-Attention\n(Encoder'a bakış)"]
            D4["Feed Forward\nNeural Network"]
            D1 --> D2 --> D3 --> D4
        end
        
        E3 -->|"bağlam"| D3
    end

    INPUT["Girdi Metni"] --> encoder
    decoder --> OUTPUT["Çıktı Metni"]
```

### Anahtar Kavramlar

#### 1. Embedding (Gömme)

Kelimeleri sayısal vektörlere dönüştürür. Anlamca yakın kelimeler, vektör uzayında birbirine yakın olur.

```
"kral"   → [0.2, 0.8, 0.1, ...]
"kraliçe" → [0.3, 0.7, 0.2, ...]  ← "kral"a yakın
"araba"  → [0.9, 0.1, 0.6, ...]  ← "kral"dan uzak

Ünlü formül: kral - erkek + kadın ≈ kraliçe
```

#### 2. Positional Encoding (Konum Kodlama)

Transformer kelimeleri aynı anda işlediği için, sıra bilgisini kaybeder. Positional Encoding, her kelimeye konum bilgisi ekler.

```
"Köpek kediye saldırdı"  vs  "Kedi köpeğe saldırdı"
→ Positional Encoding olmazsa ikisi aynı anlama gelir
```

#### 3. Self-Attention (Öz-Dikkat) - En Önemli Kavram

Self-Attention, bir cümledeki her kelimenin diğer tüm kelimelere ne kadar "dikkat etmesi" gerektiğini hesaplar.

```mermaid
flowchart TD
    subgraph attention ["Self-Attention Örneği"]
        W1["Hayvan"]
        W2["çok"]
        W3["yorgundu"]
        W4["çünkü"]
        W5["uzun"]
        W6["süre"]
        W7["koştu"]
    end

    W3 -->|"güçlü dikkat"| W1
    W3 -->|"güçlü dikkat"| W7
    W3 -->|"zayıf dikkat"| W2
    W3 -->|"orta dikkat"| W4
```

**"yorgundu" kelimesi hangi kelimelere dikkat eder?**
- "Hayvan" → güçlü (ne yorgun?)
- "koştu" → güçlü (neden yorgun?)
- "uzun süre" → orta (ne kadar koştu?)
- "çok" → zayıf (sadece pekiştirme)

#### 4. Multi-Head Attention (Çok Başlı Dikkat)

Tek bir Attention yerine, birden fazla Attention hesaplaması paralel olarak yapılır. Her "head" (baş), metnin farklı yönlerine odaklanır.

```mermaid
flowchart LR
    INPUT["Girdi"] --> H1["Head 1\n(Gramer ilişkileri)"]
    INPUT --> H2["Head 2\n(Anlamsal ilişkiler)"]
    INPUT --> H3["Head 3\n(Uzak bağımlılıklar)"]
    INPUT --> H4["Head 4\n(Yerel kalıplar)"]
    
    H1 --> CONCAT["Birleştir\n(Concat)"]
    H2 --> CONCAT
    H3 --> CONCAT
    H4 --> CONCAT
    
    CONCAT --> OUTPUT["Zengin\nTemsil"]
```

---

## GPT vs BERT: İki Farklı Kullanım

Transformer mimarisi iki farklı şekilde kullanılır:

```mermaid
flowchart TD
    TR["Transformer"]
    
    TR --> ENCODER["Sadece Encoder\n(BERT tarzı)"]
    TR --> DECODER["Sadece Decoder\n(GPT tarzı)"]
    TR --> BOTH["Encoder + Decoder\n(T5, BART tarzı)"]
    
    ENCODER --> E1["Metin anlama\nSınıflandırma\nSoru cevaplama"]
    DECODER --> D1["Metin üretme\nKod yazma\nSohbet"]
    BOTH --> B1["Çeviri\nÖzetleme"]
    
    D1 --> LLM["Modern LLM'ler\n(Claude, GPT, Gemini)"]
```

| Özellik | BERT (Encoder) | GPT (Decoder) |
|---------|----------------|---------------|
| **Yön** | Çift yönlü (Bidirectional) | Tek yönlü (Left-to-right) |
| **Görev** | Anlama | Üretme |
| **Eğitim** | Maskelenmiş kelime tahmini | Sonraki kelime tahmini |
| **Kullanım** | Arama, sınıflandırma | Chatbot, kod yazma |

> **Günümüz LLM'leri (Claude, GPT, Gemini)** Decoder-only Transformer mimarisini kullanır. Temel görevleri: verilen metnin devamını tahmin etmek.

---

## Next Token Prediction (Sonraki Token Tahmini)

Modern LLM'lerin çalışma prensibi aslında çok basittir: **bir sonraki token'ı tahmin et.**

```mermaid
sequenceDiagram
    participant U as Girdi
    participant M as Model
    participant O as Çıktı

    U->>M: "Bugün hava çok"
    M->>M: Tüm olası kelimelerin olasılıklarını hesapla
    Note over M: "güzel" → %35<br/>"sıcak" → %25<br/>"soğuk" → %20<br/>"güneşli" → %15<br/>"kötü" → %5
    M->>O: "güzel" (en yüksek olasılık)
    
    U->>M: "Bugün hava çok güzel"
    M->>M: Yeni bağlamla tekrar hesapla
    M->>O: "olduğu"
    
    U->>M: "Bugün hava çok güzel olduğu"
    M->>M: Tekrar hesapla
    M->>O: "için"
```

Bu süreç, model bir bitirme sinyali (end token) üretene kadar devam eder. Her adımda tüm önceki metin bağlam olarak kullanılır.

---

## Parameter (Parametre) ve Model Boyutu

Bir modelin "boyutu", sahip olduğu parametre sayısıyla ölçülür. Parametreler, eğitim sırasında ayarlanan ağırlık (weight) ve bias değerleridir.

| Model | Parametre Sayısı | Yıl |
|-------|-----------------|------|
| GPT-1 | 117 Milyon | 2018 |
| GPT-2 | 1.5 Milyar | 2019 |
| GPT-3 | 175 Milyar | 2020 |
| GPT-4 | ~1.8 Trilyon (tahmin) | 2023 |
| Claude 4.6 Opus | Açıklanmadı | 2026 |
| Llama 4 Maverick | 400 Milyar | 2025 |
| DeepSeek-V3.2 | 671 Milyar (37B aktif - MoE) | 2025 |

> **MoE (Mixture of Experts):** DeepSeek gibi bazı modeller, 671 milyar parametreye sahip olmasına rağmen her sorguda sadece 37 milyarını aktif kullanır. Bu, verimliliği artırır.

---

## Özet

```mermaid
flowchart TD
    NN["Neural Network\n(Temel yapı taşı)"] --> DNN["Deep Neural Network\n(Çok katmanlı)"]
    DNN --> RNN["RNN / LSTM\n(Sıralı işlem - eski)"]
    DNN --> TR["Transformer\n(Paralel işlem - 2017)"]
    
    TR --> ATT["Self-Attention\n(Her kelime diğerlerine bakar)"]
    TR --> POS["Positional Encoding\n(Sıra bilgisi)"]
    TR --> MHA["Multi-Head Attention\n(Farklı açılardan analiz)"]
    
    TR --> NTP["Next Token Prediction\n(LLM'lerin çalışma prensibi)"]
```

| Kavram | Açıklama |
|--------|----------|
| **Neural Network** | Nöronlardan oluşan hesaplama ağı |
| **Weight** | Bağlantıların gücünü belirleyen değerler |
| **Transformer** | 2017'de çıkan, Attention tabanlı mimari |
| **Self-Attention** | Her kelimenin diğer kelimelere dikkat etmesi |
| **Next Token Prediction** | LLM'lerin temel çalışma prensibi |
| **Parameter** | Modelin öğrendiği ayarlanabilir değerler |

---

## Sonraki Adım

Sinir ağları ve Transformer mimarisini anladık. Şimdi tüm bu kavramları bir araya getiren kapsamlı bir sözlüğe göz atalım:

→ [Temel Kavramlar Sözlüğü](./05-temel-kavramlar-sozlugu.md)
