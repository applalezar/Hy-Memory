# Hy Memory Kullanım Kılavuzu

Bu belge yalnızca mevcut `HyMemoryClient` tarafından açıkça paylaşılan yöntemleri ve dönüş yapısını açıklar.

## Kurulum ve Hızlı Başlangıç

```bash
pip install hy-memory==2.0.4

# İsteğe bağlı arka uçlar
pip install 'hy-memory[qdrant]==2.0.4'
pip install 'hy-memory[faiss]==2.0.4'
pip install 'hy-memory[graph]==2.0.4'     # Neo4j; Kuzu çekirdek bağımlılıklarında zaten mevcut
pip install 'hy-memory[mysql]==2.0.4'
pip install 'hy-memory[rerank]==2.0.4'    # Yerel çok dilli cross-encoder rerank
```

Varsayılan LLM ve Embedding için OpenAI-uyumlu arayüz kullanılır. OpenAI kullanırken yalnızca şunu ayarlamanız yeterlidir:

```bash
export OPENAI_API_KEY="sk-your-key"
```

```python
from hy_memory import HyMemoryClient

with HyMemoryClient(mode="pro") as client:
    client.add(
        "Kullanıcı bilimkurgu filmlerini seviyor, özellikle de \"Yıldızlararası\" filmini.",
        user_id="user_001",
    )
    result = client.search(
        "Kullanıcı ne seviyor?",
        user_ids=["user_001"],
    )
    for channel, memories in result["memories"].items():
        for memory in memories:
            print(channel, memory["score"], memory["content"])
```

## Yapılandırma

SDK proje dizinindeki `.env` dosyasını yükler. Tam yapılandırma için [env_reference.md](env_reference.md) dosyasına bakın.

```bash
MEMORY_MODE=pro
MEMORY_LLM_MODEL=deepseek-chat
MEMORY_LLM_API_KEY=sk-your-key
MEMORY_LLM_BASE_URL=https://api.deepseek.com
MEMORY_EMBEDDER_MODEL=text-embedding-v4
MEMORY_EMBEDDER_API_KEY=sk-your-key
MEMORY_EMBEDDER_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
MEMORY_EMBEDDING_DIMS=1024
```

Ayrıca mevcut açık `from_config()` yöntemini de kullanabilirsiniz:

```python
client = HyMemoryClient.from_config({
    "mode": "pro",
    "llm": {
        "model": "deepseek-chat",
        "api_key": "sk-xxx",
        "base_url": "https://api.deepseek.com",
    },
    "embedder": {
        "model": "text-embedding-v4",
        "api_key": "sk-xxx",
        "base_url": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "embedding_dims": 1024,
    },
    "vector_store": {
        "provider": "chroma",
        "embedding_dims": 1024,
    },
})
```

`from_config()` ile yapılandırırken Embedder ve Vector Store boyutları ayrı ayrı tanımlanan iki alan olarak gelir ve eşit olmalıdır; ortam değişkeni yönteminde tek alan `MEMORY_EMBEDDING_DIMS` kullanılır.

Desteklenen vektör arka uçları: `chroma`, `qdrant`, `faiss`. Graph arka uçları genellikle `kuzu` veya `neo4j` kullanır.

## Client API

Senkron ve asenkron yöntemler aynı parametreleri kullanır. Asenkron yöntemler `async_` önekiyle gelir.

### `add()` / `async_add()`

```python
result = client.add(
    [
        {"role": "user", "content": "Bir film öner"},
        {"role": "assistant", "content": "\"Yıldızlararası\" öneriyorum."},
    ],
    user_id="user_001",
    agent_id="default_agent",
    session_id="default_session",
    source="openclaw",
    scene=None,
    memory_at=None,
    metadata=None,
    meta=None,
    embed_override=None,
)
```

`metadata` ve `meta` aynı alan değil:

- `metadata`: eski çağrıcılar için korunmuş SDK işleme seçeneğidir; dahili `WriteRequest.extra` içine gider ve kalıcı olması veya arama sonuçlarında dönmesi garanti edilmez.
- `meta`: çağıranın iş bağlamı bilgileri olup kalıcı olarak saklanır ve `search()`, `get()` gibi sonuçlarda geri döner.
  Aynı hafıza birden fazla birleştirildiğinde her `meta` yazım sırasına göre liste halinde saklanır.

