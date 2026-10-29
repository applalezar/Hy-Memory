# Çok Konuşmacılı Diyalog Desteği

Hy Memory, tek bir diyalog içindeki mevcut kullanıcı, diğer isimli gerçek kişiler ve AI asistanı ayırt etmeyi destekler. Bu yetenek standart `messages` girdisindeki `role` ve `name` alanlarını kullanır; yeni `participant` tablosu veya ayrı Memory[] alanı eklemez.

## 1. Giriş sözleşmesi

Çağıran taraf, "mevcut kullanıcı" bakış açısını belirler:

- `role="user"` mevcut hafızanın ait olduğu kullanıcıyı belirtir; `name` ile kullanıcı adı verilebilir.
- `role="assistant"` ve `name` varsa başka isimli gerçek kişiyi temsil eder.
- `role="assistant"` ve `name` yoksa AI asistanı temsil eder.
- `role="tool"` içindeki `name`, konuşmacı adı değil araç adıdır.

Örnek olarak Tim mevcut kullanıcı, John diyalogdaki başka bir gerçek kişi olsun:

```python
from hy_memory import HyMemoryClient

client = HyMemoryClient(mode="pro")
result = client.add(
    [
        {"role": "user", "name": "Tim", "content": "I work at Stripe."},
        {"role": "assistant", "name": "John", "content": "I am a nurse in Berlin."},
    ],
    user_id="tim",
)
```

OpenAI tarzı string mesajlar ve Anthropic tarzı content blocks, `tool` dışı rollerde `name` alanını korur. OpenAI'nın ayrı `tool` mesajı yine `name` alanını araç adı olarak çözer.

## 2. İçsel render

Normal user-AI diyalog orijinal formatında kalır:

```text
[user]: I like coffee.
[assistant]: Noted.
```

İsimli çok konuşmacılı diyalog şu şekilde render edilir:

```text
The user's name is Tim
[user]: I work at Stripe.
[John]: I am a nurse in Berlin.
```

Türkçe mesajlar için karşılığı:

```text
Kullanıcının adı Tim
[user]: Stripe'da çalışıyorum.
[John]: Ben Berlin'de hemşireyim.
```

| Girdi | Render etiketi | Anlam |
|---|---|---|
| `role=user` | `[user]` | Mevcut kullanıcı; `name` olmasa da isim değil |
| `role=assistant, name=John` | `[John]` | Diğer isimli gerçek kişi |
| `role=assistant`, `name` yok | `[assistant]` | AI asistanı |
| `role=system/tool` | Orijinal role | Sistem veya araç mesajı |

Mevcut kullanıcının adı tek başına metnin başına yazılır; tüm cümle boyunca yalnızca bir kez görünür. İsim satırının dili, mesaj içeriğine göre otomatik olarak belirlenir.

## 3. Hafıza ataması

Çıkarım zinciri aşağıdaki kurallarla gerçekler üretir:

- Mevcut kullanıcının gerçeği konu olarak `The user ...` / `Kullanıcı ...` başlar, `owner="user"` olur.
- Diğer isimli gerçek kişinin gerçeği isimle başlar; örneğin `John is a nurse in Berlin.`, `owner="user"` olur.
- AI asistan tarafından sağlanan veya üretilen içerik, asistan başlığıyla konu edilir, `owner="agent"` olur.

İsimli gerçek kişiler, giriş protokolünde `role="assistant"` olarak görünseler bile `owner="agent"` olmazlar.

L0 basic profile yalnızca mevcut kullanıcıyı tanımlar. `role=user` içindeki `name`, `basic_info.name` için güvenilir girdi olur; diğer isimli gerçek kişilerin meslek, konum gibi bilgileri normal gerçeklere girer, mevcut kullanıcının profile'ına değil.

Reconcile prompt, farklı özniteliklere sahip gerçekleri birbirinin üzerine `UPDATE` veya `SUPERSEDE` etmeyi açıkça yasaklar. Örneğin `The user likes coffee` ile `John likes coffee` ayrı iki gerçek olarak kalır.

## 4. Geçmiş bağlamı

L1_RAW saklanan içerik yukarıdaki render formatını kullanır. Geçmiş geri çağrılırken:

- Metnin başındaki isim satırı tanınır ve kaldırılır; tek bir kullanıcı mesajı gibi yanlış yorumlanmaz.
- `[John]:` gibi isimli etiketler korunur.
- Çok satırlı mesajlar konuşmacı önekiyle ayrılır; metin devam satırı yeni mesaj gibi yanlış bölünmez.
- Geçmişteki asistan ve diğer isimli gerçek kişiler, tarihsel bağlam üst sınırı nedeniyle kırpılır; kullanıcı mesajları tam olarak korunur.

## 5. Mevcut sınırlar

- Çağıran taraf, mevcut kullanıcı perspektifini `role` ile belirtmek zorundadır; SDK isme göre kim olduğunu kestirmez.
- Şu anda isimli AI agent için ayrı kimlik modeli yoktur. `role=assistant + name` her zaman isimli gerçek kişi olarak yorumlanır.
- Konuşmacı öznesi esas olarak gerçek metin içinde kodlanır; ayrı arama filtre alanı değildir.
- Diğer isimli gerçek kişiler için ayrı L0 profile oluşturulmaz. Birden fazla kişinin ayrı profilini tutmak istiyorsanız farklı `user_id`'lerle ayrı bakış açıları yazmalısınız.

## 6. Uygulama ve doğrulama

Ana uygulama konumları:

- `hy_memory/pipelines/base.py`: `ChatMessage.name`, konuşmacı etiketi ve diyalog render.
- `hy_memory/client.py`: OpenAI/Anthropic mesaj normalizasyonu.
- `hy_memory/agent/extractor.py` ve `hy_memory/agent/prompts_zh.py`: konu, owner ve basic profile kuralları.
- `hy_memory/agent/reconciler.py`: çapraz öznitelik birleştirme kısıtları.
- `hy_memory/pipelines/writer.py`: L1_RAW geçmiş ayrıştırma.

Hedef testler:

```bash
python -m pytest -q \
  tests/test_multi_speaker_format.py \
  tests/test_reconcile_fallback.py
```

