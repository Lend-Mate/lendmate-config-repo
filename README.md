# HemenKirala Config Repository

Bu depo, LendMate mikroservis ekosisteminin merkezi konfigürasyon deposudur. Spring Cloud Config Server tarafından okunarak API Gateway ve iş servislerine ortam bazlı ayarlar sağlar. Uygulama kaynak kodu veya iş mantığı içermez; servislerin bağlantı, altyapı ve çalışma zamanı ayarlarını tek yerde yönetir.

## İçerik

Depodaki YAML dosyaları servis ve profil adına göre düzenlenir:

| Dosya deseni | Amaç |
| --- | --- |
| `application.yml` | Tüm servislerde ortak ayarlar |
| `application-{profile}.yml` | Belirli bir profil için ortak ayarlar |
| `{service}.yml` | Bir servise ait ortak ayarlar, varsa |
| `{service}-{profile}.yml` | Bir servisin belirli profildeki ayarları |

Mevcut servisler:

- `api-gateway`
- `user-service`
- `product-service`
- `order-service`
- `notification-service`

Mevcut profiller: `dev`, `test`, `stage` ve `prod`.

## Servislerin Rolü

- **API Gateway:** İstemci isteklerini ilgili servise yönlendirir, istek hızını sınırlar ve servislerin Swagger/OpenAPI dokümantasyonunu birleştirir.
- **User Service:** Kimlik doğrulama ve kullanıcı işlemlerini sunar.
- **Product Service:** Ürün, kategori, stok uygunluğu, ürün görselleri, yorumlar, favoriler ve dosya işlemlerini yönetir.
- **Order Service:** Sepet, sipariş ve sipariş kalemi işlemlerini yönetir.
- **Notification Service:** Bildirim ve e-posta işlemlerini yürütür; sipariş olaylarını Kafka üzerinden tüketir.

Servisler arasında PostgreSQL, Kafka, Redis, Elasticsearch, SMTP ve AWS gibi altyapı bağımlılıkları bulunur. Bu bağımlılıkların adresleri ve kimlik bilgileri ilgili profil dosyalarında tanımlanır.

## Ortamlar

- **dev:** Yerel geliştirme için `localhost` üzerindeki bağımlılıkları kullanır.
- **test:** Test çalıştırmaları için ayrı ayarlar ve gerektiğinde H2 gibi test kaynakları kullanır.
- **stage:** Konteyner ağı içindeki servis adları ve PostgreSQL/Kafka gibi ortak ortam bağımlılıklarını kullanır.
- **prod:** Üretim ortamı servis adlarını ve üretim veritabanlarını kullanır.

Spring uygulamaları başlatılırken uygun profil etkinleştirilmelidir. Profil dosyaları, ilgili servis konfigürasyonu ile birlikte yüklenir. Örneğin `user-service` için `dev` profili, `user-service-dev.yml` dosyasındaki ayarları kullanır.

## Dış Bağımlılıklar ve Gizli Değerler

Kimlik bilgileri depoya yazılmaz. Uygulamaları çalıştırmadan önce kullanılan ortamda aşağıdaki değişkenler sağlanmalıdır:

- `DB_USERNAME`, `DB_PASSWORD`
- `JWT_SECRET`
- `MAIL_USERNAME`, `MAIL_PASSWORD`
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`

Değişkenlerin tamamı her serviste kullanılmayabilir. Hangi değişkenin gerekli olduğu, ilgili YAML dosyasındaki `${...}` referanslarından kontrol edilmelidir. Gerçek sırları YAML dosyalarına veya Git geçmişine eklemeyin.

## Gateway Rotaları

Üretim ve stage konfigürasyonlarında API Gateway aşağıdaki yolları yönlendirir:

| Yol | Hedef servis | Varsayılan port |
| --- | --- | ---: |
| `/auth/**`, `/user/**` | User Service | `8081` |
| `/products/**`, `/categories/**`, `/product-availability/**`, `/product-image/**`, `/product-comments/**`, `/favourites/**`, `/files/**` | Product Service | `8080` |
| `/carts/**`, `/orders/**`, `/order-items/**` | Order Service | `8082` |
| `/notifications/**` | Notification Service | `8083` |

Swagger arayüzü `/swagger-ui.html` yolunda sunulur. Servis dokümantasyonları gateway üzerinden `/api/{service}/v3/api-docs` biçiminde tanımlanır.

## Geliştirme Kuralları

1. Ortak ayarları `application.yml` içine, yalnızca belirli bir ortama ait ayarları profil dosyasına ekleyin.
2. Yalnızca bir servisi etkileyen değişikliklerde ilgili `{service}-{profile}.yml` dosyasını güncelleyin.
3. Yeni bir değişken için `${VARIABLE_NAME}` biçimini kullanın ve bu README’deki değişken listesini güncelleyin.
4. Parola, token, erişim anahtarı ve benzeri sırları commit etmeyin.
5. YAML girintisini ve dosya adlandırma düzenini koruyun; değişiklik sonrası hedef servisin ilgili profiliyle başlatma/test kontrolü yapın.

## Güvenlik Notu

Actuator uçları ortak konfigürasyonda geniş biçimde dışa açılmıştır ve gateway güvenliği varsayılan olarak devre dışıdır. Bu ayarlar yalnızca kontrollü geliştirme/test ortamında kullanılmaktadır. Prod deploy pipeline'ında actuator portu/route'u dış ağdan erişime kapatılacak (yalnızca internal network / localhost üzerinden erişilebilir olacaktır).