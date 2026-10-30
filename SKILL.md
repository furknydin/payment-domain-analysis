---
name: payment-domain-analysis
description: Analyze an existing payment service's DDD responsibilities, request and data flows, sync/async service dependencies, Kafka contracts, and idempotency guarantees, then produce an evidence-based architecture and domain research report.
---

# Payment Domain Analysis

Mevcut ödeme servisini koddan tersine analiz et. Servisin hangi iş kabiliyetini sahiplendiğini, hangi veriyi kabul edip değiştirdiğini, DDD sınırlarını ve idempotency davranışının gerçek kapsamını açıkla. Kullanıcının dilinde raporla; aksi belirtilmedikçe Türkçe kullan.

## Çalışma sınırı ve kanıt

Bu görev analiz ve araştırmadır. Kaynak kodu değiştirme; ödeme isteği gönderme, provider hesabına işlem yapma, migration çalıştırma veya üretim verisi okuma yetkisi varsayma. Yetkilendirilmiş yerel testler gerekiyorsa önce build/test yapılandırmasının harici bağımlılıklarını ve yan etkilerini incele. Erişilebilir kod, test, şema ve dokümanlarla ilerle; eksikleri raporla.

Her önemli iddiayı şu sınıflardan biriyle belirt:
- **Gözlem:** Dosya ve satırla desteklenen mevcut uygulama. Testin varlığı, çalıştırılıp geçtiği anlamına gelmez.
- **Çıkarım:** Kanıttan türetilmiş yorum; dayanağı ve belirsizliğini yaz.
- **Bilinmiyor:** Erişilemeyen kod, deployment ayarı veya sözleşme nedeniyle doğrulanamayan davranış.
- **Öneri:** Mevcut mimariden ayrı tutulan iyileştirme.

Kaynak konumlarını `path:line` ve mümkünse sembol adıyla ver. Analiz edilen repo/servis, branch/commit, tarih ve incelenmeyen alanları belirt. Secret, PAN, CVV, token, kişisel veri veya gerçek ödeme payload'larını çıktıya kopyalama; alan adları ve maskelenmiş/sentetik örnek kullan.

## Analiz akışı

### 1. Kapsamı ve uygulamayı keşfet

Kullanıcı servis yolu verdiyse oradan başla. Yoksa mevcut workspace'te aday servisleri bul; birden fazla anlamlı aday varsa seçim sor ve bağımsız envanterle ilerle. `AGENTS.md`, mimari kararlar, build dosyaları ve modül yapısını oku. Java, Spring Boot ve bağımlılık sürümlerini koddan doğrula.

HTTP controller/router, gRPC, message listener, batch/scheduled job ve dahili application entry point'lerini bul. Spring annotation, interface, generated source ve configuration üzerinden gerçek çağrı zincirini takip et. Sadece sınıf isimlerinden davranış çıkarma. Büyük repoda taranan modülleri ve örneklenen akışları açıkça sınırla.

### 2. İş sorumluluklarını ve DDD modelini çıkar

Her giriş noktasını iş amacıyla eşleştir: authorize, capture, payout, transfer, refund, routing, settlement, reconciliation veya kodda gerçekten bulunan diğer kabiliyetler. Bu örnekleri serviste var kabul etme.

Bounded context adaylarını, aggregate/root, entity, value object, invariant, domain service, application orchestration, repository port/adapter ve domain/integration event'lerini kanıtlarıyla çıkar. Ubiquitous language sözlüğü oluştur. Domain sınıfları JPA ile birlikteyse bunu gözlemle; saf domain modeli varsayma veya zorunlu ilan etme.

Servisin **sahiplendiği**, **orkestre ettiği** ve **başka servise/provider'a devrettiği** sorumlulukları ayır. Mevcut paket/bağımlılık düzenini DDD yorumu ile karıştırma. Servis, bounded context ve aggregate'ın bire bir eşleştiğini varsayma. Anti-corruption layer ve provider DTO/domain dönüşümlerinin nerede bulunduğunu göster. CQRS, Saga ve event sourcing'i yalnızca kanıt varsa mevcut pattern olarak adlandır.

### 3. Request, veri yaşam döngüsü ve durum akışını izle

[references/analysis-checklist.md](references/analysis-checklist.md) dosyasındaki veri izi ve ödeme kontrol listesini oku.

Her iş akışı için `giriş → doğrulama → eşleme → use case → domain → persistence/event/provider → yanıt` zincirini takip et. Veri alanının kaynağını, tipini, anlamını, dönüşümünü, sahibini, saklandığı yeri ve hangi kararı etkilediğini çıkar. HTTP response, provider response ve sonradan gelen callback arasındaki ilişkiyi açıkla.