Gelecekte tekrar okunması gereken veriler `meta` içinde yer almalıdır; `metadata` içine koymayın.

`scene` için açık değerler:

- `None`: Normal metin doğrudan sohbet akışına gider; mesajlar içinde tool varsa hızlı sahne tespiti çalıştırılır.
- `chat`: Zorunlu olarak normal hafıza akışını kullanır.
- `working`: Zorunlu olarak üretkenlik hafıza akışını kullanır.
- `coding`: Zorunlu olarak yazılım mühendisliği varyantına gider ve sürüm evrim zincirini korur.

`working` / `coding` zorunlu hale getirilirse girişte tool mesajı bulunmalıdır; aksi halde chat'e düşer. Normal chat yazımı `success`, `scene`, `memory_id`, `request_id`, `elapsed_ms` vb. alanları döner; üretkenlik zinciri ise iki katmanlı reconcile işlemi, `experience_ids`, `fact_ids` gibi alanları döner; tam yapı için [scene_coding_usage.md](scene_coding_usage.md#4-geri-donus-yapisi) dosyasına bakın.

`mode="lite"` LLM çıkarımı yapmaz; yazılan içerik doğrudan `L1_RAW` şeklinde saklanır. Lite arama katı raw-only vektör aramasıdır: çağıranın hangi `layers`, `scene` veya `search_mode` verdiğine bakılmaksızın sadece `L1_RAW` döner; `working/coding`, Schema veya Intention akışına girmez ve sahne belirleme LLM'i çağrılmaz. `pro` / `ultra` ise kendi katman ve sahne parametrelerini takip eder; varsayılan olarak ham konuşma metni dışa açık değildir.

`source`, çağıranın uygulama kaynağıdır; örneğin `openclaw`, `cursor`, `codex`. Bu değer `agent_id` ile dikey değildir: birincisi "hangi ürün/kapıdan yazılım girdiği"ni belirtir, ikincisi hala hafıza izolasyonundaki agent kimliğini temsil eder. `source` `strip().lower()` çalıştırılarak saklanır ve chat, working, coding hafızalarıyla birlikte döner.

### `search()` / `async_search()`

Normal sohbet araması:

```python
result = client.search(
    query="Kullanıcı ne seviyor?",
    user_ids=["user_001"],
    agent_ids=None,
    session_ids=None,
    include_sources=["openclaw", "codex"],
    exclude_sources=["cursor"],
    limit=10,
    min_score=0.4,
    schema_limit=10,
    schema_min_score=0.4,
    intention_limit=0,
    reader="",                  # HY_MEMORY_READER; varsayılan legacy
    layers=None,
    created_after=None,
    search_mode="default",
)
```

Kaynak filtreleme semantiği:

```text
source IN include_sources AND source NOT IN exclude_sources
```

Herhangi bir liste boşsa ilgili koşul atlanır; ikisi çakışırsa kara liste önceliklidir. Chroma/Qdrant/Graph veritabanı aday kesmeden önce filtreleme uygular; FAISS, arama öncesinde indeks içinde ID selector kullanarak adayları sınırlar, çoklu geri çağrı ya da fazla top-k alma-sonrası filtreleme yöntemini kullanmaz. Çoklu kaynak kanıtlarından türetilen birleştirilmiş hafıza `mixed` olarak işaretlenir; herhangi bir source filtresi açıldığında varsayılan olarak `mixed` hariç tutulur; yalnızca `mixed` açıkça beyaz listeye eklenirse geri döner.

Chat başarılı yolunda dönen yapı:

```python
{
    "request_id": "...",
    "memories": {
        "normal": [],      # L2/L3 normal hafızalar; L4/L5 şu anda kullanılmıyor
        "schema": [],      # L6 Schema
        "proactive": [],   # L7 Intention; varsayılan kapalı
    },
    "rewrite": {
        "used": False,
        "original_query": "Kullanıcı ne seviyor?",
        "effective_query": "Kullanıcı ne seviyor?",
        "created_after": None,
        "scene": "normal",
    },
    "elapsed_ms": 12.3,
}
```

Reader çalışmazsa boş `memories` üç kovası ve `request_id` / `elapsed_ms` döner; mevcut yedekleme durumunda `rewrite` dahil edilmez.

