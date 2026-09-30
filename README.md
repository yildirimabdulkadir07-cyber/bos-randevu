# SonŞans — 1.0

Son dakika hizmet fırsatları için müşteri ve işletme hesapları, süreye bağlı fiyat düşüşü ve tek kapasitelik rezervasyon uygulaması. Ödeme işletmede yapılır; kart bilgisi veya çevrim içi ödeme toplanmaz.

## Çalıştırma

Node.js 24+:

```bash
npm ci
npm start
```

http://localhost:3000 adresinde açılır. Yerel kayıtlar `data/sonsans.sqlite` dosyasında tutulur. `.env.example` dosyasını `.env` olarak kopyaladıysan `node --env-file=.env server.js` kullan. Normal başlangıç örnek müşteri/işletme oluşturmaz.

Tanıtım verileriyle ayrı bir veritabanında:

```bash
SEED_DEMO=true SQLITE_PATH=data/demo.sqlite npm start
```

Örnek işletme: `isletme@sonsans.example` / `SonSansDemo2026!`. Sadece tanıtımda kullan. Örneklerin ekranında tanıtım etiketi görünür.

## Tamamlanan akışlar

- Ayrı müşteri / işletme kayıt ve giriş ekranları; profil düzenleme, şifre değiştirme, çıkış.
- HttpOnly oturum, scrypt şifre özeti, CSRF / kaynak kontrolü ve giriş deneme sınırı.
- Arama, kategori / şehir seçimi, yakınlık sıralaması (konum izniyle), favoriler.
- Geri sayım ve lineer fiyat düşüşü; taban fiyat işletme tarafından belirlenir.
- Atomik rezervasyon, tek aktif rezervasyon, fiyatın rezervasyon anında sabitlenmesi, benzersiz kod.
- Müşteri iptali boş saati tekrar açar. İşletme iptali fırsatı geri çeker.
- İşletme hizmet yönetimi, fırsat yayınlama / geri çekme, gelen randevu listesi.
- Randevu saati geldikten sonra tamamlandı / işletmede ödendi / gelmedi işlemleri.
- Yalnızca tamamlanan randevuda bir değerlendirme; gerçek puanlardan işletme ortalaması.
- Uygulama içi bildirimler, yol tarifi / paylaşma, kişisel veri indirme ve hesap anonimleştirme.
- Mobil uyumlu PWA, özel ikon ve çevrim dışı kabuk. API/kişisel veriler önbelleğe alınmaz.
- Yönetici bootstrap ortam değişkenleriyle yapılır; kullanıcı kayıt formu yönetici rolü veremez.

## Üretim

`NODE_ENV=production`, `DATABASE_URL` ve gerçek `PUBLIC_URL` gerekir. Yönetilen PostgreSQL kullanılmalıdır; kalıcı disk açıkça ayarlanmadıkça SQLite ile üretimde başlangıç reddedilir. Veritabanı tabloları `ss_` önekiyle kurulur. Mevcut Render komutu `node server-pg.js`, Boş Randevu 2 tablolarını tek işlemde aktarır. Kaynak tabloları değiştirmez; ayrıca `ss_legacy_snapshot` içinde kayıt kopyası tutar. Aktarım sayıları doğrulanır; uyumsuz veri veya dolu hedef tablolar işlemi geri alır ve sunucu başlamaz. Eski oturumlar aktarılmaz; kullanıcılar mevcut şifreleriyle yeniden giriş yapar. Eski sabit fiyatlar korunur; eski demo ödeme işaretleri gerçek ödeme sayılmaz, asıl değerler snapshot içinde kalır. Bu kopya aynı veritabanındadır; bağımsız felaket kurtarma yedeği değildir. Eski sürüme geri dönüşten önce yeni sürümde oluşan kayıtlar ayrıca korunmalıdır.

`PG_SSL=true` sertifika doğrulamasıyla TLS açar. Render dahili veritabanı bağlantısı için servisle veritabanını aynı bölgede tut. Ortamın gerektirdiği TLS ayarını doğrula; `PG_SSL_VERIFY=false` varsayılan değildir.

Render için repo içindeki `render.yaml` hazırlık dosyasıdır. Ücretsiz plan, deneme içindir: Render ücretsiz PostgreSQL 30 gün sonra sona erer. Kalıcı ticari kullanım için planı kullanıcı seçmelidir. Kaynak: https://render.com/docs/free

Şifre yenileme için `RESEND_API_KEY`, doğrulanmış `MAIL_FROM` ve `PUBLIC_URL` ayarla. Sağlayıcı ayarlı değilse uygulama şifre yenileme e-postası gönderdiğini iddia etmez. Şifre değiştirme her zaman kullanılabilir.

Kaynak paket GitHub `sonsans-v1-source` dalına yüklendi. Render çalışma alanı ve canlı güncelleme kullanıcı tarafından onaylandı. Bu paket mevcut `unzip -o bos-randevu-production-v2.zip && npm ci` / `node server-pg.js` komutlarıyla uyumludur. Node 24 gerekir. Yerel API ve arayüz testleri geçti. PostgreSQL sürücüsü, atomik rezervasyon ve aktarım testleri yerel PGlite PostgreSQL motoru üzerinden de geçti; bu ortam yönetilen Render PostgreSQL kümesiyle aynı değildir. Canlı dağıtımın başarı durumu paket içinden garanti edilmez; yayın sonrası API ve servis logları doğrulanmalıdır.

## Testler

```bash
npm run check
npm test
```

Testler gerçek API üzerinden müşteri / işletme kaydı, sahiplik, iki eşzamanlı rezervasyon, iptal / yeniden rezervasyon, fiyat sabitleme, süre dolması, tamamlanma / yorum, oturum ve CSRF, şifre değişimi, veri kalıcılığı ve hesap silme akışlarını doğrular.

## iOS / Android

Önce doğrulanmış HTTPS yayın URL'sini ayarla:

```bash
SONSANS_URL=https://YOUR-VERIFIED-HOST node scripts/configure-native.mjs
npm run mobile:sync
```

Depoda yerel platform klasörleri yoksa önce `npx cap add ios` ve `npx cap add android` ile oluştur. Mevcut `com.bosrandevu.app` uygulama kimliği korunur; görünen ad SonŞans’tır. iOS / Android projeleri web uygulamasını HTTPS sunucudan açacak şekilde hazırlanabilir. Apple imzalama, Xcode / bulut macOS derlemesi ve mağaza gönderimi ayrı dış işlemlerdir; bu pakette imzalı IPA/APK bulunmaz. App Store inceleme sonucu garanti edilmez.

## Yayın öncesi tamamlanacak dış bilgiler

- Canlı veri aktarımı logları, HTTPS ve cihaz doğrulaması; bağımsız veritabanı yedeği ve kalıcı işletim planı.
- Destek e-postası, işletmeci / veri sorumlusu bilgileri, saklama süreleri ve yayınlanacak gizlilik metni. Arayüzdeki metin genel taslaktır; kurum kimliği verilmeden tamamlanmış hukuki metin sayılmaz.
- Şifre yenileme e-postası için doğrulanmış alan adı ve sağlayıcı anahtarı.
- Apple imzalama / App Store Connect yükleme ve inceleme.

SonŞans kaynak kodu tamamlanan çekirdek akışları kapsar; dış servis bağlantıları ve mağaza yayını doğrulanana kadar ürünün tamamen yayında olduğu iddia edilmez.
