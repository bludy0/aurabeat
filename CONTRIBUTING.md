# Katkı Sağlama Rehberi (Contributing)

Öncelikle AuraBeat projesine zaman ayırmak ve geliştirmelere destek olmak istediğiniz için teşekkür ederiz! Açık kaynak topluluğunun gücüne inanıyoruz. Projeye kod yazarak, hata (bug) bildirerek veya dokümantasyonu geliştirerek katkı sağlayabilirsiniz.

Aşağıdaki rehber, katkı süreçlerinin temiz ve uyumlu ilerlemesi için hazırlanmıştır.

## 1. Hata Bildirimi (Bug Reports)

Beklenmedik bir hatayla karşılaşırsanız GitHub Issues üzerinden yeni bir bildirim açabilirsiniz. Lütfen şu detayları belirtin:

*   Hatanın net ve açıklayıcı bir başlığı.
*   Hatayı **nasıl tekrar edebileceğimize** (steps to reproduce) dair adımlar.
*   Klişe bir ifade olacak ama "Benim bilgisayarımda çalışıyor" dememek için: Tarayıcınız, İşletim Sisteminiz ve Node.js versiyonunuz.
*   Varsa konsoldan veya terminalden hata logları (Ekran görüntüsü de eklenebilir).

## 2. Geliştirme Süreci (Pull Request Açmak)

Aktif koda katkı sağlamak istiyorsanız şu adımları takip edin:

### 2.1. Branch (Dallanma) Stratejisi
Her zaman `main` veya `master` dalından değil, amaca uygun isimlendirilmiş kendi yan dalınızdan (branch) çalışın:
*   Yeni bir özellik ekliyorsanız: `feature/ozellik-adi`
*   Bir hatayı çözüyorsanız: `bugfix/hata-adi` VEYA `hotfix/hata-adi`
*   Dokümantasyon güncelliyorsanız: `docs/duzeltme-adi`

### 2.2. Geliştirme Standartları
*   Codebase **ESLint** kurallarına tabidir. Lütfen commit atmadan önce `npm run lint` veya `npm run lint:fix` komutunu çalıştırın.
*   Fonksiyonları, karmaşıklığı artırmayacak düzeyde tutmaya çalışın. Uzun ve karmaşık fonksiyonlarda açıklayıcı (`// yorum satırları`) bırakın.
*   Eklediğiniz yeni modüller için (örneğin Audio ayrıştırma veya yeni bir analiz), API dokümantasyonuna (`GrupAdıAPI.md`) gerekli endpoint bilgilerini de ekleyin.

### 2.3. Commit Mesajları
Semantic Commit Messages standartlarına uymanız PR süreçlerini çok daha hızlı geçmenizi sağlar:
*   `feat: Akor sistemi eklendi` (Yeni özellik)
*   `fix: Token sayacı eksiye düşme hatası` (Hata düzeltimi)
*   `docs: README güncellendi` (Sadece doküman)
*   `style: Header CSS düzeltmeleri` (Tasarım ve format)
*   `refactor: Auth arayüzü yeniden yazıldı` (Sistemsel iyileştirme, kod temizliği)

### 2.4. Pull Request (PR) Şablonu
PR açtığınızda formu olabildiğince doldurun. Reviewer (İnceleyen) ekibimiz kodunuza bakmadan önce bu özellik/hata çözümünün neyi hedeflediğini bilmelidir. Gerekiyorsa "Önce / Sonra" (Before/After) UI ekran görüntüleri yükleyin.

## 3. Topluluk ve Davranış Kuralları

*   Tüm yorumlarda ve tartışmalarda profesyonel ve saygılı olun.
*   Yıkıcı değil, yapıcı geri bildirimler (code review) verin.
*   Sorular için doğrudan bir ekip üyesine özelden yazmak yerine Issue veya Discussions tabını kullanın, bilginin paylaşılabilir olmasını sağlayın.

🚀 Desteğiniz ve harika kodlarınız için şimdiden teşekkürler!