L0 kullanıcı profili Search içinde yer almaz. `schema_limit` yalnızca L6 için geçerlidir ve L0 alanı için yer tutmaz.

Search aynı zamanda ayrı üretkenlik hafıza deposuna da yönlendirilebilir:

```python
result = client.search(
    "Bu istisna en sonunda nasıl düzeltildi?",
    user_ids=["user_001"],
    scene="coding",
    workspace_id="repo-id",
    branch="main",
)
```

`working` / `coding` yolu `scene="productivity"`, `facts`, `experiences` döner; chat üç kovası yapısını kullanmaz; `coding` ayrıca `chain_block` ve `chain_used` alanlarını döner (varsayılan boş; `MEMORY_CODING_CHAIN_BLOCK_ENABLED=true` olduğunda diff bloğu eklenir).

#### `search_mode`

| `search_mode` | `scene` | Davranış |
|---|---|---|
| `default` | `None` + tek `query` | Sabit chat/normal, LLM kullanılmaz |
| `default` | `None` + `queries=[...]` | Davranış Coding ve Query Rewrite anahtarlarının birleşimine göre belirlenir; aşağıdaki tabloya bakın |
| `agentic` | `None` + tek/çoklu query | Tek LLM çağrısıyla sahne kararı, yeniden yazma ve zaman çözümlemesi yapılır |
| Herhangi biri | açık `chat/working/coding` | Doğrudan yönlendirilir; judge/rewrite LLM çağrılmaz |

`query` ile `queries` birbiriyle karşılıklı olarak seçilir; `queries` son öğesi hedef query’dir, önceki öğeler bağlam çözümü için kullanılır.
LLM veya çözümleme başarısız olursa chat/normal ve orijinal query’ye düşülür.

Varsayılan çoklu-query, toplam karakter sayısı `MEMORY_QUERY_REWRITE_MAX_CHARS` değerini aşmıyorsa:

| `MEMORY_CODING_ENABLED` | `MEMORY_ENABLE_QUERY_REWRITE` | Davranış |
|---|---|---|
| `true` | herhangi | Tek LLM ile `chat / working / coding` sınıflandırması ve query rewrite yapılır |
| `false` | `true` | Tek LLM ile rewrite ve zaman analizi yapılır; sahne sınıflandırması yapılmaz |
| `false` | `false` | LLM kullanılmaz; sabit chat/normal, son query kullanılır |

Uzunluk koruması aşıldığında rewrite yapılmaz; Coding açıkken yalnızca sınırlı sahne sınıflandırması yapılır, kapalıyken LLM kullanılmaz.

### `get_user_profile()` / `async_get_user_profile()`

Kullanıcı profili yalnızca `user_id` ile tekil olarak belirlenir; `agent_id` veya `session_id` kabul edilmez:

```python
result = client.get_user_profile(user_id="user_001")
profile = result["profile"]
```

```python
{
    "request_id": "...",
    "profile": {
        "memory_id": "...",
        "user_id": "user_001",
        "content": "...",
        "layer": "l0_basic_info",
        "basic_info": {"name": "Alice"},
        "catalog": {"attached": False},
        "supplement": {"attached": False},
        "tags": [],
        "memory_at": 1234567890,
        "gmt_created": 1234567890,
    },
    "elapsed_ms": 1.2,
}
```

L0 yoksa ve eklenebilir balanced catalog da yoksa `profile` `None` olur. Bu arayüz Embedding veya LLM çağrısı yapmaz. İlgili anahtar açıkken active catalog ve supplement aynı profille eklenir; catalog temel L0 olmadan da salt okunur sentez profili oluşturabilir.

### CRUD, listeleme ve klonlama

Aşağıdakiler şu an açıkça yayınlanan yöntemlerdir:

```python
memory = client.get("memory_id")
# ham ID L1_RAW, fact ID ilgili gerçek katman döner.
raw_memory = client.get("raw_id")
result = client.update("memory_id", "Güncellenmiş içerik")
result = client.delete("memory_id")
result = client.delete_all(user_id="user_001")

page = client.list_memories(
    user_id="user_001",
    limit=100,
    offset=0,
    order="desc",
    layers=["l1_raw"],  # açıkça raw listelenir; verilmezse varsayılan anlamsal katman listesi kullanılır
    with_evolution=False,
)

result = client.clone_user("user_001", "user_002", agent_id=None)
```

