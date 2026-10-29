# Monorepo Düzeni

Bu depo şu an bağımsız olarak derlenebilen iki projeden oluşur:

```text
hy-memory/
├── .github/           # Monorepo CI, native lifecycle ve Release iş akışları
├── hy_memory/          # Hy Memory SDK uygulaması
├── tests/              # SDK testleri
├── plugins/            # SDK entegrasyonları
├── pyproject.toml      # SDK paket tanımı
├── docs/               # Paylaşılan ve SDK belgeleri
├── memhub/             # Son kullanıcı uygulama çalışma alanı
    ├── apps/core/      # Python Core / Runtime veri düzlemi
    ├── apps/cli/       # CLI
    ├── apps/desktop/   # macOS Electron uygulaması
    ├── apps/web/       # Web arayüzü
    ├── packages/       # Sözleşmeler, host bridge, operasyonlar, runtime kontrolü
    └── scripts/        # Derleme, paketleme, yayın ve doğrulama araçları
```

## Depo politikası

SDK, Python paket düzeni ve yayın akışı sabit kalsın diye depo kökünde kalır. MemHub, `memhub/` altında tamamlanmış bir alt projedir. Bu açık monorepo, her iki bileşenin de kaynak doğrusu olarak kullanılır.

Yayın iş akışları incelenmiş kaynak veya ikili üretim dosyaları üretebilir; ancak bu çıktılar bu depoyu değiştirmez. Dahili koordinasyon notları, kimlik bilgileri, özel kabul kanıtları, üretim raporları ve derleme çıktıları açık kaynak ağaçlarında yer almamalıdır.

## Mevcut sınırlar

- SDK komutları depo kökünden çalıştırılır.
- MemHub komutları `memhub/` dizininden çalıştırılır.
- Kök GitHub iş akışları SDK ve MemHub kontrollerini ilgili çalışma dizinlerinden yürütür. Native lifecycle iş akışları incelenebilir Actions çıktıları üretir; etiketleme sürümleri kamu kimliği, imza ve notarization gereksinimlerinde fail-closed davranır.
- Hy Memory, `hy-memory==2.0.4` olarak yayınlanır. MemHub bu genel dağıtımı referans alır; seçili npm paketi `memhub-cli` 1.0.5'tir.
- Hy-Memory kaynağı depo kökünde Apache-2.0 lisanslıdır. Üçüncü taraf bileşenler kendi lisanslarını ve attribution bildirimlerini korur.

