# Hy Memory SDK Ortam Değişkenleri Referansı

Bu belge yalnızca mevcut kodun gerçekten okuduğu ve kamuya açık SDK işlevi için doğrudan etkisi olan yapılandırmaları listeler. Tüm değişkenler süreç ortamı veya `.env` üzerinden ayarlanabilir; tanımlanmazsa tablo içindeki varsayılan değerler kullanılır.

## Temel Yapılandırma

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_MODE` | `pro` | `lite` / `pro` / `ultra` |
| `MEMORY_DATA_DIR` | `~/.hy_memory` | Yerel veri, önbellek ve günlük kökü |

- `lite`: Saf Embedding yazımı, Graph zorunlu kapalı.
- `pro`: LLM çıkarım ve reconcile; Graph `MEMORY_ENABLE_GRAPH` ile kontrol edilir.
- `ultra`: Pro yetenekleri taşır ve Graph zorunlu olarak açılır; System 2 çağıran tarafın açıkça `client.evolve()` çağırmasını bekler; `add()` sonrasında otomatik kuyruğa alınmaz.

## LLM

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_LLM_PROVIDER` | `openai` | OpenAI-uyumlu Chat Completions protokolü |
| `MEMORY_LLM_MODEL` | `gpt-4.1-nano` | Model adı |
| `MEMORY_LLM_API_KEY` | — | Yedek: `LLM_API_KEY`, `OPENAI_API_KEY` |
| `MEMORY_LLM_BASE_URL` | — | Yedek: `LLM_BASE_URL` |
| `MEMORY_LLM_TEMPERATURE` | `0.1` | Evrensel sıcaklık; `none` / `omit` / boş değer `temperature` göndermemek için |
| `MEMORY_LLM_OMIT_TEMPERATURE` | `false` | `true` ise `temperature` alanı tamamen gönderilmez |
| `MEMORY_LLM_MAX_TOKENS` | `1024` | Genel üretim üst sınırı |
| `MEMORY_AGENT_MAX_TOKENS` | `2000` | Extract/Reconcile/System 2 gibi Agent çağrıları için üst sınır |
| `MEMORY_LLM_TIMEOUT` | `180` | Tek istek için zaman aşımı, saniye |
| `MEMORY_LLM_MAX_RETRIES` | `5` | Maksimum yeniden deneme sayısı |
| `MEMORY_LLM_RETRY_DELAY` | `1.0` | Yeniden deneme gecikmesi, saniye |
| `MEMORY_LLM_EXTRA_HEADERS` | — | JSON nesnesi, HTTP header'ları ekler |
| `MEMORY_LLM_EXTRA_BODY` | — | JSON nesnesi, istek body'ine eklenir |
| `MEMORY_FEW_SHOT_ENABLED` | `false` | Dahili Extractor/Reconcile prompt'una few-shot ekler |
| `MEMORY_ENABLE_INTENTION` | `true` | L7 Intention çıkarılıp yazılacak mı |
| `MEMORY_LOG_LLM_CONTENT` | `false` | `true` olduğunda tam prompt/response günlük kaydı yapılır |

## Embedding

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_EMBEDDER_PROVIDER` | `openai` | OpenAI-uyumlu Embeddings protokolü |
| `MEMORY_EMBEDDER_MODEL` | `text-embedding-3-small` | Model adı |
| `MEMORY_EMBEDDER_API_KEY` | — | Yedek: `EMBEDDING_API_KEY`, `OPENAI_API_KEY` |
| `MEMORY_EMBEDDER_BASE_URL` | — | Yedek: `EMBEDDING_BASE_URL` |
| `MEMORY_EMBEDDING_DIMS` | `1536` | Collection boyutu; model çıktısıyla eşleşmelidir |
| `MEMORY_EMBEDDER_TIMEOUT` | `15` | Tek istek için zaman aşımı, saniye |
| `MEMORY_EMBEDDER_MAX_RETRIES` | `2` | Maksimum yeniden deneme sayısı |
| `MEMORY_EMBEDDER_RETRY_DELAY` | `0.3` | Yeniden deneme gecikmesi, saniye |
| `MEMORY_EMBEDDER_EXTRA_HEADERS` | — | JSON nesnesi, HTTP header'ları ekler |
| `MEMORY_EMBEDDER_EXTRA_BODY` | — | JSON nesnesi, istek body'ine eklenir |

## Vector Store

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_VECTOR_STORE` | `chroma` | `chroma` / `qdrant` / `faiss` |
| `MEMORY_COLLECTION_NAME` | `agent_memories` | Ana collection adı |
| `MEMORY_PERSIST_DIR` | `{DATA_DIR}/data/vector_db` | Yerel kalıcı dizin |
| `MEMORY_VECTOR_HOST` | — | Qdrant gibi uzak arka uç ana sunucusu |
| `MEMORY_VECTOR_PORT` | — | Uzak arka uç portu |
| `MEMORY_VECTOR_API_KEY` | — | Uzak arka uç API Anahtarı |