Karşılık gelen asenkron yöntemler: `async_get()`, `async_update()`, `async_delete()`, `async_delete_all()`, `async_list_memories()`, `async_clone_user()`.

`get()` / `async_get()` tek bir ID için evrensel okuma yöntemidir: dönüş sonuçlarındaki `layer`, `kind`, `variant` gerçek türü gösterir. Yetkilendirme SDK içinde değil, ürün sınırında yapılır; SDK layer üzerinden tahminde bulunmaz. `list_memories()` varsayılan olarak `L1_RAW` dahil etmez; açıkça `layers=["l1_raw"]` verildiğinde aynı liste sözleşmesiyle raw döner.

Normal `list_memories()` `vdb` kovasını döner ve Graph sonuçları varsa buna `graph` ekler; `workspace_id` verildiğinde üretkenlik hafızası için `scene="productivity"` ve `experiences` döner.

### `evolve()` / `async_evolve()` (Ultra)

System 2 `add()` içinde otomatik tetiklenmez; şu anda `get_write_status()` arayüzü de yoktur. Çağıran taraf toplu yazma tamamlandıktan sonra açıkça şunu çalıştırır:

```python
result = client.evolve(
    user_id="user_001",
    agent_id="default_agent",
)
```

`evolve()` yalnızca `mode="ultra"` için geçerlidir; senkron yöntem bu kez bilişsel işleme tamamlanana kadar bloklar. Mevcut VDB’nin tam içeriğini okur, L6 Schema oluşturur veya günceller ve yapılandırma tarafından izin verilen System 2 işlemleri çalıştırır.

### Diğer şu an açık yöntemler

- `finalize_session()` / `async_finalize_session()`: ilgili oturumu, etkinleştirilmiş özellik anahtarları açıkken arşivler.
- `get_user_basic_info()` / `async_get_user_basic_info()`: L0 basic info kaydını okur.
- `sweep()` / `async_sweep()`: Ultra altında cross-domain sweeper'ı açıkça çalıştırır.
- `get_metrics()` / `async_get_metrics()`: SDK metriklerini okur.
- `build_memory_graph()` / `async_build_memory_graph()`: Normal hafıza ilişki grafiğini doldurur.
- `build_entity_store()` / `async_build_entity_store()`: Bağımsız Entity Store oluşturur.
- `history()` / `async_history()`, `get_recent_history()` / `async_get_recent_history()`.

## Embedding uç noktası kapsamı

`add()` / `async_add()` / `search()` / `async_search()` her çağrı için `embed_override` destekler:

```python
override = {
    "base_url": "http://embedding-host:8000/v1",
    "api_key": "sk-xxx",
    "extra_headers": {"X-Route": "direct"},
    "timeout": 15,
}
client.add("Kullanıcı basketbolu seviyor", user_id="u1", embed_override=override)
client.search("İlgi alanları", user_ids=["u1"], embed_override=override)
```

Bu yalnızca erişim noktasını değiştirir; model, boyut veya `extra_body` alanlarını değiştirmez. Yeni uç nokta, daha önce depolanan verilerle uyumlu bir Embedding modeli çalıştırmalıdır.

## Veri izolasyonu ve işleme modları

Normal hafızalar `user_id → agent_id → session_id` üç katmanlı etki alanı kullanır. Search çağrısında `user_ids` zorunludur; `agent_ids` / `session_ids` verilmezse ilgili boyutta çapraz arama yapılabilir.

Kullanıcı profili istisnadır: L0, balanced catalog ve specific supplement üretim ve okuma aşamasında yalnızca `user_id` tarafından tekil olarak belirlenir.

- `lite`: Ham metin vektörleştirilir, çıkarım LLM çağrılmaz.
- `pro`: Embedding + LLM çıkarımı + reconcile; Graph `MEMORY_ENABLE_GRAPH` ile açılabilir.
- `ultra`: Pro yeteneklerini taşır ve Graph'i zorunlu olarak açar; System 2 çağıran tarafın açıkça `evolve()` çağırması gerekir.

Reader, Rerank ve Search parametreleri için [hy_memory_search_skill.md](hy_memory_search_skill.md) dosyasına bakın.

