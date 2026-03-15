# Bölüm 09: Bellek ve Bağlam Yönetimi

Claude Code'un gücü, bağlamı ne kadar iyi yönettiğinize bağlıdır. Bu bölüm, CLAUDE.md dosyasından context window stratejilerine, oturum yönetiminden paralel çalışmaya kadar tüm bellek ve bağlam mekanizmalarını kapsar.

## Bu Bölümde Neler Öğreneceksiniz?

```mermaid
flowchart TD
    A["CLAUDE.md Dosyası\n(Birincil Bellek)"] --> B["AGENTS.md ve\nDiğer Config"]
    B --> C["Kurallar Dizini\n(.claude/rules/)"]
    C --> D["Otomatik Bellek\n(Auto Memory)"]
    D --> E["Context Window\nYönetimi"]
    E --> F["Oturum Yönetimi"]
    F --> G["Checkpointing"]
    G --> H["Worktree ile\nParalel Çalışma"]

    style A fill:#4A90D9,color:#fff
    style E fill:#E74C3C,color:#fff
    style H fill:#27AE60,color:#fff
```

## İçerik

| # | Dosya | Konu | Süre |
|---|-------|------|------|
| 01 | [CLAUDE.md Dosyası](./01-claude-md-dosyasi.md) | Birincil bellek dosyası, yerleşim, kapsam hiyerarşisi | ~12 dk |
| 02 | [AGENTS.md ve Diğer Config](./02-agents-md-ve-diger-config.md) | Çapraz araç uyumluluğu, .cursorrules, copilot-instructions | ~10 dk |
| 03 | [Kurallar Dizini](./03-kurallar-dizini.md) | .claude/rules/*.md ile kapsam bazlı kural dosyaları | ~10 dk |
| 04 | [Otomatik Bellek](./04-otomatik-bellek.md) | Claude'un kendi öğrendiklerini kaydetmesi | ~8 dk |
| 05 | [Context Window Yönetimi](./05-context-window-yonetimi.md) | Token yönetimi, %80 kuralı, /compact, /context | ~15 dk |
| 06 | [Oturum Yönetimi](./06-oturum-yonetimi.md) | --continue, --resume, oturum geçmişi | ~10 dk |
| 07 | [Checkpointing](./07-checkpointing.md) | Değişiklikleri izleme, geri alma, durum yönetimi | ~8 dk |
| 08 | [Worktree ile Paralel Çalışma](./08-worktree-ile-paralel-calisma.md) | Git worktree, paralel oturumlar, izolasyon | ~12 dk |

## Neden Bu Kadar Önemli?

```mermaid
flowchart LR
    subgraph kotu ["❌ Kötü Bağlam Yönetimi"]
        direction TB
        K1["CLAUDE.md yok"] --> K2["Context window doluyor"]
        K2 --> K3["Her seferinde\naynı şeyleri tekrarlıyor"]
        K3 --> K4["Verim düşük"]
    end

    subgraph iyi ["✅ İyi Bağlam Yönetimi"]
        direction TB
        I1["CLAUDE.md hazır"] --> I2["Context window izleniyor"]
        I2 --> I3["Oturumlar yönetiliyor"]
        I3 --> I4["Maksimum verim"]
    end

    kotu -.->|"Bu bölümü okuyunca"| iyi
```

## Ön Koşullar

| Konu | Bölüm |
|------|-------|
| Claude Code nedir ve nasıl çalışır | [Bölüm 06](../06-claude-code-tanitim/README.md) |
| Araçlar ve komutlar | [Bölüm 08](../08-araclar/README.md) |
| Terminal / komut satırı kullanımı | Harici kaynak |
| Git temel bilgisi (worktree bölümü için) | Harici kaynak |

## Sonraki Adım

Bu bölümü tamamladıktan sonra → [10 - İzinler ve Güvenlik](../10-izinler-ve-guvenlik/README.md)
