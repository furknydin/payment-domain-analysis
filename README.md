# Payment Domain Analysis

Java/Spring Boot ödeme servisleri için kanıta dayalı DDD, veri akışı ve entegrasyon analizi yapan AI agent skill'i.

## Kapsam

- Domain sorumlulukları, bounded context, aggregate ve invariant analizi.
- Request alanları, dönüşümler, kalıcılık ve ödeme durum geçişleri.
- Senkron servis çağrıları: hedef servis, HTTP method/endpoint, gönderilen ve beklenen veri.
- Kafka: produce/consume topic'leri, mesaj sözleşmeleri, iş amacı, consumer group ve veri bağımlılıkları.
- Idempotency: eşzamanlılık, transaction, retry, callback, TTL ve harici yan etkiler.
- Resmî provider kaynaklarıyla domain araştırması.
- Kod/satır referansları, Mermaid diyagramları ve belirsizlikleri içeren rapor.

## Kullanım

Skill klasörünü ajanınızın desteklediği skill dizinine yerleştirin. Codex için kullanıcı düzeyindeki konum `~/.codex/skills/payment-domain-analysis/` olabilir.

```text
$payment-domain-analysis payment-service modülünü incele.
Domain sorumluluklarını, request/veri akışlarını, sync/async servis
etkileşimlerini, Kafka topic'lerini ve idempotency garantilerini
kod kanıtlarıyla raporla.
```

Başlangıç noktası: [SKILL.md](SKILL.md). Ayrıntılı analiz ve rapor yönergeleri [references](references/) klasöründedir.

## Çalışma biçimi

Skill analiz ve araştırma içindir; kaynak kodu değiştirmez ve canlı ödeme işlemi yapma yetkisi vermez. Gözlem, çıkarım, bilinmeyen ve önerileri ayırır. Erişilemeyen servis, config veya provider sözleşmesini tahmin etmek yerine bilgi açığı olarak raporlar.

Talimatlar Türkçedir. Çıktı kullanıcının dilinde, varsayılan olarak Türkçe hazırlanır.

## Doğrulama durumu

Dosyalar ve yerel Markdown referansları kontrol edilmiştir. Gerçek bir ödeme servisi üzerinde davranış değerlendirmesi henüz yapılmamıştır. Skill, mimari doğruluk veya uçtan uca idempotency garantisi sağlayan bir doğrulama aracı değildir.