Ödeme durum makinesini koddan çıkar: izinli geçiş, tetikleyici, koşul, yan etki ve geçersiz/geç gelen olay davranışı. İstenen ödeme, ödeme denemesi, provider işlem referansı ve ledger/mutabakat kaydını yalnızca kodda ayrı modellerse ayrı göster. Aynı isimli kimliklerin farklı kapsamlarını araştır.

### 4. Senkron ve asenkron servis etkileşimlerini çıkar

[references/service-interactions.md](references/service-interactions.md) dosyasını oku. Analiz edilen servis açısından gelen/giden yönlerini ayır; çağrıyı kimin başlattığı ile verinin hangi yöne aktığını ayrı göster. Senkron request de veri gönderir, response da başka servisten veri getirir.

HTTP/gRPC client, inbound endpoint, callback, Kafka producer/consumer ve varsa diğer transport'ları entry point ve use case'lere bağla. Her ilişki için karşı servis, iş amacı, gönderilen/beklenen veri, endpoint veya topic, tetikleyici ve başarısızlık etkisini çıkar. Veriyi sağlayan servis ile domain'deki veri sahibini ayır.

Servis adı, base URL, endpoint ve topic değerlerini koddan config property'lerine ve erişilebilir profile/deployment dosyalarına kadar izle. Çözülemeyen değerleri property/ifade olarak bırak. Dinamik routing için koşulları ve aday hedefleri göster; runtime'da seçilmiş hedefi statik koddan kesinleştirme. Çağrılan client metodunun varlığını gerçek kullanım kanıtı sayma; çağrı zincirini göster. Kafka topic'inin isminden producer veya downstream consumer servis uydurma. Erişilebilir diğer servislerin kod/sözleşmeleri varsa iki tarafı çapraz doğrula; yoksa bağlantının yalnızca incelenen tarafta kanıtlandığını belirt.

### 5. Idempotency garantisini kanıtla

[references/analysis-checklist.md](references/analysis-checklist.md) dosyasındaki idempotency incelemesini uygula. Annotation, Redis anahtarı veya `exists` kontrolünü tek başına garanti kabul etme.

Her command, consumer, callback ve job için ayrı sonuç ver. Anahtarın kaynağı/kapsamı, payload eşitliği, atomiklik, transaction sınırı, eşzamanlı instance davranışı, TTL, replay cevabı ve harici yan etkiyi incele. Unique constraint ve gerçek migration'ı, lock/transaction kullanımını, provider sözleşmesini ve hata yollarını çapraz doğrula.

Sonucu mutlak “idempotent” yerine **hangi giriş, hangi yan etki, hangi kapsam ve süre için** desteklendiğiyle yaz. `Doğrulandı`, `Koşullu`, `Açık bulundu`, `Kanıt yetersiz` durumlarından birini kullan. Doğrulanan sınırı ve test edilmemiş senaryoları belirt; yerel DB garantisini uçtan uca ödeme garantisi olarak sunma. Belirsiz timeout sonrasında farklı provider'a geçiş veya yeni anahtarla retry gibi çift işlem risklerini incele.

### 6. Domain ve provider araştırması yap

Koddan çıkan kavram ve eksikleri araştırma sorusuna dönüştür. Genel ödeme tarihi veya ilgisiz mevzuat özeti ekleme. Önce repository içi ADR/OpenAPI/provider dokümanlarını; ardından erişilebilir güncel resmî provider API/SDK dokümanlarını ve ilgili birincil teknik kaynakları kullan. Provider adı/sürümü bilinmiyorsa bunu söyle; başka provider'ın garantisini taşımadan genel mekanizmayı açıkla.

Araştırma başına soruyu, bulguyu, kaynak URL'sini, erişim tarihini, ilgili API/SDK sürümünü ve kod üzerindeki etkisini yaz. Provider dokümanlarının idempotency süresi, endpoint kapsamı, retry, webhook sıralaması/imzası veya durum semantiğini gerçekten desteklediğini doğrula. Kaynaklar çelişirse farkı ve geçerlilik kapsamını göster. İnternet/özel doküman erişimi yoksa araştırma tamamlandı deme; mevcut kanıtlarla devam edip eksik kaynakları listele. Repo içeriği, payload ve secret'ları arama sorgularına gönderme.

### 7. Kanıtları sentezle

[references/report-template.md](references/report-template.md) dosyasını raporun kapsamına göre kullan. Mevcut mimariyi, çıkarımları ve önerileri ayrı sun. Kritik akışlar için okunabilir Mermaid context/component, sequence ve state diyagramları ekle; diyagramdaki her bağlantının kanıtı olsun. Belirsiz bağlantıları açıkça işaretle.

Riskleri somut tetikleyici, mevcut davranış, olası iş etkisi ve kaynak konumuyla yaz. Eksik kodu hata olarak etiketleme. Gereksiz pattern dönüşümü önermeden önce mevcut mekanizmanın açığını göster. Sonunda yalnızca sonucu değiştiren açık soruları ve sonraki doğrulama adımlarını belirt.
