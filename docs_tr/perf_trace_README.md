# Performans ve Pipeline Gözlemlenebilirliği

Hy Memory üç tür tamamlayıcı gözlem verisi sunar:

1. `TRACE_PERF`: Yazma zincirindeki INFO günlükleri; tek istek için hızlı `grep` için uygundur.
2. Pipeline logları: Aşama bazlı yapılandırılmış kayıtlar; JSONL olarak yazılabilir ve `MemoryInspector` ile kalıcı trace sorgulanabilir.
3. Sistem metrikleri: Zaman penceresine göre toplam iş yükü, gecikme ve depolama operasyon istatistikleri.

Üçü de `request_id` ile isteği ilişkilendirir; ancak kullanım ve alanlar farklıdır.

## 1. TRACE_PERF logları

Yazma zinciri aşağıdaki formatta INFO logu üretir:

```text
TRACE_PERF [<request_id>] <STAGE> <key=value ...>
```

Şu anda olası System 1 stage'leri:

| Stage | Anlam | Ana alanlar |
|---|---|---|
| `S1_START` | Gerçek yazma işlemi başlıyor | `waiting`, `user` |
| `S1_L1_DONE` | L1_RAW embedding ve persist aşaması bitti | `l1_ms` |
| `S1_EXTRACT_DONE` | Extractor tamamlandı | `extract_ms`, `summary_ms`, `memory` |
| `S1_RECONCILE_DONE` | Reconcile kararı tamamlandı | `ops`, `candidates` ve her op sayısı |
| `S1_UPDATE_BATCH_DONE` | Toplu UPDATE tamamlandı | `nodes` |
| `S1_ADD_BATCH_DONE` | Toplu ADD tamamlandı | `nodes` |
| `S1_INTENTION_DONE` | L7 intention yazımı tamamlandı | `intentions` |
| `S1_PERSIST_DONE` | Çıkarım sonucu kalıcı hale getirildi | `ops` |
| `S1_WORKFLOW_DONE` | System 1 workflow tamamlandı | `workflow_ms` |

Her istek tüm stage'leri üretmez. Örneğin lite mod, boş çıkarım, intention yoksa veya toplu ADD/UPDATE yoksa ilgili loglar görülmez.

İstek bazında bakmak için:

```bash
grep 'TRACE_PERF \[<request_id>\]' app.log
```

`hy_memory/core/workload.py` dosyası da `TRACE_PERF` ön ekiyle workload kuyruğu logları üretir; ancak format yukarıdaki `[request_id] STAGE` biçimi değildir. Analiz ederken bunları birbirinden ayırmak gerekir.

### Yazma dönüşündeki timing

Chat yazımı mümkünse System 1 timing bilgilerini döner:

```python
result = client.add("User likes coffee", user_id="u1")
print(result.get("timing", {}))
```

Alanlar şunları içerir:

```text
sys1_waiting_ms
sys1_l1_process_ms
sys1_workflow_ms
sys1_ops_avg_ms
```

Bunlar aşama süreleridir, sabit hizmet seviyesi eşikleri değildir. Uygun aralık LLM, embedding, vektör veritabanı, makine kaynakları ve eşzamanlılık seviyesine göre değişir.

## 2. Pipeline logları

Pipeline logları iki tür çıktıya sahiptir; alan sözleşmeleri farklıdır.

JSONL dosyasındaki ana alanlar:

```text
request_id, namespace, user_id, agent_id, session_id,
step, prompt, content, parsed, memory_ids,
tokens, elapsed_ms, timestamp
```

Inspector tarafından okunan kalıcı Pipeline Trace ana alanlar:

```text
request_id, namespace, user_id, agent_id, session_id,
step, prompt, response, parsed, memory_ids,
elapsed_ms, prompt_tokens, completion_tokens, total_tokens,
event_at_ms, created_at
```

### İki çıktı türü

- JSONL log varsayılan olarak açıktır; dizin önceliği: `MEMORY_LOG_DIR` → `MEMORY_TRACE_LOG_DIR` → `{MEMORY_DATA_DIR}/logs/pipeline`.
- Kalıcı trace, `MEMORY_PIPELINE_TRACE_ENABLED` ile kontrol edilir; varsayılan açık ve cache backend'a yazılır; Inspector tarafından sorgulanabilir.

