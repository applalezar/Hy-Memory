<p align="center">
  <img src="docs/images/hunyuanlogo.png" alt="Tencent Hunyuan" width="260">
</p>

<h1 align="center">Hy Memory</h1>

<p align="center">Üretim seviyesinde, LLM ajanları için ikili sistem bilişsel hafıza.</p>

<p align="center">
  <a href="#son-haberler"><img src="https://img.shields.io/badge/news-latest-e65100?style=flat-square" alt="Latest News"></a>
  <a href="#makaleler"><img src="https://img.shields.io/badge/papers-research-68448b?style=flat-square" alt="Papers"></a>
  <a href="#hizli-baslangic"><img src="https://img.shields.io/badge/get_started-quickstart-149954?style=flat-square" alt="Get Started"></a>
  <a href="#sistem-mimarisi"><img src="https://img.shields.io/badge/architecture-overview-007ec6?style=flat-square" alt="System Architecture"></a>
  <a href="#destek"><img src="https://img.shields.io/badge/support-docs-00897b?style=flat-square" alt="Support"></a>
  <a href="https://huggingface.co/tencent/Hy-Memory-30B-A3B"><img src="https://img.shields.io/badge/model-Hugging%20Face-ffd21e?style=flat-square" alt="Hugging Face Model"></a>
  <a href="memhub/README.md"><img src="https://img.shields.io/badge/MemHub-README-f28c45?style=flat-square" alt="MemHub"></a>
  <a href="https://memory.hunyuan.tencent.com/"><img src="https://img.shields.io/badge/website-Hy%20Memory-0052d9?style=flat-square" alt="Website"></a>
  <a href="https://discord.gg/x56wEb2PX"><img src="https://img.shields.io/badge/Discord-memory%20channel-5865f2?style=flat-square&logo=discord&logoColor=white" alt="Discord"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Apache--2.0-4c6ef5?style=flat-square" alt="Apache-2.0 License"></a>
</p>

[English](readme_en.md) | Türkçe | [中文](README.zh.md)