Hybrid anahtar kelime adayları arka uç uygulamasına bağlıdır. Chroma net şekilde boş keyword sonucu döndürür; bu nedenle `hybrid` Chroma üzerinde saf vektör adaylarına düşer. Qdrant gibi sparse/BM25 destekleyen arka uçlar ek adaylar üretir.

## Graph Store

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_ENABLE_GRAPH` | `false` | Yalnızca pro için etkili; lite zorunlu kapalı, ultra zorunlu açık |
| `MEMORY_GRAPH_PROVIDER` | `kuzu` | `kuzu` / `neo4j` |
| `MEMORY_GRAPH_DB_PATH` | `{DATA_DIR}/data/kuzu_db` | Kuzu veritabanı yolu |
| `NEO4J_URL` | `bolt://localhost:7687` | Neo4j Bolt URL; `MEMGRAPH_URL` ile yedeklenebilir |
| `NEO4J_USERNAME` | — | `MEMGRAPH_USERNAME` ile yedeklenebilir |
| `NEO4J_PASSWORD` | — | `MEMGRAPH_PASSWORD` ile yedeklenebilir |
| `NEO4J_DATABASE` | `neo4j` | Neo4j database |

Graph yazma ve okuma tarafı Rerank için gelişmiş anahtarlar için [pro_graph_design.md](pro_graph_design.md) dosyasına bakın.

## Cache

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_CACHE_BACKEND` | `sqlite` | `sqlite` / `mysql`; Redis desteklenmez |
| `MEMORY_CACHE_DB_PATH` | `{DATA_DIR}/data/cache.db` | SQLite dosya yolu |
| `MEMORY_MYSQL_HOST` | `localhost` | MySQL ana sunucu |
| `MEMORY_MYSQL_PORT` | `3306` | MySQL portu |
| `MEMORY_MYSQL_USER` | `root` | MySQL kullanıcı adı |
| `MEMORY_MYSQL_PASSWORD` | — | MySQL şifresi |
| `MEMORY_MYSQL_DATABASE` | `hy_memory` | MySQL veritabanı |
| `MEMORY_MYSQL_POOL_SIZE` | `10` | Bağlantı havuzu boyutu |
| `MEMORY_MYSQL_POOL_RECYCLE` | `3600` | Bağlantı geri dönüş süresi, saniye |

## Yazma, Sahne ve Query Rewrite

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `RECONCILE_ENABLED` | `true` | Aday arama ve reconcile çalıştırılsın mı |
| `MEMORY_ENABLE_SEARCH_QUERY` | `false` | Reconcile öncesi ek LLM ile arama query üretilecek mi |
| `MEMORY_EXTRACT_HISTORY_TURNS` | `5` | Extractor'a eklenen önceki kullanıcı turu sayısı |
| `MEMORY_CODING_ENABLED` | `true` | `working/coding` ayrı akışları açık mı |
| `MEMORY_CODING_DB_PATH` | `{DATA_DIR}/data/coding_memory.db` | Üretkenlik hafıza SQLite yolu |
| `MEMORY_CODING_TOOL_RESULT_MAX_BYTES` | `2048` | Tek tool result için maksimum bayt |
| `MEMORY_CODING_VERBATIM_FALLBACK_ENABLED` | `false` | Çıkarım sonucu yoksa renderlanmış konuşma working fact olarak kaydedilsin mi |
| `MEMORY_ENABLE_QUERY_REWRITE` | `false` | Coding kapalı ise default çoklu-query rewrite kontrolü; Coding açık ise çoklu-query her durumda sınıflandırma ve rewrite yapar |
| `MEMORY_QUERY_REWRITE_MAX_CHARS` | `1000` | default çoklu-query rewrite için karakter koruma limiti |

Varsayılan tek query Search sıfır-LLM hafif aramadır. `search_mode=agentic` altında tek veya çoklu query için tek bir birleştirilmiş sahne değerlendirme, rewrite ve zaman çözümlemesi yapılır. Default çoklu-query, uzunluk koruma limitini aşarsa rewrite yapılmaz; Coding açıkken yalnızca sınırlı sahne sınıflandırması yapılır, kapalıyken LLM kullanılmaz.

## System 2 ve Kullanıcı Profili

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `SWEEPER_ENABLED` | `true` | Ultra evolve cross-domain sweeper çalıştırır mı |
| `MEMORY_SUMMARY_ENABLED_IN_SYS2` | `false` | System 2 materyali L3 Summary içeriyor mu |
| `MEMORY_EVOLING_BALANCED_CATALOG_ENABLED` | `false` | Ultra evolve balanced catalog üretir mi |
| `MEMORY_EVOLING_BALANCED_CATALOG_MAX_FACTS` | `120` | Tek seferde incelenecek maksimum L2 sayısı |
| `MEMORY_EVOLING_BALANCED_CATALOG_CONTENT_CHARS` | `180` | Her L2 için prompt içindeki karakter limiti |
| `MEMORY_EVOLING_BALANCED_CATALOG_MAX_TOKENS` | `512` | Catalog seçim çıktısı üst sınırı |
| `MEMORY_EVOLING_SPECIFIC_SUPPLEMENT_ENABLED` | `false` | Ultra evolve supplement üretir mi |
| `MEMORY_USER_PROFILE_BALANCED_CATALOG_ENABLED` | `false` | `get_user_profile()` active catalog eklesin mi |
| `MEMORY_USER_PROFILE_SPECIFIC_SUPPLEMENT_ENABLED` | `false` | Active supplement eklensin mi |

Catalog ve supplement yalnızca `user_id` ile belirlenir; normal L6 olarak Search içinde görünmezler ve `schema_limit` içine sayılmazlar.

### Cross-domain Sweeper

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_CROSS_DOMAIN_ENABLE` | `true` | Cross-domain çarpışma taraması çalıştırılsın mı |
| `MEMORY_CROSS_DOMAIN_MIN_BASICS` | `5` | Genel çarpışma taramasına giriş için gerekli minimum schema sayısı |
| `MEMORY_CROSS_DOMAIN_BEH_THRESHOLD` | `0.70` | Davranış vektörü benzerlik eşik değeri |
| `MEMORY_CROSS_DOMAIN_CON_THRESHOLD` | `0.60` | İçerik vektörünün çapraz alan farkı eşik değeri |

