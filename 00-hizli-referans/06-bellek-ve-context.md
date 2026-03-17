# Bellek ve Context Window

Claude Code'un bellek mekanizmaları ve context window yönetiminin özet referansıdır. Detaylı anlatım için [Bölüm 09 - Otomatik Bellek](../09-bellek-ve-baglam/04-otomatik-bellek.md) ve [Context Window Yönetimi](../09-bellek-ve-baglam/05-context-window-yonetimi.md) dosyalarına bakın.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| CLAUDE.md ve Kurallar | [CLAUDE.md ve Kurallar](./05-claude-md-ve-kurallar.md) |
| Claude Code temel kullanım | [Bölüm 06](../06-claude-code-tanitim/README.md) |

---

## 1. Otomatik Bellek

Claude Code, çalışma sırasında yaptığınız düzeltmeleri ve verdiğiniz geri bildirimleri fark eder. Bu öğrenmeleri `~/.claude/user_memory.md` dosyasına kaydeder ve sonraki oturumlarda bu bilgileri kullanır.

### Bellek Türleri

| Tür | Dosya Konumu | Kapsam |
|-----|-------------|--------|
| **User Memory** | `~/.claude/user_memory.md` | Tüm projeler için geçerli kişisel tercihler |
| **Feedback Memory** | Oturum içinde otomatik | Düzeltme ve geri bildirim kalıpları |
| **Project Memory** | `~/.claude/projects/<proje>/memory/` | Projeye özel öğrenilmiş bilgiler |
| **Reference Memory** | CLAUDE.md + .claude/rules/ | Siz tarafından yazılan kurallar |

### Bellek Dosyaları

```
~/.claude/
├── user_memory.md              # Kişisel tercihler (tüm projeler)
├── CLAUDE.md                   # Kullanıcı bazlı kurallar
└── projects/
    └── <proje-hash>/
        └── memory/             # Projeye özel öğrenilmiş bilgiler
```

### Bellek Yönetimi İpuçları

- Otomatik bellek **ilk 200 satır** ile sınırlıdır; üzerini Claude okumaz.
- Ayda bir `~/.claude/user_memory.md` dosyasını gözden geçirin.
- Tekrarlı girdileri birleştirin, eski girdileri silin.
- Projeye özel kuralları CLAUDE.md'ye taşıyın.

---

## 2. Context Window

Context window, Claude'un bir oturumda aynı anda görebildiği bilgi miktarıdır.

| Model | Kapasite |
|-------|----------|
| Sonnet | 200.000 token (~150.000 kelime) |
| Opus | 1.000.000 token (~750.000 kelime) |

### %80 Kuralı

Context window %80 doluluğa ulaştığında yeni oturum başlatmanız önerilir. Performans bu eşikten sonra belirgin şekilde düşer.

### Doluluk Kontrolü

```bash
# Context window doluluğunu görüntüle
> /context

# Örnek çıktı:
# Context usage: 67,340 / 200,000 tokens (33.7%)
# ████████████████░░░░░░░░░░░░░░░░░░
```

### /compact ile Sıkıştırma

```bash
# Temel sıkıştırma
> /compact

# Belirli bir odak noktasıyla sıkıştırma
> /compact auth modülündeki değişikliklere odaklan
```

`/compact` konuşma geçmişini özetleyerek context window'da yer açar. CLAUDE.md kuralları, önemli kararlar ve aktif görev bağlamı korunur.

---

## 3. Token Tasarrufu İpuçları

| Yaklaşım | Kötü Örnek | İyi Örnek |
|-----------|-----------|-----------|
| Spesifik dosya iste | "Projedeki tüm dosyaları oku" | "src/auth/middleware.ts dosyasını oku" |
| Odaklı görev ver | "Bu projeyi iyileştir" | "Rate limiting fonksiyonuna Redis desteği ekle" |
| Çıktıyı sınırla | "Tüm test sonuçlarını göster" | "Testleri çalıştır, sadece başarısız olanları göster" |
| Görev parçala | Tek oturumda 50 dosya | Her oturumda 5-10 dosya |

---

## 4. Performans Degradasyon Eğrisi

Context window doldukça Claude Code'un performansı aşağıdaki şekilde düşer:

```
Performans (%)
100 |====
 95 |============
 90 |=================
 85 |=====================
 80 |========================
 70 |===========================
 60 |==============================
 50 |================================
 35 |==================================
 20 |====================================
    +----+----+----+----+----+----+----+----+----+----+
    0%   10%  20%  30%  40%  50%  60%  70%  80%  90% 100%
                       Doluluk (%)
```

| Doluluk Aralığı | Performans | Eylem |
|------------------|------------|-------|
| **0-25%** | Optimal | Normal çalışma |
| **25-50%** | İyi | Normal çalışma |
| **50-73%** | Azalan | `/compact` kullanmayı düşünün |
| **73-80%** | Düşük | `/compact` kullanın |
| **80%+** | Kritik | Yeni oturum başlatın |

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| **Otomatik Bellek** | Düzeltmelerden öğrenilenler `~/.claude/user_memory.md`'ye kaydedilir |
| **200 satır limiti** | Otomatik bellek dosyasının yalnızca ilk 200 satırı okunur |
| **Context Window** | 200K (Sonnet) / 1M (Opus) token kapasiteli bilgi penceresi |
| **%80 Kuralı** | %80 dolulukta yeni oturum başlatın |
| **/context** | Token kullanımını görselleştirir |
| **/compact** | Konuşma geçmişini özetleyerek yer açar |

---

## Sonraki Adım

Oturum yönetimi mekanizmalarını öğrenin:

> [Oturum Yönetimi](./07-oturum-yonetimi.md)
