# Servisler arası iletişim ve veri bağımlılıkları

## Keşif ve hedef çözümleme

- REST/HTTP: Feign, RestClient, RestTemplate, WebClient, HTTP Interface, generated OpenAPI client ve ortak SDK wrapper'larını gerçek call site'larına kadar izle. gRPC için stub, service/method ve proto sözleşmesini takip et. Kullanılmayan bean/client tanımlarını aktif çağrılardan ayrı tut.
- Inbound controller/router, webhook ve RPC handler'larını çıkar. Endpoint'in çağıranını yalnızca gateway route, erişilebilir caller kodu veya sözleşme destekliyorsa isimlendir; endpoint'in varlığı caller servisi kanıtlamaz.
- Class/method path birleşimi, context-path, gateway rewrite, base URL, service discovery adı, path/query parametreleri ve API versiyonunu ayrı çözümle. Farklı profile değerlerini aynı runtime adresi gibi birleştirme. Header isimlerini ve işlevini göster; credential değerlerini gösterme.
- Kafka: `@KafkaListener`, `KafkaTemplate`, `StreamBridge`, Spring Cloud Stream bindings, Kafka Streams topology, wrapper publisher, outbox relay ve CDC connector config'lerini ara. Listener pattern subscription, SpEL/property, dinamik destination, retry topic ve DLT/DLQ ilişkilerini çözümle. Outbox'a yazan uygulama ile broker'a yayınlayan relay/connector'ı ayrı göster.
- `application*.yml/properties`, config import, Helm/Kubernetes, environment placeholder ve erişilebilir IaC üzerinden değer kaynağını izle. Secret veya canlı altyapı erişimi gerektiren değerleri bilgi açığı olarak bırak. Bağımlılık ağacındaki bir SDK'yı tek başına aktif servis bağımlılığı sayma.

## Senkron çağrı envanteri

Yönü analiz edilen servis açısından yaz. HTTP dışı transport'ta endpoint alanına RPC tam adını veya mevcut adresleme biçimini koy.

| Yön / transport | Çağıran → hedef servis | Tetikleyici / use case | Method + path / RPC | Base URL / discovery / property / profile | Gönderilen alanlar ve model | Beklenen response alanları ve model | İş amacı / verinin kullanıldığı karar | Timeout / retry / hata etkisi | Call site + config + sözleşme kanıtı |
|---|---|---|---|---|---|---|---|---|---|

Her çağrıda gerekli/opsiyonel response alanları, fallback/cache, senkron bekleme, sonuç belirsizliği ve veri gelmediğinde payment state davranışını araştır. HTTP kullanılıyor diye tüm iş akışını senkron sayma: 202, polling veya callback ile tamamlanma olabilir. Reactive/non-blocking client kullanımı tek başına business-level async entegrasyon değildir.

## Kafka produce / consume envanteri

Produce ve consume için ayrı satır kullan. Bir topic'te birden çok event tipi veya amaç varsa ayrıştır.

| Yön | Topic adı / pattern / çözülemeyen ifade | İş amacı / command-event türü | Producer servis / yayın mekanizması | Consumer servis / handler / group | Payload tipi / schema / version | Key / partition / correlation | Tetikleyici veya beklenen veri | İşlem / DB / üretilen çıktı | Ack / retry / DLT / ordering | Kanıt ve belirsizlik |
|---|---|---|---|---|---|---|---|---|---|---|

- Produce: Ne zaman, hangi iş gerçeği veya komut için yayınlanıyor? Model hangi alanları kimden alıyor? Publish sonucu bekleniyor mu; hata hangi akışı etkiliyor? Downstream consumer bilinmiyorsa “bilinmiyor” yaz.
- Consume: Hangi iş bilgisini bekliyor, kim üretiyor (kanıt varsa), hangi alanları okuyor, hangi invariant/state/DB kaydını etkiliyor, başka endpoint/topic'e ne aktarıyor? Consumer group servis kimliği değildir.
- Topic oluşturma config'ini aktif publish/consume kanıtından ayır. Kafka Streams join/window/aggregation, repartition/changelog ve CDC topic'lerini business event topic'leriyle karıştırma; teknik topic'lerin rolünü ayrıca açıkla.
- DB/outbox commit, publish acknowledgment, offset commit ve yan etki sınırlarını mevcut idempotency matrisiyle ilişkilendir. Broker producer idempotence/transaction ayarı ile ödeme etkisinin uçtan uca idempotency'sini eşitleme.

## Servisler arası alan eşleme ve veri sahipliği

| Kaynak servis / endpoint / topic | Alınan alan / anlam | İç model / dönüşüm | Kullanıldığı kural / state / kayıt | Hedef servis / endpoint / topic | Gönderilen alan / anlam | Domain veri sahibi | Kanıt |
|---|---|---|---|---|---|---|---|

Kritik kimlik, amount/currency, provider reference, status/error, tenant/merchant ve idempotency/correlation alanlarını uçtan uca izle. Servisin veriyi üretmesi, değiştirmesi, zenginleştirmesi veya yalnızca taşıması ayrımını yap. Alan isimleri aynı olsa da anlam eşitliğini varsayma. Bir downstream servis yalnızca event'in bir kısmını okuyorsa, kaynak erişimi varsa gerçekten okunan alanları göster; yoksa yalnızca yayınlanan sözleşmeyi raporla.

## Kapsam ve güvenilirlik

Envanteri bulunan tüm gerçek call site ve binding'lerle karşılaştır. Aynı wrapper altında gizlenen farklı endpoint'leri veya profile ile kapatılmış listener'ları atlama. Büyük kapsamda örnekleme gerekiyorsa kullanıcıya hangi alanların eksik kaldığını söyle. Config'te tanımlı, kodda referanslı ve runtime'da doğrulanmış durumlarını ayır; yalnızca statik analiz yapıldıysa üretimde aktif topoloji iddiasında bulunma.
