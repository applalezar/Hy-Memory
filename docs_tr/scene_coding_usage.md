# Çalışma ve Kodlama Sahnesi

Hy Memory'nin açık `scene` parametresi üç değeri destekler:

| `scene` | Amaç | Depolama ve arama yolu |
|---|---|---|
| `"chat"` | Konuşma, kişisel gerçekler, genel soru-cevap | Genel hafıza pipeline |
| `"working"` | Belge, ofis, araştırma, finans, içerik üretimi, bilgisayar operasyonları gibi üretkenlik görevleri | working çift katman hafıza |
| `"coding"` | Kodlama, hata ayıklama, test, derleme, API/SDK, script, SQL gibi yazılım mühendisliği işleri | coding çift katman hafıza ve sürüm evrim zinciri |

`working` ve `coding`, aynı üretkenlik pipeline yapısını ve veri şemasını paylaşır; fakat farklı çıkarım kuralları, SQLite dosyaları ve vektör koleksiyonları kullanırlar; veriler karışmaz.

## 1. Yazma yönlendirmesi

### `scene` açıkça belirtilirse

Üretkenlik yazımı tool message veya assistant `tool_calls` içermelidir:

```python
from hy_memory import HyMemoryClient

client = HyMemoryClient(mode="pro")

messages = [
    {"role": "user", "content": "Arama istemcisindeki timeout sorunu düzelt."},
    {
        "role": "assistant",
        "content": "",
        "tool_calls": [
            {
                "id": "call_1",
                "type": "function",
                "function": {
                    "name": "apply_patch",
                    "arguments": '{"path":"src/search.py"}'
                },
            }
        ],
    },
    {
        "role": "tool",
        "tool_call_id": "call_1",
        "name": "apply_patch",
        "content": "Patch applied successfully",
    },
]

result = client.add(
    messages,
    scene="coding",
    user_id="user_123",
    workspace_id="github.com/example/service",
    branch="fix/search-timeout",
)
```

Açık davranış:

- `scene="chat"`: scene judge atlanır; tool izleri kaldırıldıktan sonra chat pipeline'a girer.
- `scene="working"`: scene judge atlanır; working pipeline'a girer.
- `scene="coding"`: scene judge atlanır; coding pipeline'a girer ve sürüm evrim zincirini kaydetmeye尝试 eder.
- Açık `working/coding` ama girişte tool/tool_calls yoksa mevcut uygulama chat pipeline'a düşer.
- Geçersiz scene `ValueError` fırlatır.

### `scene` belirtilmezse

`scene=None` durumda:

- Girişte tool/tool_calls yoksa doğrudan chat pipeline'a girer; scene judge LLM çağrılmaz.
- Girişte tool/tool_calls varsa kullanıcı sorgusu ve araç adının sınırlı özeti üzerinden `chat / working / coding` üçlü sınıflandırma yapılır.
- Judge başarısız veya parse edilemezse chat'e düşülür.
- Chat olarak karar verildiğinde tool izleri kaldırılır ve chat pipeline'a girilir.

Sınıflandırma sınırları:

- `coding`: Kod veya üretim çıktısı işin konusu veya uygulama aracıdır; kaynak kodu, script, SQL, notebook, test, derleme, derleme/çalıştırma hataları, API/SDK ve kod incelemeleri buna girer.
- `working`: Açık üretkenlik işlemi veya teslimatı vardır ancak kod ana iş ortamı değildir; örneğin belge, PPT, tablo, çeviri, finans/iş analizi, araştırma raporu, e-posta takvimi, proje yönetimi gibi durumlar.
- `chat`: Konuşma, kişisel bilgi, öneri, genel gerçek soru-cevap. Yalnızca arama/browser aracı kullanılmakla otomatik working olmaz.

## 2. Arama yönlendirmesi

`scene` açıkça verildiğinde ilgili arama yolu doğrudan seçilir; judge/rewrite LLM çağrılmaz:

```python
result = client.search(
    "Sonunda hangi timeout uygulaması korundu?",
    scene="coding",
    user_ids=["user_123"],
    workspace_id="github.com/example/service",
    branch="fix/search-timeout",
    limit=5,
)
```

Varsayılan ve agentic davranış:

| Çağrı şekli | Mevcut davranış |
|---|---|
| `search(query="...", search_mode="default")` | Sabit chat/normal hafif arama, sıfır LLM |
| `search(queries=[...], search_mode="default")` | Coding açıkken sınıflandırma ve rewrite; yalnızca Query Rewrite açıkken sadece rewrite/time; ikisi kapalıysa sıfır LLM |
| `search(query="...", search_mode="agentic")` | LLM üçlü sınıflandırma, query rewrite ve zaman aralığı çözümlemesi |
| `search(queries=[...], search_mode="agentic")` | Aynı, önceden gelen query bağlam olarak kullanılır |
| Herhangi bir modda açık `scene` | scene önceliklidir, sıfır judge/rewrite LLM |