## Reader ve Yerel Rerank

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `HY_MEMORY_READER` | `legacy` | `legacy` (vektör) / `hybrid` (vektör + arka uç keyword adayları) |
| `MEMORY_HYBRID_V2_VDB_WEIGHT_BM25` | `0.35` | Hybrid BM25 katkı ağırlığı; eski önekin korunduğu yer |
| `MEMORY_LOCAL_RERANK_ENABLED` | `false` | İki Reader tarafından paylaşılan yerel cross-encoder Rerank |
| `MEMORY_LOCAL_RERANK_MODEL` | `nreimers/mmarco-mMiniLMv2-L6-H384-v1` | Küçük çok dilli Hugging Face modeli adı veya yerel yol |
| `MEMORY_LOCAL_RERANK_DEVICE` | `auto` | `auto` / `cpu` / `cuda` gibi cihaz |
| `MEMORY_LOCAL_RERANK_BATCH_SIZE` | `16` | Tek forward işlemi için batch boyutu, istek eşzamanlılığı değil |
| `MEMORY_LOCAL_RERANK_MAX_LENGTH` | `512` | Tokenizer maksimum uzunluğu |
| `MEMORY_LOCAL_RERANK_POOL_FACTOR` | `3` | Aday havuzunun nihai limit ile oranı |
| `HY_MEMORY_TAG_INDEX_WRITE` | `true` | Yazı tarafı Tag Index bakımını yapılıp yapılmayacağı |

SDK, Rerank Search için eşzamanlılık sınırı belirlemez. Etkinleştirildiğinde model `HyMemoryClient` başlatma aşamasında yüklenir ve bir ısınma forward çalıştırır; başlatma başarısız olursa Client ready durumu doğrudan engellenir. Çalışma sırasında tek bir çıkarımın başarısız olması fail-open davranışına göre sürdürülür.
Ayrıntılar için [hy_memory_search_skill.md](hy_memory_search_skill.md) dosyasına bakın.

