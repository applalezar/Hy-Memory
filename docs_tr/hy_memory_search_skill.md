# Hy Memory Search ve Reader Rehberi

Bu belge mevcut `HyMemoryClient.search()` / `async_search()` için açık parametreleri ve dönüş sözleşmesini açıklar.

## Reader seçimi

```python
client.search("query", user_ids=["u1"], reader="hybrid")
client.search("query", user_ids=["u1"])  # HY_MEMORY_READER; varsayılan legacy
```

Şu anda yalnızca şunlar desteklenir:

- `legacy`: Aday geri çağrı vektör araması kullanır.
- `hybrid`: Vektör adayına ek olarak arka uç keyword/BM25 adaylarını ekler.

Bilinmeyen isimler warning logu yazıp `legacy`'e düşer. İki Reader aynı paylaşılan okuma hattını kullanır; tek fark keyword aday geri çağrısı yapıp yapmamalarıdır. Schema, Intention, Episode, Tag/Entity, Graph, evrim zinciri, yerel Rerank, kota ve açık sonuç formatı aynıdır.

Keyword kanalı Vector Store'a dayanır. Chroma açıkça boş keyword sonucu döndürür; bu yüzden `hybrid` Chroma üzerinde saf vektör aday geri çağrısına düşer. Keyword veya sparse/BM25 destekleyen arka uçlar ek aday üretir.

## Chat arama parametreleri

```python
result = client.search(
    query="Kullanıcı ne seviyor?",
    user_ids=["u1"],
    agent_ids=None,
    session_ids=None,
    include_sources=["openclaw", "codex"],
    exclude_sources=["cursor"],
    limit=10,
    min_score=0.4,
    schema_limit=10,
    schema_min_score=0.4,
    intention_limit=0,
    created_after=None,
    layers=None,
    reader="",
)
```

| Parametre | Mevcut anlam |
|---|---|
| `limit` | `normal` kanalının maksimum dönen sayısı, varsayılan `10` |
| `min_score` | `normal` kanalının minimum puanı, varsayılan `0.4` |
| `schema_limit` | L6 Schema için ayrı limit, varsayılan `10`; `0` kapatır, negatif değer sınırsız demektir |
| `schema_min_score` | L6 pozitif anlamsal geri çağrı eşiği, varsayılan `0.4`; ters kanıtlarla gelen Schema için Client katmanında bu değer yeniden filtrelenmez |
| `intention_limit` | L7 Proactive için ayrı limit, varsayılan `0` yani kapalı |
| `created_after` | Olay zamanının alt sınırı, Unix timestamp |
| `layers` | Yalnızca belirtilen MemoryLayer geri çağrılır; `l1_raw` Search tarafından dışarıya döndürülmez |
| `reader` | Bu çağrı için Reader geçersiz kılınır; boş string `HY_MEMORY_READER` değerini kullanır |
| `include_sources` | Çağrı kaynağı beyaz listesi; boş liste sınırsız anlamına gelir |
| `exclude_sources` | Çağrı kaynağı kara listesi; beyaz liste ile çakışırsa kara liste önceliklidir |

`query` ve `queries` birbirinden seçilir; `user_ids` boş olmayan liste olmalıdır.

L0 basic info Search'den ayrıldı:

```python
profile = client.get_user_profile(user_id="u1")
```

Bu arayüz yalnızca `user_id` ile tam profil okur. Ultra catalog/supplement de yalnızca bu arayüzden eklenir ve `schema_limit` altında sayılmaz.

## Search modu ve üretkenlik yönlendirme

- `search_mode="default"` + tek `query`: sabit chat/normal, sıfır LLM.
- `search_mode="default"` + `queries=[...]`: Coding açıkken tek LLM ile sahne sınıflandırması ve rewrite; Coding kapalı ama Query Rewrite açıkken yalnızca rewrite/time; ikisi de kapalıysa sıfır LLM.
- `search_mode="agentic"`: tek veya çoklu query için tek LLM ile sahne değerlendirmesi, query rewrite ve zaman aralığı çözümlemesi.
- Açık `scene="chat" | "working" | "coding"`: judge/rewrite LLM atlanır.

Default çoklu-query, `MEMORY_QUERY_REWRITE_MAX_CHARS` aşıldığında rewrite yapılmaz: Coding açık ise yalnızca sınıflandırma yapılır, kapalıysa son query kullanılır.

`working` / `coding` rotasından sonra Search ayrı üretkenlik hafıza veritabanını okur ve `facts` / `experiences` döner; chat üç kanal yapısını kullanmaz. `coding` ayrıca `chain_block` / `chain_used` döner (varsayılan boş; `MEMORY_CODING_CHAIN_BLOCK_ENABLED` açıkken diff bloğu eklenir).

