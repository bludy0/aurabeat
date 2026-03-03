# AuraBeat - REST API DOKÜMANTASYONU (OpenAPI Formatı)

**Versiyon:** 1.0.0
**Proje Adı:** AuraBeat - Yapay Zeka Destekli Dijital Müzik Asistanı
**Ders:** YMH354 Web Tasarım ve Programlama Final Projesi
**Base URL:** `https://api.aurabeat.com/v1`
*(Not: Geliştirme ortamında (localhost:5000) çalıştırılabilir)*

---

## 1. Authentication (Kimlik Doğrulama) İşlemleri

Bu bölüm, kullanıcıların sisteme kayıt olması, giriş yapması ve oturum açma token'ı (JWT) oluşturmasını sağlayan endpoint'leri içerir.

### 1.1. Kullanıcı Kaydı (Register)
*   **Açıklama:** Sisteme yeni bir kullanıcı kaydeder ve veri tabanına (MongoDB) ekler. Şifreler hashlenerek saklanır.
*   **Method:** `POST`
*   **URL:** `/auth/register`
*   **Auth Koruması:** Yok (Açık Endpoint)
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "username": "johndoe",
      "email": "johndoe@example.com",
      "password": "StrongPassword123"
    }
    ```
*   **Response (201 Created):**
    ```json
    {
      "status": "success",
      "message": "Kullanıcı başarıyla oluşturuldu.",
      "data": {
        "userId": "64b5f92a1c2d3e4f5a6b7c8d",
        "username": "johndoe"
      }
    }
    ```
*   **Frontend Kullanımı:** Kayıt sayfasında (Register Page) kullanıcının girdiği verileri `fetch()` veya `axios` ile bu endpoint'e gönderir. Hatalı şifre uzunluğu vb. durumlarda dönen 400 (Bad Request) koduyla Form Validasyonu sağlanır.

### 1.2. Kullanıcı Girişi (Login)
*   **Açıklama:** Kayıtlı kullanıcının e-posta ve şifresini doğrulayarak bir **JSON Web Token (JWT)** döner.
*   **Method:** `POST`
*   **URL:** `/auth/login`
*   **Auth Koruması:** Yok
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "email": "johndoe@example.com",
      "password": "StrongPassword123"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "message": "Giriş başarılı.",
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "user": {
         "id": "64b5f92a1c2d3e4f5a6b7c8d",
         "role": "user"
      }
    }
    ```
*   **Frontend Kullanımı:** Dönen token, uygulamanın durum yönetiminde (Context API/Redux) ve tarayıcının `localStorage`/`sessionStorage` alanında saklanır. Diğer yetki gerektiren API isteklerinde `Authorization: Bearer <token>` Header'ına eklenir.

---

## 2. Jeneratif Yapay Zeka (AI) İşlemleri

Bu bölüm, LLM servisleri ile sunucumuz arasında köprü kurarak, yetkilendirilmiş kullanıcılara üretim çıktısı sağlayan endpoint'leri listeler.

### 2.1. İskelet Üretimi (Generate Blueprint)
*   **Açıklama:** Kullanıcının seçtiği müzik türü ve duyguya göre BPM, Gam, Çalgı paleti ve mix ipuçları içeren detaylı bir prodüksiyon planı oluşturur. Sunucu, bu parametreleri AI servisine (Gemini/OpenAI) yapılandırılmış bir prompt olarak gönderir.
*   **Method:** `POST`
*   **URL:** `/ai/generate-blueprint`
*   **Auth Koruması:** Gerekli (JWT Token Bekler)
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "genre": "Synthwave",
      "mood": "Nostalgic, Energetic"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "bpm": "110-120",
        "scale": "C Minor",
        "instruments": ["Analog Bass Synth", "LinnDrum", "Pad Synthesizer"],
        "chordProgression": ["Cm", "G#", "A#", "Fm"],
        "mixingTips": ["Bass sentezleyiciye hafif chorus efekti uygulayın.", "Kick frekansında 60Hz bölgesini boost edin."]
      },
      "tokensUsed": 150
    }
    ```
*   **Frontend Kullanımı:** AI özellikleri modülünde "Üret" butonuna basıldığında tetiklenir. Dönen yanıt, state içerisine kaydedilip ekrandaki interaktif kartlarda (BPM, Akorlar, Enstrümanlar olarak) ayrı alanlarda veri görselleştirmesine yansıtılır.

### 2.2. Sıradaki Akor Tahmini (Predict Next Chord)
*   **Açıklama:** Kullanıcı tarafından halihazırda oluşturulmuş bir akor zincirini dizide alır ve LLM'e (Büyük Dil Modelleri) sorarak, duygu durumuna en uygun **3 alternatif akor önerisi (ve kısa açıklamasını)** talep eder.
*   **Method:** `POST`
*   **URL:** `/ai/predict-chord`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "currentProgression": ["Am", "F", "C"],
      "mood": "Dark/Sad"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "predictions": [
        { "chord": "G", "reason": "Basit, popüler bir sonlama sağlar." },
        { "chord": "E7", "reason": "Minor bir çözünürlük hissi katar, gerilimi yükseltir." },
        { "chord": "Dm", "reason": "Daha karamsar ve aşağıya inen bir geçiş için." }
      ]
    }
    ```