## Episode ve Özetler

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_EPISODE_GAP_SECONDS` | `1800` | Bu aralığı aşan sürelerde yeni episode başlatılır |
| `MEMORY_EPISODE_MAX_MEMORIES` | `50` | Tek episode için maksimum kullanıcı tarafı hafıza sayısı |
| `MEMORY_EPISODE_SUMMARY_ENABLED` | `false` | Episode sona erdiğinde L3 Summary üretilsin mi |
| `MEMORY_EPISODE_SUMMARY_MIN_MEMORIES` | `3` | Summary için minimum hafıza sayısı |
| `MEMORY_EPISODE_RECALL_ENABLED` | `false` | Search, Episode ilişkilerini ağırlıklandıracak mı |
| `MEMORY_EPISODE_RECALL_WEIGHT` | `0.7` | Episode yapısının destek ağırlığı |
| `MEMORY_TOPIC_SUMMARY_ENABLED` | `false` | Topic Summary üretilecek mi |
| `MEMORY_TOPIC_SUMMARY_MIN_MEMORIES` | `5` | Topic Summary için minimum hafıza sayısı |
| `MEMORY_TOPIC_SUMMARY_STEP` | `5` | Summary yeniden oluşturulmadan önce kaç hafıza eklenecek |
| `MEMORY_SESSION_EPISODE_ENABLED` | `false` | `finalize_session()` ile belirli bir session arşivlenebilsin mi |
| `MEMORY_SESSION_EPISODE_CONTEXT_ENABLED` | `false` | Reader Session Episode bağlamı geri çağırsın mı |
| `MEMORY_SESSION_EPISODE_LIMIT` | `5` | Session Episode aday limiti |
| `MEMORY_SESSION_EPISODE_FACT_LIMIT` | `2` | Her Session Episode için eklenebilecek gerçek sayısı |

## Recall Yardımcı Yetenekler

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_RECALL_STRENGTH_ENABLED` | `false` | Erişim sayısına ve boşta kalma süresine göre normal sonuçlar yeniden sıralansın mı |
| `MEMORY_RECALL_STRENGTH_TAU` | `180` | Bozulan zaman sabiti, gün |
| `MEMORY_RECALL_ACCESS_TRACKING` | `true` | Strength açıkken erişim bilgileri tekrar yazılsın mı |
| `MEMORY_ENTITY_STORE_ENABLED` | `false` | Ayrı entity collection sürdürülür mü; mevcut Reader doğrudan okuma yapmaz |

Eski veriler için mevcut açık yöntem `client.build_entity_store(user_id)` çağrılarak Entity Store yeniden oluşturulabilir.

## Loglar, Trace ve Geçmiş

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `MEMORY_LOG_LEVEL` | `INFO` | Çalışma log seviyesi |
| `MEMORY_LOG_FORMAT` | `text` | `text` / `json` |
| `MEMORY_LOG_DIR` | — | Birleşik günlük kökü |
| `MEMORY_TRACE_LOG_DIR` | — | JSONL log dizini için eski isim; öncelik `MEMORY_LOG_DIR`'e verilir |
| `MEMORY_PIPELINE_LOG_ENABLED` | `true` | Pipeline JSONL günlük yazılsın mı (`{MEMORY_LOG_DIR}/<subdir>/<date>.log`); `false` ise tamamen yazma olmaz |
| `MEMORY_PIPELINE_TRACE_ENABLED` | `true` | Cache içindeki pipeline trace tablosuna yazılsın mı |
| `MEMORY_MEMORY_OPERATIONS_ENABLED` | `false` | Hafıza operasyon denetim kaydı yazılsın mı |
| `MEMORY_HISTORY_ENABLE` | `true` | Geçmiş kaydı açık mı |
| `MEMORY_HISTORY_DB_PATH` | `{DATA_DIR}/data/history.db` | History SQLite yolu |
| `MEMORY_HISTORY_RECORD_SEARCHES` | `false` | Search işlemleri kaydedilsin mi |
| `MEMORY_CODING_CHAIN_REPO_ROOT` | — | coding sürüm zinciri için yerel Git diff ve repo kökü |
| `MEMORY_CODING_CHAIN_BLOCK_ENABLED` | `false` | search(scene=coding) sürüm evrim zinciri bloğu (`chain_block` / diff) eklesin mi; varsayılan kapalı |

## HTTP Sunucusu

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `HY_MEMORY_SERVER_HOST` | `127.0.0.1` | Sunucu dinleme adresi |
| `HY_MEMORY_SERVER_PORT` | `19527` | Sunucu dinleme portu |

Geçmiş konfigürasyonlar etkili olmaz; geçerli seçenekler bu sayfadaki tablolar ve `.env.example` dosyası ile belirlenir. Cache backend şu an yalnızca `sqlite` ve `mysql` destekler.

