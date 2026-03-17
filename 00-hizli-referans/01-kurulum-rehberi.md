# Kurulum Rehberi

Claude Code'u kurmak ve ilk oturumunuzu başlatmak için gereken adımların özet referansıdır. Detaylı anlatım için [Bölüm 06 - Kurulum ve Gereksinimler](../06-claude-code-tanitim/03-kurulum-ve-gereksinimler.md) dosyasına bakın.

## Ön Koşullar

| Gereksinim | Minimum |
|------------|---------|
| İşletim sistemi | Windows 11, macOS 12+, Ubuntu 20.04+ |
| Node.js | v18 veya üzeri |
| Claude planı | Pro veya Max abonelik |
| Terminal | Herhangi bir terminal uygulaması |

## Adım 1: Kurulum

```bash
npm install -g @anthropic-ai/claude-code
```

Kurulumu doğrulamak için:

```bash
claude --version
```

## Adım 2: İlk Başlatma

Proje klasörünüze gidin ve Claude Code'u başlatın:

```bash
cd proje-klasoru
claude
```

## Adım 3: Tema Seçimi

İlk çalıştırmada tema tercihiniz sorulur. Daha sonra değiştirmek için:

```
/theme
```

Dark ve Light mod arasından seçim yapabilirsiniz.

## Adım 4: Giriş Yöntemi

Üç giriş seçeneği sunulur:

| Seçenek | Açıklama |
|---------|----------|
| 1 - Claude subscription | Pro veya Max abonelik ile giriş |
| 2 - API Console | Anthropic API anahtarı ile giriş |
| 3 - 3rd party | Üçüncü parti API sağlayıcı |

Claude subscription seçildiğinde e-posta adresinizi girin ve tarayıcıda açılan sayfadan onay verin.

## Adım 5: CLAUDE.md Oluşturma

Projenize özel bellek dosyasını oluşturmak için:

```
/init
```

Bu komut, proje kök dizininde bir `CLAUDE.md` dosyası oluşturur. Detay için [Bölüm 09 - CLAUDE.md Dosyası](../09-bellek-ve-baglam/01-claude-md-dosyasi.md) bölümüne bakın.

## Adım 6: Kullanım Takibi

Oturum sırasında token kullanımınızı ve maliyetinizi görmek için:

```
/cost
```

## Temel Komutlar

| Komut | İşlev |
|-------|-------|
| `claude` | Claude Code'u başlatır |
| `/help` | Yardım menüsünü gösterir |
| `/init` | CLAUDE.md dosyasını oluşturur |
| `/plan` | Plan modunu açar/kapatır |
| `/cost` | Token kullanımını ve maliyeti gösterir |
| `/clear` | Oturum geçmişini temizler |
| `/logout` | Oturumu kapatır |
| `/theme` | Tema değiştirir |
| `/model` | Kullanılan modeli değiştirir |

## Sık Karşılaşılan Sorunlar

| Sorun | Çözüm |
|-------|-------|
| Authentication failed | `/logout` yapın, tekrar giriş yapın |
| API key hatası | Anahtar geçerliliğini Anthropic Console'dan kontrol edin |
| Limit exceeded | Plan limitinizi kontrol edin, gerekirse Max plana geçin |
| Node.js sürüm hatası | `node --version` ile v18+ olduğundan emin olun |

## Hızlı İpuçları

- **CLAUDE.md dosyasını iyi yazın:** Proje kuralları, mimari kararlar ve kodlama standartlarınızı ekleyin.
- **Plan Mode kullanın:** Karmaşık görevlerde önce `/plan` ile plan oluşturun, sonra uygulayın.
- **Git ile çalışın:** Claude Code, Git deposu içinde en verimli şekilde çalışır.
- **Spesifik olun:** Talimatlarınızı ne kadar net yazarsanız, sonuçlar o kadar iyi olur.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude ekosistemi hakkında genel bilgi | [Bölüm 05](../05-claude-ekosistemi/README.md) |
| Terminal temel kullanımı | Harici kaynak |

## Sonraki Adım

Bu dosyayı tamamladıktan sonra devam edin: [02 - MCP, Plugin ve Skill Karşılaştırması](./02-mcp-plugin-skill-karsilastirmasi.md)
