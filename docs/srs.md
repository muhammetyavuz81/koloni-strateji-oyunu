# Koloni / Strateji Oyunu - Yazılım Gereksinim Belgesi (SRS)

## 1. Proje Özeti
Bu proje, web tabanlı, eşzamanlı olmayan (asynchronous) bir Orta Çağ strateji oyunudur. Oyuncular bir köy yönetir, zamanla kaynak (odun, taş, altın) üretir, binalar inşa eder, asker eğitir ve diğer oyunculara saldırılar düzenler. Oyunun temel mühendislik problemi, arka planda çalışan bir betik (cron job) olmadan, tüm üretim ve bekleme sürelerinin zaman damgası (timestamp) farklarıyla hesaplanmasıdır.

## 2. Kullanıcı Rolleri
- **Oyuncu (Player):** Kayıt olup giriş yapan, köyünü geliştiren ve savaşan temel kullanıcı.
- **Yönetici (Admin):** Tüm oyuncuları, köyleri ve raporları görüntüleyebilen üst yetkili.

## 3. Kullanıcı Hikayeleri (User Stories) ve Backlog

### Aşama 1: Temel Kaynak Motoru
- **US-01:** Bir oyuncu olarak oyuna kayıt olabilmeli ve giriş yapabilmeliyim.
- **US-02:** Bir oyuncu olarak köy sayfamı açtığımda, en son girişimden bu yana geçen süreye göre kaynaklarımın (odun, taş, altın) arttığını görebilmeliyim.
- **US-03:** Bir oyuncu olarak sayfam açıkken kaynaklarımın periyodik olarak (AJAX ile) güncellendiğini görebilmeliyim.

### Aşama 2: İnşaat Sistemi
- **US-04:** Bir oyuncu olarak kaynaklarım yeterliyse bir binanın (Örn: Oduncu) seviyesini yükseltme emri verebilmeliyim.
- **US-05:** Bir oyuncu olarak kaynaklarım yetersizse, sistem inşaat emrimi reddetmeli ve hata mesajı göstermelidir.
- **US-06:** Bir oyuncu olarak inşaat emri verdiğimde, işlemin kuyruğa alındığını ve kalan süreyi görebilmeliyim.
- **US-07:** Bir oyuncu olarak inşaat süresi bittiğinde, binamın seviyesinin ve saatlik üretimimin arttığını görebilmeliyim.

### Aşama 3: Askeri Sistem ve Savaş
- **US-08:** Bir oyuncu olarak Kışla binasından kaynak harcayarak asker (Piyade, Süvari) eğitebilmeliyim.
- **US-09:** Bir oyuncu olarak diğer oyuncuların listesini görebilmeli ve birine saldırı emri verebilmeliyim.
- **US-10:** Bir oyuncu olarak saldırı süresi dolduğunda, savaşın sonucunu (kayıplar, kazanılan ganimet) bir rapor olarak görebilmeliyim.

## 4. Kullanım Senaryosu (Use-Case) Diyagramı Metni
*(Not: Sunum için bu metni draw.io veya benzeri bir araçla görselleştirmeliyiz)*
- **Aktör:** Oyuncu
- **Senaryolar:** 
  - Kayıt Ol / Giriş Yap
  - Kaynak Durumunu Görüntüle
  - Bina Yükselt
  - Asker Eğit
  - Rakibe Saldır
  - Savaş Raporu Oku
- **Includes (Kapsamalar):** 
  - "Bina Yükselt" senaryosu `<<include>>` "Kaynak Kontrolü"
  - "Rakibe Saldır" senaryosu `<<include>>` "Savaş Formülü Hesaplaması"

## 5. Çekirdek Mühendislik Kararları
- **Zaman Yönetimi:** PHP'nin `DateTime` sınıfı ve test edilebilirlik için özel bir `Clock` arayüzü kullanılacaktır.
- **Savaş Formülü:** *Strategy* tasarım deseni ile soyutlanacaktır.
- **Veri Katmanı:** *Repository* deseni ile soyutlanıp, Aşama 2'de JSON, Aşama 3'te MySQL + PDO olarak uygulanacaktır. Eşzamanlılık (yarış durumu) koşullu `UPDATE` sorguları ile çözülecektir.
6. UML Sınıf Diyagramı (Taslak)
Aşağıdaki diyagram, sistemin çekirdek kaynak hesaplama ve mimari tasarımını göstermektedir.

```mermaid
classDiagram
    class ClockInterface {
        <<interface>>
        +now() DateTimeImmutable
    }
    class SystemClock {
        +now() DateTimeImmutable
    }
    ClockInterface <|.. SystemClock

    class Village {
        +int id
        +int user_id
        +float wood
        +float stone
        +float gold
        +DateTimeImmutable last_update
    }

    class ResourceCalculatorService {
        -ClockInterface clock
        -array buildingConfigs
        +calculateCurrentResources(Village village) Village
        -calculateProductionSince(DateTimeImmutable lastUpdate, float baseProduction) float
    }

    class VillageRepositoryInterface {
        <<interface>>
        +find(int id) Village
        +save(Village village) bool
    }

    ResourceCalculatorService ..> ClockInterface : uses
    ResourceCalculatorService ..> Village : updates