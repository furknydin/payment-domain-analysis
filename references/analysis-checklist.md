# Ödeme servisi analizi: ayrıntılı kontroller

## Veri izi

Her entry point ve kritik alan grubu için aşağıdakileri incele. Alan sayısı fazlaysa domain açısından kritik alanları ayrıntılandır, kalan grupların kapsamını belirt.

| Alan / model | Kaynak | Tip / anlam | Validasyon ve dönüşüm | Kullanıldığı karar | Yazıldığı / iletildiği yer | Veri sahibi | Kanıt |
|---|---|---|---|---|---|---|---|

- Kimlikler: tenant/merchant, request, business payment, attempt, provider reference, event/delivery, correlation ve idempotency kimliklerinin kapsamı ve birbirine dönüşümü.
- Para: amount tipi, currency, major/minor unit, decimal scale, rounding, kur kaynağı ve zamanı, fee ve net/gross ayrımı. Floating-point veya tutar dönüşümünü ancak kod kanıtıyla bulguya dönüştür.
- Provider mapping: iç ve dış durum/hata kodları; bilinmeyen yeni status, eksik alan, nullable alan ve default davranışı. Teknik hata ile iş reddi ve sonucu belirsiz işlem ayrılıyor mu?
- Kalıcılık: tablo/collection, PK/unique/index, audit/version, optimistic/pessimistic locking, soft delete, retention/TTL, outbox/inbox ve cache. Entity annotation ile migration uyuşmazlığı varsa belirt; hangi şemanın deployment'ta aktif olduğu bilinmiyorsa garanti çıkarma.
- Yetki bağlamı: request içindeki merchant/tenant kimliği ile doğrulanmış identity eşleşmesi; idempotency ve sorguların bu kapsama bağlılığı.
- Mesajlar: producer, topic/queue, partition/key, ordering sınırı, schema/version, consumer group, ack ve transaction ilişkisi, retry/DLQ/redrive. Broker garantisini business-effect garantisiyle karıştırma.
- Provider çağrısı: endpoint, auth/signature, API/SDK sürümü, timeout, retry sınıflandırması, rate limit, circuit breaker, callback/status-query ve mutabakat yolu.
- Hassas veri: alan kategorileri, log/trace/error yolları ve maskeleme. Koddan gözlenmeyen uyumluluk sertifikası veya mevzuat uygunluğu iddia etme.

## Idempotency incelemesi

Her giriş noktası için ayrı kayıt oluştur:

| İşlem / giriş | Korunan yan etki | Key ve kapsam | Payload çakışması | Atomik mekanizma | Saklama / TTL | Replay sonucu | Provider garantisi | Kanıt / test | Sonuç |
|---|---|---|---|---|---|---|---|---|---|

Şunları değerlendir:

1. Anahtar kullanıcıdan mı, iş kimliğinden mi, provider'dan mı türetiliyor? Retry boyunca aynı mı? Farklı tenant/operation/provider/version kapsamları ayrılıyor mu?
2. Aynı anahtar farklı amount/currency/beneficiary gibi içerikle gelirse ne oluyor? Request fingerprint canonicalization, null/default alan ve sıralama davranışı belirli mi?
3. Check-then-act yarışına karşı DB unique constraint, koşullu update, lock veya atomik cache işlemi var mı? Aynı anda iki node geldiğinde korunan şey yalnızca kayıt mı, gerçek yan etki mi?
4. Key reservation, iş kaydı, outbox ve response kaydı hangi transaction içinde? Lock/lease süresi dolunca eski worker devam edebiliyor mu? DB rollback harici ödeme çağrısını geri almış sayılıyor mu?
5. In-progress, succeeded, failed ve outcome-unknown durumlarında replay nasıl cevaplanıyor? Önceki HTTP body/status mu dönüyor, yoksa güncel resource mu okunuyor? Yanıt yeniden oluşturuluyorsa semantiğini belirt.
6. TTL veya kayıt temizliği sonrası aynı istek yeniden işleniyor mu? Redis erişilemezse fail-open mı fail-closed mu? Davranışı kod/config ile kanıtla.
7. Provider kendi idempotency mekanizmasını hangi endpoint, süre ve kapsamda sağlıyor? İç key dış çağrıya nasıl taşınıyor? Failover ve credential/account farkı key kapsamını değiştiriyor mu?
8. Callback deduplication event ID'ye mi delivery ID'ye mi bağlı? Aynı business transition farklı event'lerle gelirse ne oluyor? Raw-body signature doğrulaması ve replay penceresi provider sözleşmesine uygun mu?
9. Consumer ack/commit kaybı, redelivery, DLQ'dan geri alma, scheduled job çakışması ve manuel retry aynı iş etkisini tekrar üretebiliyor mu?

## Arıza pencereleri

Her kritik ödeme akışında aşağıdaki zaman noktaları için mevcut davranışı ve varsa testini ara:
- Key ayrıldıktan sonra iş kaydı oluşmadan çökme.
- Provider isteği kabul ettikten sonra response alınamaması.
- Provider sonucu alındıktan sonra DB commit'inden önce çökme.
- DB commit'inden sonra event publish veya HTTP response öncesinde çökme.
- Callback'in senkron yanıttan önce gelmesi; tekrar veya ters sırada gelmesi.
- İki eşzamanlı aynı istek; aynı key farklı payload; key expiry sonrası tekrar.
- Outbox publish sonrası işaretleme öncesi çökme ve consumer effect sonrası ack kaybı.
- Kısmi capture/refund veya çok adımlı transferin yalnızca bir adımının tamamlanması; yalnızca servis bu işlemleri destekliyorsa.

Bunlar test tasarım sorularıdır, tüm servislerde bulunduğu varsayılan hatalar değildir. Çalıştırılmayan senaryoları “önerilen test” olarak işaretle. “Exactly once” ifadesini ancak bileşen, işlem ve hata sınırı açıkça kanıtlanıyorsa kullan.
