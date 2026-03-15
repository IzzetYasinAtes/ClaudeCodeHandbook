# Sıkça Sorulan Sorular (SSS)

Claude Code hakkında en sık sorulan soruların yanıtları. Kurulum, kullanım, fiyatlandırma, güvenlik, takım kullanımı, entegrasyon ve ileri düzey özellikler başlıkları altında düzenlenmiştir.

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code temelleri | [Bölüm 06](../06-claude-code-tanitim/README.md) |

---

## Kurulum ve Başlangıç

### S1: Claude Code'u nasıl yüklerim?

**Y:** Node.js 18 veya üzeri yüklü olmalıdır. Ardından:

```bash
npm install -g @anthropic-ai/claude-code
```

Windows, macOS ve Linux'ta çalışır. Detaylı kurulum rehberi için [Kurulum ve Gereksinimler](../06-claude-code-tanitim/03-kurulum-ve-gereksinimler.md) bölümüne bakın.

---

### S2: Hangi Node.js versiyonu gerekli?

**Y:** Minimum Node.js 18 gereklidir. Node.js 20 LTS önerilir. Versiyon kontrolü:

```bash
node --version
```

---

### S3: API anahtarı nereden alınır?

**Y:** İki yol vardır:
1. **Anthropic Console:** [console.anthropic.com](https://console.anthropic.com) adresinden API key oluşturun
2. **Tarayıcı tabanlı giriş:** `claude login` komutuyla tarayıcı üzerinden giriş yapın (API key gerektirmez)

---

### S4: Claude Code'u güncellemek için ne yapmalıyım?

**Y:**

```bash
npm update -g @anthropic-ai/claude-code
```

Mevcut versiyonunuzu kontrol etmek için:

```bash
claude --version
```

---

## Kullanım

### S5: Claude Code ile Claude AI (claude.ai) arasındaki fark nedir?

**Y:**

| Özellik | Claude AI (Web) | Claude Code (Terminal) |
|---------|-----------------|----------------------|
| Arayüz | Web tarayıcı | Terminal / CLI |
| Dosya erişimi | Yüklenen dosyalar | Tüm proje dosyaları |
| Kod çalıştırma | Sınırlı | Tam Bash erişimi |
| Git entegrasyonu | Yok | Tam entegrasyon |
| Otomasyon | Yok | Hooks, CI/CD, SDK |
| Hedef kitle | Genel kullanıcı | Yazılım geliştirici |

Detaylar için [Claude Code vs Claude AI](../06-claude-code-tanitim/06-claude-code-vs-claude-ai.md) bölümüne bakın.

---

### S6: Claude Code hangi programlama dillerini destekler?

**Y:** Claude Code dil bağımsızdır. Herhangi bir programlama dilinde kod yazabilir, düzenleyebilir ve analiz edebilir. En güçlü olduğu diller: Python, JavaScript/TypeScript, Java, C#, Go, Rust, Ruby, PHP, Swift ve Kotlin.

---

### S7: Claude Code internet erişimi kullanabilir mi?

**Y:** Evet. Claude Code, `fetch` aracı ile web sayfalarını okuyabilir ve `Bash` aracı ile `curl` gibi HTTP istemcileri çalıştırabilir. Ayrıca MCP sunucuları üzerinden harici API'lere erişebilir.

---

### S8: Birden fazla dosyada aynı anda değişiklik yapabilir mi?

**Y:** Evet. Claude Code tek bir istek içinde birden fazla dosyayı okuyabilir, düzenleyebilir, oluşturabilir ve silebilir. Tüm değişiklikler tek bir oturumda yapılır.

---

### S9: /compact komutu ne işe yarar?

**Y:** `/compact`, konuşma geçmişini özetleyerek context window'u temizler. Uzun oturumlarda performansı korumak için kullanılır. Opsiyonel olarak bir özet mesajı verilebilir:

```bash
> /compact "Auth modülünü refactor ediyorum, login servisi tamamlandı"
```

---

### S10: Plan Mode nedir ve ne zaman kullanılmalı?

**Y:** Plan Mode, Claude Code'un dosya değişikliği yapmadan sadece plan oluşturmasını sağlayan moddur. Büyük refactoring, mimari kararlar veya karmaşık görevlerde önce plan yapıp sonra uygulamak için kullanılır.

```bash
# Plan Mode'a geç
> /plan

# veya Shift+Tab ile modlar arasında geçiş
```

---

## Fiyatlandırma ve Kullanım Limitleri

### S11: Claude Code ücretsiz mi?

**Y:** Claude Code CLI aracı ücretsizdir, ancak API kullanımı ücretlidir. Anthropic API'sine yapılan her istek, kullanılan token miktarına göre faturalandırılır. Detaylı fiyatlandırma için [Anthropic Pricing](https://www.anthropic.com/pricing) sayfasını ziyaret edin.

---

### S12: Ne kadar harcadığımı nasıl görebilirim?

**Y:** Birkaç yol vardır:
1. **Anthropic Console:** Dashboard'da kullanım istatistikleri
2. **Claude Code içinde:** Oturum sonunda gösterilen token kullanım özeti
3. **Konfigürasyon:** `settings.json` dosyasında maliyet sınırları ayarlanabilir

---

### S13: Token kullanımını nasıl azaltabilirim?

**Y:**
- `/compact` ile gereksiz context'i temizleyin
- Büyük dosyaları tamamen okutmak yerine ilgili bölümleri hedefleyin
- CLAUDE.md dosyasını kısa ve öz tutun
- Görevleri küçük parçalara bölün
- `--fast` flag'i ile daha hızlı ve ucuz model kullanın

---

## Güvenlik

### S14: Claude Code kodlarımı Anthropic sunucularına gönderiyor mu?

**Y:** Claude Code, API isteklerinde konuşma geçmişini ve okunan dosya içeriklerini Anthropic'in API'sine gönderir. Anthropic'in veri kullanım politikası gereği, API aracılığıyla gönderilen veriler model eğitiminde kullanılmaz. Detaylar için [Anthropic Privacy Policy](https://www.anthropic.com/privacy) sayfasını ziyaret edin.

---

### S15: Claude Code tehlikeli komutlar çalıştırabilir mi?

**Y:** Claude Code, komut çalıştırmadan önce kullanıcı onayı ister (varsayılan modda). İzin modları ile kontrol edebilirsiniz:
- **Suggest Mode:** Her araç kullanımı için onay ister
- **Auto-edit Mode:** Dosya düzenlemelerine otomatik izin, Bash komutları için onay ister
- **YOLO Mode:** Tüm araçlara otomatik izin (dikkatli kullanın)

Hooks ile tehlikeli komutları engelleyebilirsiniz. Detaylar için [İzin Sistemi](../10-izinler-ve-guvenlik/01-izin-sistemi.md) bölümüne bakın.

---

### S16: .env dosyalarım güvende mi?

**Y:** Claude Code, `.env` dosyalarını okuyabilir. Hassas bilgilerin korunması için:
- `.claude/settings.json` dosyasında `.env` dosyalarının okunmasını engelleyen kural ekleyin
- Hooks ile hassas dosya erişimini izleyin
- `.gitignore` dosyasında `.env` bulunduğundan emin olun

---

## Takım Kullanımı

### S17: Claude Code'u ekip genelinde nasıl kullanırız?

**Y:**
1. **CLAUDE.md** dosyasını repo'ya ekleyerek takım genelinde kurallar tanımlayın
2. **Proje bazlı settings.json** ile izin kurallarını standartlaştırın
3. **Hooks** ile kod standartlarını zorunlu kılın (örn: her commit'te lint)
4. **MCP sunucuları** ile takım araçlarına erişim sağlayın

---

### S18: CLAUDE.md dosyasını repo'ya commit etmeli miyim?

**Y:** Evet, önerilir. CLAUDE.md dosyası projenin kurallarını ve bağlamını tanımlar. Takım üyeleri Claude Code'u çalıştırdığında aynı kuralları otomatik olarak uygular. Hassas bilgi içermediğinden emin olun.

---

## Entegrasyon

### S19: Claude Code'u VS Code veya Cursor ile kullanabilir miyim?

**Y:** Evet. Claude Code terminal tabanlı çalıştığı için herhangi bir IDE'nin entegre terminali içinde çalıştırılabilir. Ayrıca VS Code uzantısı ve Cursor'da özel entegrasyon seçenekleri vardır. Detaylar için [IDE Entegrasyonları](../15-entegrasyonlar/README.md) bölümüne bakın.

---

### S20: Claude Code'u CI/CD pipeline'da kullanabilir miyim?

**Y:** Evet. Claude Code, headless mode (arayüzsüz mod) ile CI/CD ortamlarında çalıştırılabilir:

```bash
# GitHub Actions örneği
claude -p "Bu PR'daki değişiklikleri incele ve code review yap" --output-format json
```

Detaylar için [CI/CD ve DevOps](../16-cicd-ve-devops/README.md) bölümüne bakın.

---

### S21: MCP nedir ve neden kullanmalıyım?

**Y:** MCP (Model Context Protocol), Claude Code'un dış araçlara ve veri kaynaklarına erişmesini sağlayan açık bir protokoldür. Veritabanı sorgulama, Jira ticket oluşturma, Slack mesajı gönderme gibi görevler için MCP sunucuları kullanılır. Detaylar için [MCP](../11-mcp/README.md) bölümüne bakın.

---

## İleri Düzey Özellikler

### S22: Subagent nedir?

**Y:** Subagent (alt ajan), Claude Code'un karmaşık görevleri paralel olarak yürütmek için oluşturduğu bağımsız çalışan yardımcı ajanlardır. Ana ajan koordinasyon yaparken subagent'lar farklı dosyalar veya görevler üzerinde çalışır. Detaylar için [Subagent'lar](../13-subagentlar-ve-agent-takimlari/README.md) bölümüne bakın.

---

### S23: --worktree ne işe yarar?

**Y:** `--worktree` flag'i, aynı repo'da bağımsız çalışma dizinleri oluşturarak paralel görevlerin birbirini etkilemeden çalışmasını sağlar. Farklı branch'lerde farklı görevleri aynı anda yürütebilirsiniz:

```bash
claude --worktree feature/auth "Auth modülünü yeniden yaz"
claude --worktree feature/api "API endpoint'lerini güncelle"
```

---

### S24: Hooks ile neler yapabilirim?

**Y:** Hooks, Claude Code'un yaşam döngüsündeki belirli anlarda otomatik eylemler tetiklemenizi sağlar:
- Dosya düzenlemelerinden sonra otomatik formatlama (prettier)
- Tehlikeli komutları engelleme
- Her oturum sonunda Slack'e bildirim gönderme
- Kod değişikliklerinden sonra otomatik test çalıştırma
- Commit mesajı formatını zorunlu kılma

Detaylar için [Hooks ve Otomasyon](../14-hooks-ve-otomasyon/README.md) bölümüne bakın.

---

### S25: Claude Code'un desteklediği maksimum dosya boyutu nedir?

**Y:** Claude Code, teknik olarak herhangi bir boyuttaki dosyayı okuyabilir. Ancak büyük dosyalar context window'u hızlıca doldurur. Pratik öneriler:
- **1.000 satıra kadar:** Sorunsuz okunabilir
- **1.000-5.000 satır:** İlgili bölümleri hedefleyin
- **5.000+ satır:** Parça parça okuyun veya özet isteyin

---

### S26: Claude Code offline çalışabilir mi?

**Y:** Hayır. Claude Code, Anthropic API'sine bağlantı gerektirir. İnternet bağlantısı olmadan çalışmaz. Ancak yerel dosya sistemi erişimi doğrudan yapılır, API'ye yalnızca model yanıtları için ihtiyaç duyulur.

---

### S27: Claude Code'da birden fazla model kullanabilir miyim?

**Y:** Evet. Claude Code varsayılan olarak en yetenekli modeli kullanır, ancak `--model` flag'i ile farklı modeller seçilebilir:

```bash
claude --model claude-sonnet-4-20250514 "Bu fonksiyonu optimize et"
```

Ayrıca `--fast` flag'i ile daha hızlı ve ekonomik model tercih edilebilir.

---

## Sorun Giderme Genel

### S28: Claude Code yanıt vermiyorsa ne yapmalıyım?

**Y:**
1. İnternet bağlantınızı kontrol edin
2. API key'in geçerliliğini doğrulayın
3. `Ctrl+C` ile mevcut oturumu sonlandırıp yeniden başlatın
4. `claude --debug` ile detaylı log alın
5. Sorun devam ederse `npm update -g @anthropic-ai/claude-code` ile güncelleyin

---

### S29: Claude Code yanlış dosyayı düzenliyorsa ne yapmalıyım?

**Y:** Bu genellikle context window kirlenmesinin belirtisidir:
1. `/compact` ile context'i temizleyin
2. Daha spesifik talimat verin: dosya yolunu açıkça belirtin
3. CLAUDE.md'de dosya yapısını tanımlayın
4. Gerekirse yeni oturum başlatın

---

### S30: Bir değişikliği geri alabilir miyim?

**Y:** Evet. Claude Code, checkpointing desteği sunar:

```bash
# Önceki duruma geri dön
> /undo

# Git ile geri al
git checkout -- <dosya>
git stash
```

Ayrıca Claude Code her oturumda otomatik checkpoint oluşturabilir. Detaylar için [Checkpointing](../09-bellek-ve-baglam/07-checkpointing.md) bölümüne bakın.

---

## Bu SSS'de Yanıtınızı Bulamadıysanız

1. [Çok Karşılaşılan Sorunlar](./01-cok-sorulan-sorunlar.md) bölümünü inceleyin
2. [Context Window Sorunları](./02-context-window-sorunlari.md) bölümünü inceleyin
3. [Anthropic Documentation](https://docs.anthropic.com) resmi dokümantasyonunu kontrol edin
4. [GitHub Issues](https://github.com/anthropics/claude-code/issues) üzerinden arama yapın
5. Topluluk forumlarında sorunuzu paylaşın