**[Hy Memory](https://memory.hunyuan.tencent.com/)** ([Resmî Web Sitesi](https://memory.hunyuan.tencent.com/)) üretim seviyesinde, LLM ajanları için ikili sistem bilişsel hafıza motoru ve Python SDK'sidir (`hy-memory` paketi). Bu sistem, kişisel ve evrimsel bellek akışlarını, kodlama ve üretkenlik iş akışlarını bir arada yönetir.

<div align="center">
  <video src="https://github.com/user-attachments/assets/94405890-8429-47c2-9084-38f7ed9f126f" controls width="640"></video>
  <p><b>MemHub Demo Videosu</b></p>
</div>

---

## Son Haberler

* *<mark>[yeni]</mark>* 🔥🔥🔥 **[2026-10]**: 🚀 **Açık kaynak olarak yayımlandı** kuantize **FP8** (**[Hy-Memory-30B-A3B-FP8](https://huggingface.co/tencent/Hy-Memory-30B-A3B-FP8)**) ve **GGUF** (**[Hy-Memory-30B-A3B-GGUF](https://huggingface.co/tencent/Hy-Memory-30B-A3B-GGUF)**)
* *<mark>[yeni]</mark>* 🔥🔥🔥 **[2026-10]**: 📦 **[MemHub v1.0.5](https://github.com/Tencent-Hunyuan/Hy-Memory/releases/tag/v1.0.5)** sürümü yayınlandı; Çin, İngilizce ve İspanyolca arayüzler dahil.
* *<mark>[yeni]</mark>* 🔥🔥🔥 **[2026-10]**: 📦 **[MemHub v1.0.4](https://github.com/Tencent-Hunyuan/Hy-Memory/releases/tag/v1.0.4)** ve **[memhub-cli 1.0.5](https://www.npmjs.com/package/memhub-cli)** yayınlandı.
* *<mark>[yeni]</mark>* 🔥🔥🔥 **[2026-10]**: 🚀 **Hy Memory 2.0** (`hy-memory==2.0.4`) **[PyPI](https://pypi.org/project/hy-memory/2.0.4/)** üzerinde yayınlandı ve **Hy-Memory** açık kaynak olarak paylaşıldı.
* *<mark>[yeni]</mark>* 🔥🔥🔥 **[2026-10]**: 🚀 **[Hy-Memory-30B-A3B](https://huggingface.co/tencent/Hy-Memory-30B-A3B)** Hugging Face üzerinde açık kaynak olarak yayınlandı; teknik rapor da eşlik ediyor.
* *<mark>[yeni]</mark>* 🔥🔥 **[2026-09]**: 📊 **[CMB: A Coding Memory Benchmark for Long-Horizon and Interleaved Coding Agents](docs/papers/CMB_A_Coding_Memory_Benchmark_for_Long_Horizon_and_Interleaved_Coding_Agents.pdf)** yayınlandı.
* *<mark>[yeni]</mark>* 🔥🔥 **[2026-08]**: 🎉 **EMNLP 2026**'da kabul edildi: **[Memory Beyond Recall: A Dual-Process Cognitive Memory System for Self-Evolving LLM Agents](https://arxiv.org/abs/2606.09483)**.
* 🔥 **[2026-05]**: 🌟 **İlk sürüm**: ilk jenerasyon dual-process bilişsel hafıza çerçevesi **[Hy Memory 1.0](https://mp.weixin.qq.com/s/dH5PGpxqArFZkSq3-QxWYA)** yayınlandı.

---

## Makaleler

**Hy Memory Araştırma Ailesi**'ne hoş geldiniz; bu aile, bilişsel sohbet hafızası, uzun süreli kodlama hafızası ve RL ile eğitilmiş hafıza yazma modellerini kapsar:

> [**Memory Beyond Recall: A Dual-Process Cognitive Memory System for Self-Evolving LLM Agents**](https://arxiv.org/abs/2606.09483) *(EMNLP 2026'da kabul edildi)*  
> **TL;DR:** **Chat** için ikili sistem (System 1 + System 2) bilişsel hafıza mimarisini önerir; kişisel hafızayı 8 katmanlı bir hiyerarşi halinde düzenler (ham diyaloglar, atomik gerçekler, özetler, kimlik, bilgi yapıları vb.).  
> 📄 Makale: https://arxiv.org/abs/2606.09483

> [**CMB: A Coding Memory Benchmark for Long-Horizon and Interleaved Coding Agents**](docs/papers/CMB_A_Coding_Memory_Benchmark_for_Long_Horizon_and_Interleaved_Coding_Agents.pdf) *(ICLR 2027 için gönderildi)*  
> **TL;DR:** Uzun süreli **Coding** ajanları için değerlendirme kıyaslaması ve iki katmanlı + sürüm evrimi diff-chain hafıza yöntemi tanıtır; yeniden kullanılabilir mühendislik deneyimlerini ve tam kod değişim bağlamını birlikte korur.  
> 📄 Makale: [docs/papers/CMB_A_Coding_Memory_Benchmark_for_Long_Horizon_and_Interleaved_Coding_Agents.pdf](docs/papers/CMB_A_Coding_Memory_Benchmark_for_Long_Horizon_and_Interleaved_Coding_Agents.pdf)

> [**Hy-Memory-30B-A3B: Attributed Reinforcement Learning for Long-Term Memory Writing**](https://arxiv.org/abs/2610.22196) *(Açık Model Teknik Raporu)*  
> **TL;DR:** Uzun vadeli hafıza çıkarımı ve çakışma çözümlemesi için öznitelikli pekiştirmeli öğrenme (attributed RL) ile 30B-A3B MoE modeli eğitir.  
> 📄 Makale: https://arxiv.org/abs/2610.22196 · Model: https://huggingface.co/tencent/Hy-Memory-30B-A3B

---

## Hızlı Başlangıç

### 1. Kurulum

```bash
pip install hy-memory==2.0.4              # çekirdek (ChromaDB ve Kuzu dahil)
pip install "hy-memory[qdrant]==2.0.4"    # + Qdrant
pip install "hy-memory[faiss]==2.0.4"     # + FAISS
pip install "hy-memory[graph]==2.0.4"     # + Neo4j
pip install "hy-memory[mysql]==2.0.4"     # + MySQL önbellek
pip install "hy-memory[rerank]==2.0.4"    # + yerel çok dilli cross-encoder rerank
pip install "hy-memory[all]==2.0.4"       # her şey
```

### 2. Yapılandırma

Yalnızca bir LLM ve bir embedding uç noktası yeterlidir; OpenAI uyumlu herhangi bir servis çalışır:

```bash
export MEMORY_LLM_API_KEY="sk-your-key"
export MEMORY_LLM_BASE_URL="https://api.deepseek.com"
export MEMORY_LLM_MODEL="deepseek-chat"

export MEMORY_EMBEDDER_API_KEY="sk-your-key"
export MEMORY_EMBEDDER_BASE_URL="https://dashscope.aliyuncs.com/compatible-mode/v1"
export MEMORY_EMBEDDER_MODEL="text-embedding-v3"
export MEMORY_EMBEDDING_DIMS=1024
```

Veya tek bir anahtar ile OpenAI varsayılanlarını kullanabilirsiniz:

```bash
export OPENAI_API_KEY="sk-your-key"
```

Tam liste için [Çevre Değişkeni Referansı](docs/env_reference.md) bölümüne bakın.

### 3. Sohbet Hafızası

```python
from hy_memory import HyMemoryClient

client = HyMemoryClient(mode="pro")

# Yazma: düz metin veya OpenAI formatı mesajlar
client.add("Astronomi ve özellikle Interstellar filmlerini seviyorum", user_id="user_1")
client.add([
    {"role": "user", "content": "Bir film öner"},
    {"role": "assistant", "content": "Interstellar'ı dene — Nolan'ın bilim kurgu şaheseri"},
], user_id="user_1")

# Arama: profile / normal / proactive kanallarını döndürür
results = client.search("Kullanıcının hangi filmleri sevdiğini sor", user_ids=["user_1"], limit=10)
for mem in results["memories"]["normal"]:
    print(f"  [{mem['score']:.2f}] {mem['content']}")

client.close()
```

### 4. Çalışma ve Kodlama Hafızası

Yazım işlemleri `tool_calls` veya `role="tool"` mesajlarını içermelidir ve yalnızca `pro` / `ultra` modunda etkin olur (`lite` modunda `scene` yok sayılır ve her şey ham hafıza olarak saklanır):

```python
messages = [
    {"role": "user", "content": "Arama istemcisindeki zaman aşımını düzelt."},
    {"role": "assistant", "content": "", "tool_calls": [{
        "id": "call_1", "type": "function",
        "function": {"name": "apply_patch", "arguments": '{"path":"src/search.py"}'},
    }]},
    {"role": "tool", "tool_call_id": "call_1", "name": "apply_patch",
     "content": "Patch applied successfully"},
]

client.add(
    messages,
    scene="coding",                              # veya "working"
    user_id="user_123",
    workspace_id="github.com/example/service",   # isteğe bağlı: depo / proje bazlı izole eder
    branch="fix/search-timeout",                 # isteğe bağlı: workspace_id gerekli
)

result = client.search(
    "Sonunda hangi zaman aşımı uygulaması korunmuş?",
    scene="coding",
    user_ids=["user_123"],
    workspace_id="github.com/example/service",
    branch="fix/search-timeout",
    limit=5,
)

result["facts"]          # olgu katmanı
result["experiences"]    # deneyim katmanı
result["chain_block"]    # yalnız coding: sürüm evrimi bağlamı (MEMORY_CODING_CHAIN_BLOCK_ENABLED olana kadar boş)
```

### 5. Yönlendirme, Kaynak İzolasyonu ve Gelişmiş API'ler

- **Otomatik sahne yönlendirmesi**: `scene` verilmezse, araç çağrısı olmayan yazılar doğrudan `chat`'e gider (LLM çağrısı yok), araç çağrısı içeren yazılar tek bir LLM çağrısı ile `chat` / `working` / `coding` arasında sınıflandırılır.
- **Kaynaklara göre ayrı hafızalar**: `add()` içinde `source="openclaw" | "cursor" | "codex"` parametresini geçirin; `search()` içinde `include_sources` / `exclude_sources` ile filtreleyin.
- **CRUD ve System 2 evrimi**: `client.get(id)`, `client.update(id, text)`, `client.delete(id)`, `client.list_memories(user_id=...)` ve `client.evolve(user_id=...)` (ultra modunda System 2'yi tetikler).
- **İsteğe özel embedding uç noktası override**: `add()` / `search()` içinde `embed_override={"base_url": ..., "api_key": ...}` kullanın veya `with embed_endpoint_override(...):` konteyneriyle eşzamanlı güvenli çağrı yapın.

Sağlayıcı yapılandırması, depolama arka uçları, sahneler ve tam API için [Kullanım Kılavuzu](docs/usage.md) bölümüne bakın.

---

## Sistem Mimarisi

<div align="center">
  <img src="docs/images/architecture.svg" alt="Hy Memory architecture" width="95%">
</div>

Hy Memory, her görev tipine göre üç hafıza sahnesi (`scene`) sunar. `add()` / `search()` önce sahne yönlendiriciden geçer; ardından Chat veya Working / Coding akışına girer. (Turuncu kısımlar yalnızca Coding'e özeldir.)

| `scene` | Görevler | Hafıza şekli |
|---|---|---|
| `chat` | Küçük konuşmalar, kişisel bilgiler, tercihler, genel Soru-Cevap | Hiyerarşik hafıza (L0–L7) |
| `working` | Verimlilik işleri: dokümanlar, elektronik tablolar, araştırma, finans analizi, içerik üretimi, bilgisayar operasyonları | İki katmanlı hafıza: oturum gerçeği + yeniden kullanılabilir deneyimler |
| `coding` | Yazılım mühendisliği: kodlama, hata ayıklama, test, derleme, API/SDK, betikler, SQL | `working` ile aynı iki katmanlı hafıza + kod sürüm evrimi zinciri |

### 1. Chat: Hafıza Hiyerarşisi

Uzun vadeli kişisel hafıza için yapılandırılmıştır. Ham diyaloglar, katman katman gerçeklere, özetlere, profillere ve daha yüksek düzey bilişsel yapılara dönüştürülür:

| Katman | Adı | İçerik |
|---|---|---|
| L0 | `BASIC_INFO` | Temel bilgiler |
| L1 | `RAW` | Ham diyalog (yalnızca eklemeli) |
| L2 | `FACT` | Atomik gerçekler (sürüm takibiyle, değiştirilemez kayıtlar) |
| L3 | `SUMMARY` | Oturum özetleri |
| L4 | `IDENTITY` | Rezerve |
| L5 | `KNOWLEDGE` | Rezerve |
| L6 | `SCHEMA` | Zihinsel modeller (soyut anlatı şablonları, grafik katmanı) |
| L7 | `INTENTION` | Geleceğe dönük niyetler (son kullanma süresi vardır; süresi dolunca sıradan gerçeğe düşer) |

**Modlar**

| Mod | Ne yapar | Grafik veritabanı | Kullanım durumu |
|------|----------|------------------|-----------------|
| `lite` | Yalnızca ham hafıza için embedding yazma ve arama; LLM çağrısı yok; `scene` ve `layers` yok sayılır | Hayır | Saf vektör RAG, hızlı giriş, sıfır LLM maliyeti |
| `pro` | + LLM çıkarımı + conflict reconcile | İsteğe bağlı (`MEMORY_ENABLE_GRAPH`, varsayılan kapalı) | Standart iş yükleri |
| `ultra` | + System 2 şema türetimi + çapraz alan sweeper (`evolve()` ile tetiklenir) | Evet (her zaman açık) | Tam bilişsel mimari |

### 2. Working: İki Katmanlı Verimlilik Hafızası

Yapılması gereken işleri hatırlamak için kullanılır. Bir araç çağrısı içeren oturum, aynı anda iki katman üretir:

| Katman | Ne kaydeder | Her oturumda | Yapı |
|---|---|---|---|
| **Facts** (`WorkingFact`) | Bu oturum ne yaptığını: görev, yaklaşım, sonuç | Tam olarak 1, eşik yok | `statement`, `subject`, `files`, `details` (orijinal kaynak) |
| **Experiences** | Benzer bir görev tekrar ortaya çıktığında yeniden kullanılabilecek yaklaşımlar | 0–N, değer eşiği geçmeli | `task`, `search_keys`, `solution`, `boundary_envs`, `confidence` |

Fact katmanı "yapılan işin unutulmayacağını" garanti eder; deneyim katmanı "öğrenilen şeyin tekrar kullanılabilir" olmasını sağlar. İki katman birbiriyle bağlantılıdır (`fact.derived_experience_ids` ↔ `experience.source_fact_ids`), eski gerçekler güncellendiğinde yerini yeni olanlara bırakır.

### 3. Coding: İki Katmanlı Hafıza + Sürüm Evrimi Zinciri

Coding, Working ile **aynı iki katmanlı pipeline ve veri şemasını** kullanır. Üç farkı vardır:

| | Working | Coding |
|---|---|---|
| **Çıkarım spesifikasyonu** | Verimlilik görevi | Yazılım mühendisliği spesifikasyonu: konu kod veya karar olmalı; kodu yapıştırmak yerine kesin koordinatlar (dosya / fonksiyon / dal) kaydedilir |
| **Depolama** | `coding_memory.db` + `*_coding_keys` / `*_working_facts` koleksiyonları | `coding_memory_cv.db` + `*_cv` koleksiyonları, working'den tamamen izole |
| **Sürüm evrimi zinciri** | Yok | **Var**: aynı kod ne kadar kez değiştiği ve her değişikliğin ne olduğu |

**Neden sürüm evrimi zinciri var?**: Olgu çıkarımı bilinçli şekilde kodu yapıştırmaz (bütçe daha bilgi içeren koordinatlara ayrılır), bu yüzden daha sonra geri alınan bir sürüm hiç bir olguda görünmez. O sürüm genellikle çalışma alanında ya da HEAD üzerinde bulunmaz; dolayısıyla sonradan okunamaz. Zincir, her versiyonun gerçek diff'ini LLM çağrısı olmadan saklar; bu da kararı vermeden kaçırılan bilgiyi eksiksiz tamamlar.

**Tasarım öne çıkanlar** (`hy_memory/coding/chain_store.py`):

- **Getirici değil dekoratör**: vektör katmanı hangi gerçeklerin döndürüleceğine karar verir; zincir yalnızca her gerçek altında ilgili kodun tüm revizyonlarını ekler (o revizyon için kullanıcının kendi kelimeleri + gerçek diff).
- **Değişiklik kaynakları**: yerel Git diff ( `MEMORY_CODING_CHAIN_REPO_ROOT` ayarlayın ) veya araç izindeki `path` / `old` / `new` değerleriyle deterministik replace eylemleri.
- **Zincir granülerliği ve izolasyonu**: bir thread, tek bir dal, tek bir dosya ve tek bir fonksiyon bloğu için tek bir akışı temsil eder; `user_id` + `workspace_id` ile izole edilir.

---

## Destek

### Depolama Arka Uçları

- Vektör depoları: ChromaDB (varsayılan), Qdrant, FAISS
- Grafik depoları (ultra modu): Kuzu (varsayılan), Neo4j
- Working / Coding meta verileri: yerel SQLite

### Dökümantasyon

- [Kullanım kılavuzu](docs/usage.md) — tam yapılandırma referansı, API ayrıntıları, dağıtım modları
- [Çevre değişkenleri](docs/env_reference.md) — ayarlar ve varsayılan değerler
- [Arama ve Reader kılavuzu](docs/hy_memory_search_skill.md) — reader seçimi, şema kota yönetimi, rerank ve dönüş kanalları
- [Working / Coding sahneleri](docs/scene_coding_usage.md) — üretkenlik hafızası yönlendirme, yanıt şekilleri ve kodlama zinciri davranışı
- [Graph hafıza tasarımı](docs/pro_graph_design.md) — grafik projeksiyonu, rerank, ilişki yolları ve dağıtım sınırları
- [Çevre değişkeni şablonu](.env.example) — doğrudan kopyalanabilir yapılandırma şablonu
- [MemHub README](memhub/README.md) — yerel öncelikli masaüstü ve CLI ürün özeti, çalışma zamanı komutları ve gizlilik sınırları
- [Hermes native memory provider](plugins/hermes/README.md) — açık sahiple/agent alan ayrımı; ayrı HTTP servis; otomatik yakalama ve hatırlama isteğe bağlıdır.
- [OpenClaw eklentisi](plugins/openclaw/README.md) — `plugins/` altında agent entegrasyonları (özet: [plugins/README.md](plugins/README.md))

### Lisans

Hy-Memory Apache-2.0 lisansı altında kullanılır. See [LICENSE](LICENSE); üçüncü taraf bileşenler kendi lisansları ve bildirimleri altında kalır.