## Chat başarılı dönüş yapısı

```json
{
  "request_id": "...",
  "memories": {
    "normal": [],
    "schema": [],
    "proactive": []
  },
  "rewrite": {
    "used": false,
    "original_query": "Kullanıcı ne seviyor?",
    "effective_query": "Kullanıcı ne seviyor?",
    "created_after": null,
    "scene": "normal"
  },
  "elapsed_ms": 123.4
}
```

Reader başarısız olsa bile boş `memories` üç kovası, `request_id` ve `elapsed_ms` döner; mevcut fail-back durumunda `rewrite` alanı dahil edilmez.

Her açık sonuç için sabit temel alanlar: `memory_id`, `content`, `score`, `layer`, `user_id`, `agent_id`, `source`, `retrieval_source`, `owner`, `tags`, `memory_at`, `gmt_created`, `meta`. `source` yazım uygulama kaynağıdır; `retrieval_source` iç geri çağrı kanalı (ör. VDB / Schema / Episode) tanılayıcı etiketidir. Bazı yetenekler `evolution_chain`, `episode_id`, `capsule_memories`, `timeline` veya `rerank_score` gibi alanlar da ekleyebilir.

Reader içindeki `_semantic`, `_bm25` gibi özel puanlar `HyMemoryClient.search()` tarafından tek bir hafıza alanı olarak dışarıya verilmez; daha detaylı geri çağrı tanısı için pipeline trace kullanılmalıdır.

Kaynak filtresi, alt seviye backend tarafında `limit` / Top-K öncesinde yapılır: Qdrant aynı payload filter kullanır, Chroma metadata `where` kullanır, FAISS ID selector kullanır, Neo4j/Kuzu Graph önce özellik filtresi sonra benzerlik hesaplar. Kaynak bazlı çoklu geri çağrı yapılmaz; over-fetch sonrası filtreleme kullanmaz. Çok kaynaklı türetilmiş hafızalar `source="mixed"` olarak işaretlenir; herhangi bir source filtresi açıkken varsayılan fail-closed olarak dışarıda tutulur; yalnızca beyaz listedeki `mixed` açıkça dahil edilirse döner.

## Yerel Rerank

Yerel cross-encoder Rerank iki Reader tarafından paylaşılan son işlem olarak çalışır ve varsayılan kapalıdır:

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_LOCAL_RERANK_ENABLED` | `false` | Yerel Rerank açık mı |
| `MEMORY_LOCAL_RERANK_MODEL` | `nreimers/mmarco-mMiniLMv2-L6-H384-v1` | Küçük çok dilli Hugging Face modeli adı veya yerel yol |
| `MEMORY_LOCAL_RERANK_DEVICE` | `auto` | `auto` / `cpu` / `cuda` gibi cihaz |
| `MEMORY_LOCAL_RERANK_BATCH_SIZE` | `16` | Tek model forward için batch boyutu |
| `MEMORY_LOCAL_RERANK_MAX_LENGTH` | `512` | Tokenizer maksimum uzunluğu |
| `MEMORY_LOCAL_RERANK_POOL_FACTOR` | `3` | Aday havuzunun nihai `limit` ile oranı |

Ayrıca Client yapılandırmasıyla da etkinleştirilebilir:

```python
from hy_memory import HyMemoryClient

client = HyMemoryClient.from_config({
    "recall": {
        "local_rerank_enabled": True,
        "local_rerank_model": "nreimers/mmarco-mMiniLMv2-L6-H384-v1",
        "local_rerank_device": "auto",
        "local_rerank_batch_size": 16,
        "local_rerank_max_length": 512,
        "local_rerank_pool_factor": 3,
    },
})
```

İsteğe bağlı bağımlılık kurulum:

```bash
pip install 'hy-memory[rerank]==2.0.4'
```

Etkinleştirildiğinde `HyMemoryClient(...)` oluşturma aşamasında model indirme/okuma, device yerleştirme ve bir dummy forward yapılır; oluşturma başarılı olduktan sonra Search tekrar model yükleme yapmaz. Başlatma başarısız olursa doğrudan `RuntimeError` fırlatılır; böylece servis hazır gibi görünürken ilk kullanıcı isteğinde sorun çıkarılmaz. Çalışma sırasında tek bir inference başarısız olursa veya geçersiz skor gelirse fail-open davranışı sürdürülür ve Rerank öncesi sıra korunur.

SDK, Rerank Search için eşzamanlılık üst sınırı belirlemez ve model forward'ını da sıralı hale getirmez; istek eşzamanlılığı çağıran taraf veya dağıtım katmanı tarafından kontrol edilir. `MEMORY_LOCAL_RERANK_BATCH_SIZE` yalnızca tek forward için batch boyutunu ayarlar, eşzamanlılık sınırı değildir.

