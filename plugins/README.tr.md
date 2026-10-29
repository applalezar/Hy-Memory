# Hy Memory Eklentileri

Ajan / düzenleyici platformları için tüm entegrasyon kodları. Bellek yetenekleri, kök dizindeki Python SDK (`hy_memory/`, PyPI: `hy-memory`) tarafından sağlanır; eklenti katmanı ise ana bilgisayar çerçevelerine bağlanır (HTTP, hook'lar, MCP vb.).

## Yerleşim

| Dizin | Açıklama | Yayın / Doküman |
|------|---------|------------------|
| `hermes/` | Hermes yerel memory provider (standart kütüphane HTTP istemcisi, ayrı servis) | [Kurulum notları](hermes/README.tr.md) |
| `openclaw/` | OpenClaw hafıza eklentisi (TypeScript, Python `hy_memory.server`'ı başlatır) | [OpenClaw rehberi](openclaw/README.tr.md) |
| `hook/` | Claude Code benzeri hook entegrasyonları (rezerv) | — |
| `mcp/` | MCP sunucu entegrasyonu (rezerv) | — |

`Mem0` ile fark: Mem0'un OpenClaw eklentisi **TypeScript SDK** (`mem0ai`) bağımlıdır; bu depodaki eklentiler **Python SDK**'ye dayanır ve yerel HTTP hizmeti üzerinden iletişim kurar.

## Geliştirme

```bash
# OpenClaw eklentisi
cd plugins/openclaw
npm install
npm run build
```

npm'ye yayınlama:

```bash
bash scripts/publish_npmjs.sh <version>
```