`search_mode="agentic"` sınırlı query görünümü kullanır; tek veya çoklu query için judge/rewrite çalışır. `search_mode="default"` tek query'de bu LLM kesinlikle çağrılmaz.

Default çoklu-query, `MEMORY_QUERY_REWRITE_MAX_CHARS` üzerindeyse rewrite yapılmaz; Coding açıkken yine sınırlı sahne sınıflandırması yapılır, kapalıyken son query kullanılır.

Coding özelliği `MEMORY_CODING_ENABLED=false` ile kapatılırsa üretkenlik araması güvenli şekilde chat/normal aramaya düşer.

## 3. Extractor giriş penceresi

`working/coding` extractor girdiyi şöyle böler:

- `LAST 5 USER TURNS`: önceki yazılar, yalnızca referans çözümü ve bağlam için kullanılır; tekrar çıkarım yapılmaz.
- `NEW MESSAGES`: bu `add()` çağrısına iletilen tüm mesajlar, son 5 tur kırpılmasına uğramadan çıkarım konusu olur.

Mevcut sıkıştırma kuralları:

- tool result varsayılan olarak 2048 baytla sınırlandırılır; baş ve son tutulur, orta kısım atılır; `MEMORY_CODING_TOOL_RESULT_MAX_BYTES` ile ayarlanır.
- Tek bir assistant metni 6000 karakteri aşarsa varsayılan olarak ilk 3000 ve son 3000 karakter korunur.
- Geçmiş user-turn penceresiyle kırpılır; bir user-turn, o kullanıcı mesajı ve sonraki kullanıcı mesajı öncesi gelen assistant/tool mesajlarını kapsar.
- `MEMORY_EXTRACT_HISTORY_TURNS` kaç tarihsel user-turn okunacağını kontrol eder; varsayılan 5; mevcut extractor final olarak en fazla son 5 user-turn ekler; bu yüzden daha büyük değerler prompt içindeki geçmişi genişletmez.

Örneğin tek bir yazımda 10 user-turn varsa, tüm 10 tur `NEW MESSAGES` içine girer; geçmiş bölümü ise en fazla son 5 user-turn sağlar.

`working/coding` Extractor, deneyim Reconcile, gerçek Reconcile ve çapraz session ilerleme özeti İngilizce ve Çince şablonları içerir. Yönlendirme önce bu yeni girdinin ilk 10 geçerli dil karakterine bakar; Extractor yalnızca bu `NEW user` mesajlara bakar, geçmiş veya tool result dili belirlemez. Çince başlama sonrasında çok kod, yol veya hata mesajı gelirken bile Çince şablon kullanılır; İngilizce başlama sonrası nadiren Çince terimlere rastlanırsa İngilizce şablon korunur.

## 4. Dönüş yapısı

`working/coding` yazımı çift katmanlı sonuç döner:

```python
{
    "success": True,
    "scene": "productivity",
    "request_id": "...",
    "ops": [...],
    "ops_summary": {...},
    "experience_ids": [...],
    "fact_ops": [...],
    "fact_ops_summary": {...},
    "fact_ids": [...],
    "memory_id": "..." or None,
    "elapsed_ms": 123.4,
}
```

`scene="coding"` ayrıca `chain_rounds` döner. Geri uyumluluk nedeniyle bu alan şu anda bu çağrıda tanımlanan diff hunk sayısını sayar; nihai birleştirilmiş benzersiz evrim round sayısına eşit olmayabilir. `0`, kaydedilecek kod değişikliği tanınmadığı anlamına gelir; bu çift katman hafıza yazımının başarısız olduğu anlamına gelmez.

`working/coding` araması şu yapıyı döner:

```python
{
    "request_id": "...",
    "scene": "productivity",
    "facts": [...],
    "experiences": [...],
    "elapsed_ms": 123.4,
}
```

`limit`, facts ve experiences için ayrı ayrı uygulanır; böylece iki katman da en fazla `limit` sonuç dönebilir.

`scene="coding"` ayrıca `chain_block` ve `chain_used` alanları döner (anahtarlar her zaman vardır; varsayılan içerik boştur):

