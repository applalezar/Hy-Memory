# Graph Hafıza Mimarisi ve İşlemleri

Bu belge mevcut SDK tarafından uygulanan Graph özelliklerini açıklar.
Bu belge, Graph'i etkinleştiren operatörler ve `pipelines/_graph`, `reader_base.py` veya bir GraphStore arka ucu üzerinde çalışan geliştiriciler için tasarlanmıştır.

Graph isteğe bağlı, fail-open bir geliştirmedir. Vektör geri çağrısı asıl kaynak olmayı sürdürür: Graph kullanılamıyorsa, eksikse veya kapalıysa normal hafıza araması çalışmaya devam etmelidir.

## 1. Graph'i etkinleştirme

Graph kullanılabilirliği işleme moduna göre değişir:

| Mod | Graph davranışı |
|---|---|
| `lite` | Devre dışı |
| `pro` | `MEMORY_ENABLE_GRAPH` ile kontrol edilir, varsayılan `false` |
| `ultra` | Etkin |

Varsayılan arka uç gömülü Kuzu'dur. Neo4j `graph` opsiyonel bağımlılığı ile kullanılabilir.

```bash
# Gömülü, tek işlemli dağıtım
MEMORY_MODE=pro
MEMORY_ENABLE_GRAPH=true
MEMORY_GRAPH_PROVIDER=kuzu
MEMORY_GRAPH_DB_PATH=/data/hy-memory/kuzu

# Çok işlemli veya uzak dağıtım
MEMORY_GRAPH_PROVIDER=neo4j
NEO4J_URL=bolt://localhost:7687
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=secret
NEO4J_DATABASE=neo4j
```

Kuzu, aynı veritabanı yolunu birden fazla işlem tarafından açmaya izin vermez. Birden çok işçinin aynı Graph'i paylaşması gerekiyorsa Neo4j kullanın.

## 2. Yazı tarafı projeksiyon

Graph ve `MEMORY_GRAPH_WRITE` açıkken, kalıcı L2 gerçeği Graph'e asenkron şekilde yansıtılır. Projeksiyon hataları loglanır ve VDB yazımını geri almaz.

### 2.1 Düğüm modeli

Bir L2 gerçeği `Memory(layer="l2_fact")` olarak temsil edilir; `VdbRef` değil. Bu sayede düğüm Topic, Entity, ilişki ve zamansal kenarlara katılabilir. `VdbRef`, L6 `DERIVED_FROM` kanıt bağlantılarında kullanılan hafif hedef olarak kalır.

Graph projeksiyonu VDB'yi değiştirmez. İçerik, vektörler, olay zamanı ve normal geri çağrı VDB sahibi olur.

### 2.2 Kenar kaynakları

Mevcut ilişki kaynakları şunlardır:

| İlişki | Kaynak | Ek LLM çağrısı |
|---|---|---|
| `co_occurs` | Tek yazım topluluğunda oluşturulan L2 gerçekler | Hayır |
| `semantic_neighbor` | `MEMORY_GRAPH_EDGE_MIN_SCORE` üzerindeki Reconcile adayları | Hayır |
| `causes` | Reconcile `related` çıktısı | Ek çağrı yok |
| `elaborates` | Reconcile `related` çıktısı | Ek çağrı yok |
| `contrasts` | Reconcile `related` çıktısı | Ek çağrı yok |
| `same_series` | Reconcile `related` çıktısı | Ek çağrı yok |
| `TAGGED_WITH` | Topic düğümlerine yansıtılan hafıza etiketleri | Yeni hafıza LLM çağrısı değil |
| `MENTIONS` | Entity düğümlerine yansıtılan NER varlıklar | Hayır |
| `TEMPORAL_NEXT` | İsteğe bağlı olay-zaman sıralaması | Hayır |

`co_occurs`, O(n²) edge patlamasını önlemek için `MEMORY_GRAPH_CO_OCCURS_MAX_BATCH` ile sınırlandırılır. `semantic_neighbor` ucuzdur ama çoğunlukla vektör benzerlikte zaten mevcut bilginin tekrarıdır; yazılı ve eşzamanlı ilişkiler genelde daha yapısal bilgi taşır.

### 2.3 Yaşam döngüsü senkronizasyonu

Graph durumu artık hafızanın yaşam döngüsü değişikliklerine göre takip edilir:

- ADD yeni aktif gerçeği ve kenarlarını projelendirir.
- UPDATE projelendirilmiş düğümü yeniler.
- SUPERSEDE ve SHADOW eski projeksiyonu aktif geri çağrıdan kaldırır.
- Search eski veya shadowlanmış düğümleri, hatta eski kenarlar kalmış olsa bile filtreler.

Projeksiyon kodu `hy_memory/pipelines/_graph/projector.py` içinde bulunur ve `hy_memory/pipelines/writer.py` üzerinden planlanır.

## 3. Okuma tarafı Graph kullanımı

`legacy` ve `hybrid` reader'lar aynı Graph post-processing'i paylaşır. Reader'lar yalnızca aday geri çağrısında farklıdır: `legacy` yoğun vektör kullanır; `hybrid` ek olarak keyword/BM25 kanalını ekler.

Graph post-processing'i şu şekilde etkinleştirin:

```bash
MEMORY_GRAPH_RERANK_ENABLED=true
```

Tüm Graph işlemleri fail-open davranır. İstisnalar Graph öncesi aday sırasını döndürür; kesin davranış isteği olmayan testlerde bu durum geçerlidir.

### 3.1 A — Support propagation

Support propagation, genişletilmiş geri çağrı havuzunda zaten bulunan adayları yeniden sıralar. Yeni aday eklemez.

Aday `v` için uygulama, geri çağrılan komşulardan sınırlı destek hesaplar ve küçük bir skor ayarlaması uygular:

```text
support(v) = sum(edge_prior(u,v) * score(u)) / sqrt(pool_degree(v))
score'(v)  = score(v) + beta * min(support(v), support_cap)
```

İlgili ayarlar:

```bash
MEMORY_GRAPH_PROPAGATE_BETA=0.15
MEMORY_GRAPH_PROPAGATE_ITERS=1
MEMORY_GRAPH_SUPPORT_CAP=1.0
```

### 3.2 B — Sınırlı ilişki genişletme

İlişki genişletme, başlangıç havuzunun dışında düğümler önerilebilir. Kabul, aşağıdakilerin tümüyle sınırlandırılır:

- traversal hop limiti;
- seed score floor;
- prefilter ve expansion limitleri;
- ilişki tipi kotası;
- semantik Sert kapı `sim(candidate) >= sim(seed) - delta`;
- nihai çok kanallı puanlama.

Kabul edilen öğeler varsayılan olarak seed'e birleştirilir; bu yüzden ekstra top-k slotu tüketmezler.

Temel ayarlar ve mevcut varsayılanlar:

```bash
MEMORY_GRAPH_RELAX_GAMMA=0.3
MEMORY_GRAPH_SEED_FLOOR=0.5
MEMORY_GRAPH_MAX_EXPAND=20
MEMORY_GRAPH_EXPAND_HOPS=2
MEMORY_GRAPH_EXPAND_PREFILTER_TOP_N=50
MEMORY_GRAPH_SEED_SIM_DELTA=0.2
MEMORY_GRAPH_EXPAND_MERGE_SEED=true
```

`MEMORY_GRAPH_RELAX_GAMMA=0` olarak ayarlarsanız support propagation ve relation paths korunurken ilişki genişletme kapatılır.

### 3.3 C — Relation paths

Relation paths, düz hafıza listesinde kaybolacak bağlantı anlamsal değerlerini korur. Paylaşılan reader bunları normal hafızalardan bağımsız şekilde tanılayıcı `ReadResponse.extra["related_paths"]` alanında saklar:

```python
read_response.extra["related_paths"]
```

Her yol, sıralı düğümleri ve ilişki tiplerini içerir. Zamansal kenarlar ayrıca yönü de taşır (`earlier -> later`).

```bash
MEMORY_GRAPH_PATH_ENABLED=true
MEMORY_GRAPH_PATH_HIGH_FLOOR=0.5
```

### 3.4 İlişki tipi öncelikleri

Reader, kenarı hem depolanan ağırlığına hem de ilişki tipi önceliğine göre ağırlıklandırır. Mevcut öncelikler `pipelines/_graph/rerank.py` içinde tanımlanmıştır:

| İlişki | Öncelik |
|---|---:|
| `causes` | 0.90 |
| `elaborates` | 0.90 |
| `same_series` | 0.85 |
| `contrasts` | 0.70 |
| `temporal_next` | 0.70 |
| `co_occurs` | 0.55 |
| `episode` | 0.55 |
| türsiz `related` | 0.50 |
| `semantic_neighbor` | 0.35 |

Daha yüksek öncelikler, vektör benzerliğiyle zaten kodlanmamış bilgiyi sağlayan ilişkilere ayrılmıştır. Derece normalize etme ve support cap, yüksek dereceli düğümlerin sıralamaya domine etmesini engeller.

## 4. Zamansal kenarlar ve episode'lar

`MEMORY_GRAPH_TEMPORAL_SPINE=true`, VDB `memory_at` değerlerini kullanarak yönlü `TEMPORAL_NEXT` kenarları oluşturur. Graph düğümleri kanonik olay zamanına sahip değildir; projeksiyon, spine oluşturulurken zaman damgalarını VDB'den çözer.

Episode'lar Graph'ten ayrı çalışır. `episode_id`, tek bir diyalogu oluşturan hafızaları bir araya getirir ve Graph kapalıyken bile sınırlı aday desteği sağlayabilir.

```bash
MEMORY_EPISODE_RECALL_ENABLED=false
MEMORY_EPISODE_RECALL_WEIGHT=0.7
MEMORY_EPISODE_SUMMARY_ENABLED=false
```

Ayrıntılı Episode ve özet ayarları `docs/env_reference.md` içinde listelenmiştir.

## 5. Backfill ve yeniden inşa

Mevcut VDB gerçekleri şu şekilde projeksiyona sokulabilir:

```python
client.build_memory_graph(
    user_id="u1",
    agent_id=None,      # None tüm agent'ları temsil eder
    clear_first=False,
)
```

Backfill şunları yeniden inşa edebilir:

- L2 Memory düğümleri;
- Topic ve Entity projeksiyonları;
- depolanmış vektörlerden `semantic_neighbor` kenarları;
- isteğe bağlı `TEMPORAL_NEXT` kenarlar.

Backfill, toplu `co_occurs` kenarlarını veya LLM tipli ilişkileri yeniden oluşturamaz; çünkü orijinal yazım topluluğu ve reconcile kararları kurtarılamaz.

Projeksiyon kurallarını değiştirdikten sonra `clear_first=True` kullanın. Bu, seçili L2 projeksiyonunu ve erişimsiz Topic/Entity düğümlerini temizler; ancak System 2 L6 schema'larını silmez.

## 6. Gözlemlenebilirlik

Paylaşılan reader dahili tanılama bilgilerini şu alanlarda saklar:

```python
read_response.extra["graph_rerank"]   # kenar, propagation ve kabul istatistikleri
read_response.extra["related_paths"]  # dekore edilmiş ilişki yolları
```

`HyMemoryClient.search()` şu anda bu iki dahili alanı dışarıya iletmez.
Kamuya açık operasyonel görünürlük için pipeline trace'leri kullanın: read-side Graph işi `READ_GRAPH_RERANK` olarak kaydedilir; yazma projeksiyonu ise `GRAPH_PROJECT` istatistikleri ve hataları raporlar.

## 7. Uygulama haritası

| Sorumluluk | Modül |
|---|---|
| Projeksiyon ve yaşam döngüsü senkronizasyonu | `hy_memory/pipelines/_graph/projector.py` |
| Support, expansion, paths, priors | `hy_memory/pipelines/_graph/rerank.py` |
| Reader entegrasyonu | `hy_memory/pipelines/reader_base.py` |
| Yazma planlaması | `hy_memory/pipelines/writer.py` |
| İlişki çıktısı ayrıştırma | `hy_memory/agent/reconciler.py` |
| GraphStore arayüzü | `hy_memory/data/graph_store_base.py` |
| Kuzu arka ucu | `hy_memory/data/graph_store_kuzu.py` |
| Neo4j arka ucu | `hy_memory/data/graph_store_neo4j.py` |
| Graph backfill API | `HyMemoryClient.build_memory_graph()` |

Bu sözleşmeleri kapsayan testler öncelikle `tests/test_memory_graph.py`, `tests/test_graph_edge_idempotency.py` ve `tests/test_profile_evidence.py` dosyalarında yer alır.