### 2.3. Doğrudan Ses (Audio) İstemi (Generate Audio)
*   **Açıklama:** Sunucu üzerinden Suno, MusicGen veya Udio gibi bir sese dönüştürme (audio-generation) API'sine istek atar ve oluşturulan gerçek `mp3/wav` dosyasının stream (akış) bağlantısını geri döner.
*   **Method:** `POST`
*   **URL:** `/ai/generate-audio`
*   **Auth Koruması:** Gerekli (Premium/Limit Kontrolü çalıştırılır)
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "prompt": "Nostalgic Synthwave track with a driving mid-tempo beat, C Minor scale, atmospheric pads."
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "audioUrl": "https://cdn.aurabeat.com/audio/8a7b6c5d.mp3",
      "duration": 120
    }
    ```

---

## 3. Kullanıcı Veri ve Dashboard (Dashboard API) İşlemleri

Bu API bölümü, kullanıcının kişisel profili, token limitleri (rate limit) ve üretim geçmişini frontend'e taşımakla yükümlüdür.

### 3.1. Kullanıcı Dashboard Verilerini Getir
*   **Açıklama:** Oturum açan kullanıcının dashboard istatistiklerini (**Toplam üretim sayısı, favori sayısı, kullanılan AI token sayısı**) veri tabanından tek bir JSON paketi halinde aggregate ederek (toplayarak) döner.
*   **Method:** `GET`
*   **URL:** `/user/dashboard`
*   **Auth Koruması:** Gerekli (Bearer Token'dan User ID elde edilir)
*   **Request Parametreleri:** Yok (User ID, header'daki JWT içinden Parselenir).
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "dashboardStats": {
        "totalGenerations": 42,
        "totalTokensSpent": 15400,
        "favoriteCount": 5,
        "genreDistribution": {
          "Synthwave": 20,
           "Lo-Fi": 15,
           "Rock": 7
        },
        "recentActivity": [
          {"type": "blueprint", "date": "2026-03-02T14:20:00Z"},
          {"type": "audio", "date": "2026-03-01T10:15:00Z"}
        ]
      }
    }
    ```
*   **Frontend Kullanımı:** Dashboard sayfasına (Örn: `Dashboard.js` component'ine) React `useEffect` hook'u ile yükleme esnasında çağrılır. Gelen veriler Chart.js (veya Recharts) kullanılarak Pasta (Pie) ve Çizgi (Line) grafikleri çizdirmek için doğrudan state'e beslenir.

---

### *Güvenlik ve Validasyon Notları*
* Tüm endpoint'lerde Express.js ara katmanı (Middleware) kullanılarak şema (Schema) kontrolü (Mongoose Joi/Zod) yapılacaktır.
* Yapay zeka maliyetlerini önlemek amacıyla kullanıcının günde maksimum kaç kez `/ai/` endpoint'ine erişebileceğini sınırlayan (Rate Limiting) mekanizma kurulmuştur. Limit aşımında API, HTTP 429 (Too Many Requests) hatası dönecektir.