- diff / sürüm evrim bloğunu eklemek için açıkça `MEMORY_CODING_CHAIN_BLOCK_ENABLED=true` ayarlanmalıdır (veya `CodingConfig.chain_block_enabled=True`). Varsayılan kapalıdır, prompt token tasarrufu için opt-in yapılır.
- Açıkken `chain_block`, geri çağrılan fact'lara eşlenen sürüm evrim bağlamıdır; zincir yapılamazsa `""` olur.
- `chain_used`, chain block'a alınmış `facts` indisleridir.
- SDK, `facts` içinden bu öğeleri silmez. Çağıran taraf, "chain block + kalan facts" oluşturmak isterse `chain_used` ile süzgeç uygulamalıdır.

## 5. Depolama ve etki alanı

Varsayılan yerel dosyalar:

```text
<coding db directory>/
├── coding_memory.db          # working deneyim/gerçek metadata
├── coding_memory_cv.db       # coding deneyim/gerçek metadata
├── coding_memory_trace.db    # working/coding ortak geçmiş mesajlar
└── hy_code_chain/
    └── hy_code.db            # coding sürüm evrim zinciri
```

Vektör koleksiyonları:

| Varyant | Deneyim katmanı | Gerçek katmanı |
|---|---|---|
| working | `{base}_coding_keys` | `{base}_working_facts` |
| coding | `{base}_coding_keys_cv` | `{base}_working_facts_cv` |

Aynı client working ve coding arasında geçiş yapabilir; uygulama her varyant için ayrı writer/store önbelleği kullanır.

`workspace_id` ve `branch`, üretkenlik hafızasının etki alanını belirler:

- `workspace_id` verilmezse workspace filtresi uygulanmaz.
- `workspace_id` verildiğinde tam eşleşen workspace seçilir.
- `branch` yalnızca `workspace_id` verildiğinde etkili olur.
- branch filtresi, branch bilgisi kaydedilmemiş genel deneyimlerin yanlış silinmesini önlemek için `branch IS NULL` geçmiş verileri korur.

Yazma ve arama için aynı normalize edilmiş `workspace_id` kullanılması önerilir. Sürüm zinciri `owner`, `user_id` ve `workspace_id` birlikte oluşturulur; böylece farklı kullanıcı veya depo patch'leri birbirine bağlanmaz.

## 6. Kodlama sürüm evrim zinciri

Sürüm evrim zinciri `hy_memory/coding/chain_store.py` içinde uygulanır. LLM veya embedding çağrısı yapmaz; değişikliği aşağıdaki kaynaklardan tanır:

- `MEMORY_CODING_CHAIN_REPO_ROOT` ayarlanmışsa yerel Git çalışma alanından veya değişikliklerden diff okur.
- Araç izlerinden path, old, new içeren belirleyici değişiklik eylemleri tanınır.

Güvenilir şekilde çözülemeyen düzenleme biçimleri atlanır. Zincir katmanı best-effort'tir: kayıt başarısız olsa bile working/coding çift katman hafıza yazımı veya araması engellenmez.

## 7. Yapılandırma

| Değişken | Varsayılan | Etki |
|---|---|---|
| `MEMORY_CODING_ENABLED` | `true` | working/coding pipeline etkin mi |
| `MEMORY_CODING_DB_PATH` | `{DATA_DIR}/data/coding_memory.db` | working temel SQLite yolu; coding/trace yolları buradan türetilir |
| `MEMORY_CODING_TOOL_RESULT_MAX_BYTES` | `2048` | Tek tool result için bayt üst sınırı |
| `MEMORY_CODING_VERBATIM_FALLBACK_ENABLED` | `false` | Model sonucu yoksa ham fact kaydedilsin mi; varsayılan kapalı, asistan/tool izi doğrudan gösterilmesin |
| `MEMORY_CODING_CHAIN_BLOCK_ENABLED` | `false` | search, sürüm evrim zinciri bloğu (`chain_block` / diff) eklesin mi; varsayılan kapalı |
| `MEMORY_EXTRACT_HISTORY_TURNS` | `5` | Geri çağrılan geçmiş user-turn sayısı; mevcut prompt sonunda en fazla 5 kullanılır |
| `MEMORY_CODING_CHAIN_REPO_ROOT` | boş | coding sürüm zinciri için yerel Git diff repo yolu |

`HCM_*` değişkenleri sürüm zinciri için gelişmiş ayarlardır; stabil kamu API yapılandırması değildir. Normal erişimde varsayılanlarına dayanmak önerilmez.
`HCM_BLOCK_OFF=1`, `MEMORY_CODING_CHAIN_BLOCK_ENABLED` açıkken yapılan bloğu tamamen devre dışı bırakmak için kullanılabilir.

## 8. Hedefli doğrulama

```bash
python -m pytest -q \
  tests/coding/test_write_scene_routing.py \
  tests/coding/test_search_scene_routing.py \
  tests/coding/test_scene_coding_variant.py \
  tests/coding/test_scene_coding_search.py \
  tests/coding/test_coding_pipeline_log.py
```

