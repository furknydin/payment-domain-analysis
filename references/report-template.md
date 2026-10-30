# Rapor yapısı

Kapsama göre birleştir veya kısalt; aynı kanıtı tekrarlama. İlgisiz bölümü “yok” diye doldurmak yerine neden uygulanmadığını kısaca belirt.

## 1. Yönetici özeti ve kapsam
- Servisin iş amacı, sahiplendiği kabiliyet ve en önemli bulgular.
- Repo, servis/modül, branch/commit, tarih, sürümler, kullanılan kaynaklar ve erişim sınırları.
- Statik inceleme, incelenen test ve gerçekten çalıştırılan test ayrımı.

## 2. Domain sözlüğü ve sorumluluklar
| Kavram | Kod karşılığı | İş anlamı / invariant | Veri sahibi | Kanıt | Gözlem / çıkarım |
|---|---|---|---|---|---|

Sahiplenilen, orkestre edilen ve devredilen sorumlulukları; kapsam dışında kalan işleri belirt. Bounded context ve diğer servis/provider ilişkilerini context diyagramıyla göster.

## 3. Mevcut DDD mimarisi
| Bileşen / sembol | Mevcut katman / modül | Domain rolü | Sorumluluk | Bağımlılıklar | Kanıt |
|---|---|---|---|---|---|

Aggregate, invariant, transaction sınırı, application/domain service, repository, port/adapter ve event ayrımını açıklayan component diyagramı. DDD kavramının açık model mi yoksa analist yorumu mu olduğunu belirt.

## 4. Girişler, iş akışları ve veri yaşam döngüsü
| Giriş / operation | İş amacı | Request / event modeli | Validasyon / yetki | Use case | DB / event / provider yan etkisi | Response | Kanıt |
|---|---|---|---|---|---|---|---|

Kritik request alanlarının izini analysis-checklist.md tablosuyla göster. Önemli akışların başarı, hata ve asenkron tamamlanma yollarını sequence diyagramında açıkla. State diyagramını geçiş tablosuyla destekle:

| Önceki durum | Tetikleyici | Koşul | Sonraki durum | Yan etki | Geç/tekrar olay davranışı | Kanıt |
|---|---|---|---|---|---|---|

## 5. Servis etkileşimleri ve veri bağımlılıkları

service-interactions.md içindeki senkron çağrı, Kafka ve alan eşleme tablolarını doldur. Envanterde aynı endpoint/topic farklı iş akışlarında farklı amaçla kullanılıyorsa ayrı satırlar kullan.

Her komşu servis için kısa bir profil ver: hangi iş için etkileşiliyor, hangi veri sağlanıyor, hangi veri bekleniyor, bu veri hangi kararı/durum geçişini etkiliyor ve veri gelmezse akış nasıl davranıyor? Karşı servis kimliği doğrulanamıyorsa profil yerine açık bilgi açığı yaz.

Mermaid servis bağımlılık diyagramında sync/async yönlerini, HTTP method/path veya Kafka topic etiketlerini göster. Sequence diyagramında request, response ve asenkron event'i ayrı oklarla belirt. Gecikmiş sonuç, callback ve event üzerinden tamamlanan akışlarda correlation mekanizmasını göster.

Envanter kapsamını özetle: bulunan/izlenen entry point, client call site, producer ve consumer sayıları; çözülemeyen hedefler/topic'ler; incelenmeyen modüller ve kullanılmayan aday tanımlar. Statik tanım ile çalışan runtime topolojisini ayır.

## 6. Idempotency ve tutarlılık
Checklist'teki işlem bazlı matrisi doldur. Sonuç: Doğrulandı / Koşullu / Açık bulundu / Kanıt yetersiz. Her sonuç için garanti kapsamı, süre, harici yan etki, crash-window ve test kanıtını açıkla.

## 7. Domain ve provider araştırması
| Araştırma sorusu | Birincil kaynak / sürüm / erişim tarihi | Kaynak bulgusu | Mevcut kodla ilişkisi | Belirsizlik |
|---|---|---|---|---|

Domain kavramlarını servisin gerçek akışlarıyla anlat. Kodun mevcut davranışı, provider'ın dokümante ettiği davranış ve önerilen yaklaşımı ayrı tut. Üçüncü taraf araştırmasını şirketin iş kuralı diye sunma.

## 8. Bulgular ve sonraki adımlar
| Önem | Bulgu / açık soru | Somut tetikleyici | İş etkisi | Kanıt | Önerilen doğrulama / düzeltme |
|---|---|---|---|---|---|

Önemi kanıtlanan iş etkisine göre sırala; olasılığı bilinmeyen olayı gerçekleşmiş gibi yazma. Erişilemeyen deployment config, ortak kütüphane veya provider sözleşmesini bilgi açığı olarak listele. En sonda domain sahibine sorulacak yalnızca karar değiştirici soruları ekle.