İki yazma türü de best-effort'tir; gözlem yazımı başarısız olsa ana okuma/yazma akışını durdurmamalıdır.

Varsayılan olarak `MEMORY_LOG_LLM_CONTENT=false`; `prompt` ve LLM çıktıları (JSONL içindeki `content`, kalıcı Trace içindeki `response`) yalnızca uzunluk ve özet hash olarak saklanır. Sadece açıkça `true` olarak ayarlanınca tam giriş ve çıktı kaydedilir; üretim ortamında etkinleştirmeden önce hassas veri riski değerlendirilmelidir.

### Yaygın step'ler

Yazma zincirinde sık görülen step'ler:

```text
S1_EMBED
S1_L1_UPSERT
EXTRACT
SEARCH_QUERY
RECONCILE_CANDIDATES
RECONCILE
RECONCILE_APPLY
TOOL_CALLS
WRITE_SUMMARY
S1_AGENT_FAILED
S1_L1_ROLLBACK
```

working/coding zincirinde sık görülen step'ler:

```text
CODING_JUDGE
CODING_EXTRACT
CODING_RECONCILE
READ_CODING_EMBED
READ_CODING_VDB
READ_CODING_RESULT
```

Chat arama tarafı read-side step'leri `READ_` önekiyle başlar. Şu an tanımlı olanlar:

```text
READ_SCENE_JUDGE
READ_REQUEST
READ_EMBED_QUERY
READ_INTENT
READ_RECALL_VEC
READ_HIDDEN_L0_CONTEXT
READ_KEYWORD_EMBED
READ_TAG_MATCH
READ_RECALL_TAG
READ_BM25
READ_RRF
READ_MERGE_SCHEMA
READ_EVOLUTION
READ_INTENTION
READ_GRAPH_RERANK
READ_SUMMARY
```

Gerçek step'ler reader, sahne, yapılandırma ve geri çağrı sonucuna bağlıdır; her aramada tam liste bulunmaz.

### MemoryInspector kullanımı

`MemoryInspector` kamuya açık bir API'dir:

```python
from hy_memory import HyMemoryClient, MemoryInspector

client = HyMemoryClient(mode="pro")
inspector = MemoryInspector(client)

write_result = client.add("User likes coffee", user_id="u1")
write_trace = inspector.get_full_trace(write_result["request_id"])

search_result = client.search("What does the user like?", user_ids=["u1"])
read_trace = inspector.get_read_trace(search_result["request_id"])

extract_logs = inspector.get_pipeline_logs(
    request_id=write_result["request_id"],
    step="EXTRACT",
)
```

Asenkron çağrılar karşılığı: `async_get_full_trace()`, `async_get_read_trace()`, `async_get_pipeline_logs()`.

## 3. Sistem metrikleri

İşlem düzeyi metrics şu anda toplanır:

- System 1 started, completed, failed ve active sayıları; her aşama için ortalama süre.
- Son 60 saniyede tamamlanan istek sayısı.
- VDB ve Graph operasyon sayıları ile ortalama süreleri.

Dönüş şemasında `active_sys2`, `queued_sys2`, `sys2_requests` ve System 2 latency alanları korunur; ancak mevcut üretim zinciri `sys2_enqueue()`, `sys2_start()`, `sys2_end()` çağırmadığı için ...

Açık SDK:

```python
metrics = client.get_metrics(minutes=10)
metrics = inspector.get_system_metrics(minutes=10)
metrics = await client.async_get_metrics(minutes=10)
```

HTTP Sunucu girişi:

```text
GET /api/v1/metrics?minutes=10
```

Ana dönen alanlar:

```text
uptime_seconds
window_minutes
requests
avg_latency_ms
throughput
storage_ops
sys2_requests
since
```

Ortalama gecikme veya depolama alanları örnek yoksa boş olabilir. Metrikler dakika bazında toplanır; cache backend ilgili yöntemleri destekliyorsa kalıcı hale getirilir, arka planda her dakika flush yapılır ve 7 gün önceki veriler temizlenir. active/queued gibi alanlar aynı zaman penceresindeki canlı işlere ilişkin sayıları gösterir.

## 4. Hedefli doğrulama

```bash
python -m pytest -q \
  tests/test_pipeline_observability.py \
  tests/test_op_trace_logs.py \
  tests/coding/test_coding_pipeline_log.py \
  tests/test_shared_runtime.py
```

