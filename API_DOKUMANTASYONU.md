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

---

## 4. Şarkı Sözü ve Metin İşlemleri (Lyrics API)

### 4.1. Şarkı Sözü Üretimi (Generate Lyrics)
*   **Açıklama:** Kullanıcının belirlediği tema, dil, kafiye şeması ve hece kısıtlarına göre AI destekli şarkı sözü oluşturur.
*   **Method:** `POST`
*   **URL:** `/ai/generate-lyrics`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "theme": "Yağmurlu bir gece, yalnızlık",
      "language": "tr",
      "rhymeScheme": "ABAB",
      "syllableTarget": 8,
      "sectionType": "verse",
      "mood": "Melancholic"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "content": "Yağmur damlaları camda süzülür\nGeceye karışır sessiz bir ah\nYüreğim boşluğa doğru yürür\nKaybolan hayaller, solgun bir bah",
        "sectionType": "verse",
        "rhymeScheme": "ABAB",
        "syllableCount": 8
      },
      "tokensUsed": 85
    }
    ```

### 4.2. Satır Yeniden Yazma (Rewrite Line)
*   **Açıklama:** Mevcut sözlerden seçili bir satırı bağlamı koruyarak AI ile yeniden yazar (Micro-Rewrite).
*   **Method:** `POST`
*   **URL:** `/ai/rewrite-line`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "fullLyrics": "Yağmur damlaları camda süzülür\nGeceye karışır sessiz bir ah",
      "lineToRewrite": "Geceye karışır sessiz bir ah",
      "instruction": "Daha umutlu bir ton ver"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "rewrittenLine": "Geceye fısıldar yeni bir sabah",
        "originalLine": "Geceye karışır sessiz bir ah"
      },
      "tokensUsed": 40
    }
    ```

### 4.3. Telif Benzerlik Kontrolü (Similarity Check)
*   **Açıklama:** Üretilen sözlerin veya MIDI dizilimlerinin bilinen popüler şarkılarla yapısal benzerliğini vektörel veritabanı üzerinde kontrol eder.
*   **Method:** `POST`
*   **URL:** `/ai/similarity-check`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "type": "lyrics",
      "content": "Yesterday, all my troubles seemed so far away"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "similarityScore": 0.92,
        "isRisky": true,
        "matches": [
          {
            "songTitle": "Yesterday",
            "artist": "The Beatles",
            "similarity": 0.92,
            "matchedSection": "verse"
          }
        ],
        "recommendation": "Bu metin bilinen bir şarkıyla yüksek benzerlik taşıyor. Yeniden yazılması önerilir."
      }
    }
    ```

---

## 5. Proje Yönetimi (Projects API)

### 5.1. Projeleri Listele
*   **Method:** `GET`
*   **URL:** `/projects`
*   **Auth Koruması:** Gerekli
*   **Query Parametreleri:** `?page=1&limit=10&sort=createdAt&order=desc&genre=Synthwave&favorite=true`
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "projects": [
          {
            "_id": "64b5f92a1c2d3e4f5a6b7c8d",
            "title": "Synthwave Demo #1",
            "blueprintInfo": { "genre": "Synthwave", "mood": "Nostalgic", "bpm": 115 },
            "isFavorite": true,
            "isPublic": false,
            "createdAt": "2026-03-01T14:20:00Z"
          }
        ],
        "pagination": {
          "currentPage": 1,
          "totalPages": 5,
          "totalItems": 42,
          "limit": 10
        }
      }
    }
    ```

### 5.2. Proje Detayı
*   **Method:** `GET`
*   **URL:** `/projects/:id`
*   **Auth Koruması:** Gerekli
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "_id": "64b5f92a1c2d3e4f5a6b7c8d",
        "title": "Synthwave Demo #1",
        "blueprintInfo": { "genre": "Synthwave", "mood": "Nostalgic", "bpm": 115, "scale": "C Minor" },
        "chordProgressions": [ { "_id": "...", "chords": ["Cm", "G#", "A#", "Fm"], "key": "C Minor" } ],
        "lyrics": [ { "_id": "...", "sectionType": "verse", "content": "..." } ],
        "audioGenerations": [ { "_id": "...", "urls": { "masterAudio": "https://..." } } ],
        "isFavorite": true,
        "isPublic": false,
        "projectHash": "a1b2c3d4",
        "createdAt": "2026-03-01T14:20:00Z",
        "updatedAt": "2026-03-02T10:15:00Z"
      }
    }
    ```

### 5.3. Yeni Proje Oluştur
*   **Method:** `POST`
*   **URL:** `/projects`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body - application/json):**
    ```json
    {
      "title": "Yeni Lo-Fi Projesi"
    }
    ```
*   **Response (201 Created):**
    ```json
    {
      "status": "success",
      "message": "Proje başarıyla oluşturuldu.",
      "data": {
        "_id": "64b5f92a1c2d3e4f5a6b7c9e",
        "title": "Yeni Lo-Fi Projesi",
        "projectHash": "e5f6g7h8"
      }
    }
    ```

### 5.4. Proje Güncelle
*   **Method:** `PUT`
*   **URL:** `/projects/:id`
*   **Auth Koruması:** Gerekli (Sahip kontrolü)
*   **Request Parametreleri (Body):**
    ```json
    {
      "title": "Lo-Fi Beats Vol. 2",
      "isFavorite": true,
      "isPublic": true
    }
    ```
*   **Response (200 OK):**
    ```json
    { "status": "success", "message": "Proje güncellendi." }
    ```

### 5.5. Proje Sil
*   **Method:** `DELETE`
*   **URL:** `/projects/:id`
*   **Auth Koruması:** Gerekli (Sahip kontrolü)
*   **Response (200 OK):**
    ```json
    { "status": "success", "message": "Proje ve ilişkili tüm veriler silindi." }
    ```

---

## 6. Kullanıcı Profili ve Tercihler

### 6.1. Profil Bilgisi Getir
*   **Method:** `GET`
*   **URL:** `/user/profile`
*   **Auth Koruması:** Gerekli
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "username": "johndoe",
        "email": "johndoe@example.com",
        "role": "user",
        "preferences": { "theme": "dark", "language": "tr" },
        "dailyApiLimits": { "audioRemaining": 3, "textRemaining": 42 },
        "createdAt": "2026-01-15T08:00:00Z"
      }
    }
    ```

### 6.2. Tercihleri Güncelle
*   **Method:** `PUT`
*   **URL:** `/user/preferences`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body):**
    ```json
    {
      "theme": "dark",
      "language": "en"
    }
    ```
*   **Response (200 OK):**
    ```json
    { "status": "success", "message": "Tercihler güncellendi." }
    ```

---

## 7. Dışa Aktarım (Export API)

### 7.1. MIDI Dosyası Oluştur
*   **Açıklama:** Akor zincirlerini MIDI dosyasına dönüştürüp indirme bağlantısı döner.
*   **Method:** `POST`
*   **URL:** `/export/midi`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body):**
    ```json
    {
      "chordSequenceId": "64b5f92a1c2d3e4f5a6b7c8d",
      "bpm": 120
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "downloadUrl": "/downloads/temp/chord_export_a1b2.mid",
      "expiresIn": "5 dakika"
    }
    ```

### 7.2. PDF Rapor Oluştur
*   **Açıklama:** Proje meta verilerini (BPM, enstrümanlar, sözler, mixing notları) profesyonel bir PDF raporuna derler.
*   **Method:** `POST`
*   **URL:** `/export/pdf`
*   **Auth Koruması:** Gerekli
*   **Request Parametreleri (Body):**
    ```json
    {
      "projectId": "64b5f92a1c2d3e4f5a6b7c8d"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "downloadUrl": "/downloads/temp/project_report_a1b2.pdf",
      "expiresIn": "5 dakika"
    }
    ```

### 7.3. Audio-to-MIDI Dönüşüm
*   **Açıklama:** Yüklenen bir ses dosyasını veya mırıldanmayı analiz ederek MIDI notalarına dönüştürür.
*   **Method:** `POST`
*   **URL:** `/tools/audio-to-midi`
*   **Auth Koruması:** Gerekli
*   **Content-Type:** `multipart/form-data`
*   **Request Parametreleri:** `audioFile` (mp3/wav, max 10MB)
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "detectedNotes": [
          { "note": "C4", "startTime": 0.0, "duration": 0.5 },
          { "note": "E4", "startTime": 0.5, "duration": 0.5 },
          { "note": "G4", "startTime": 1.0, "duration": 1.0 }
        ],
        "suggestedChords": ["C", "Am", "F"],
        "detectedBpm": 95,
        "midiDownloadUrl": "/downloads/temp/humming_result.mid"
      }
    }
    ```

---

## 8. Ses Ayrıştırma (Stem Separation)

### 8.1. Stem Ayrıştırma İsteği
*   **Açıklama:** Bir audio track'i vokal, davul, bas ve diğer katmanlara ayırır.
*   **Method:** `POST`
*   **URL:** `/ai/stem-separation`
*   **Auth Koruması:** Gerekli (Premium/Limit kontrolü)
*   **Request Parametreleri (Body):**
    ```json
    {
      "audioTrackId": "64b5f92a1c2d3e4f5a6b7c8d"
    }
    ```
*   **Response (202 Accepted):**
    ```json
    {
      "status": "processing",
      "message": "Ses ayrıştırma işlemi başlatıldı. Tamamlandığında bildirim alacaksınız.",
      "jobId": "job_xyz789",
      "estimatedTime": "30-60 saniye"
    }
    ```
*   **Sonuç Sorgulama:** `GET /ai/stem-separation/status/:jobId`
    ```json
    {
      "status": "completed",
      "stems": {
        "vocalUrl": "https://cdn.aurabeat.com/stems/vocal_xyz.wav",
        "drumsUrl": "https://cdn.aurabeat.com/stems/drums_xyz.wav",
        "bassUrl": "https://cdn.aurabeat.com/stems/bass_xyz.wav",
        "otherUrl": "https://cdn.aurabeat.com/stems/other_xyz.wav"
      }
    }
    ```

---

## 9. Admin Panel İşlemleri

### 9.1. Sistem Metrikleri
*   **Method:** `GET`
*   **URL:** `/admin/metrics`
*   **Auth Koruması:** Gerekli (Sadece `role: "admin"`)
*   **Response (200 OK):**
    ```json
    {
      "status": "success",
      "data": {
        "totalUsers": 1250,
        "activeToday": 87,
        "totalGenerations": 15420,
        "successRate": 0.96,
        "totalTokensSpent": 2450000,
        "topGenres": [
          { "genre": "Lo-Fi", "count": 4500 },
          { "genre": "Synthwave", "count": 3200 }
        ],
        "dailyUsage": [
          { "date": "2026-03-01", "requests": 520 },
          { "date": "2026-03-02", "requests": 480 }
        ]
      }
    }
    ```

### 9.2. Kullanıcı Yönetimi
*   **Method:** `GET` `/admin/users?page=1&limit=20&sort=totalTokensUsed&order=desc`
*   **Auth Koruması:** Gerekli (Admin)
*   **Method:** `PUT` `/admin/users/:id/suspend` — Kullanıcı hesabını askıya al
*   **Method:** `PUT` `/admin/users/:id/limit` — Kullanıcı limitlerini güncelle

---

## 10. Real-Time Collaboration (WebSocket Events)

Ortak çalışma modülü Socket.IO üzerinden aşağıdaki event'leri kullanır:

### Client → Server (Emit)
| Event | Payload | Açıklama |
|---|---|---|
| `join-session` | `{ projectId, inviteCode }` | Bir collaboration oturumuna katılma |
| `leave-session` | `{ projectId }` | Oturumdan ayrılma |
| `chord-update` | `{ projectId, chordData }` | Akor zincirinde değişiklik |
| `lyrics-update` | `{ projectId, lyricBlockId, content }` | Söz bloğunda düzenleme |
| `cursor-move` | `{ projectId, position }` | İmleç pozisyonu güncelleme |
| `save-snapshot` | `{ projectId, label }` | Mevcut durumu kaydetme |

### Server → Client (Broadcast)
| Event | Payload | Açıklama |
|---|---|---|
| `user-joined` | `{ userId, username }` | Yeni kullanıcı katıldı |
| `user-left` | `{ userId }` | Kullanıcı ayrıldı |
| `chord-changed` | `{ chordData, updatedBy }` | Başka biri akor değiştirdi |
| `lyrics-changed` | `{ lyricBlockId, content, updatedBy }` | Başka biri söz düzenledi |
| `cursor-moved` | `{ userId, position }` | Başka birinin imleç konumu |
| `snapshot-saved` | `{ snapshotId, label, createdBy }` | Yeni kayıt noktası oluşturuldu |

---

## 11. Standart Hata Yanıtları (Error Responses)

Tüm endpoint'ler hata durumunda aşağıdaki standart formatta yanıt döner:

```json
{
  "status": "error",
  "code": 400,
  "message": "İnsan-okunabilir hata açıklaması",
  "details": { "field": "email", "issue": "Geçersiz e-posta formatı" }
}
```

### HTTP Durum Kodları

| Kod | Durum | Açıklama |
|---|---|---|
| `400` | Bad Request | Eksik veya geçersiz parametre (validasyon hatası) |
| `401` | Unauthorized | JWT token eksik veya süresi dolmuş |
| `403` | Forbidden | Yetkisiz erişim (ör. admin endpoint'ine user erişimi) |
| `404` | Not Found | İstenen kaynak bulunamadı |
| `409` | Conflict | Çakışma (ör. zaten kayıtlı e-posta) |
| `429` | Too Many Requests | Günlük API limiti aşıldı |
| `500` | Internal Server Error | Sunucu tarafı beklenmeyen hata |

### Rate Limiting Başlıkları (Response Headers)

AI endpoint'lerine (`/ai/*`) yapılan her istekte aşağıdaki header'lar döner:

```
X-RateLimit-Limit: 50          # Günlük toplam izin verilen istek
X-RateLimit-Remaining: 42      # Kalan istek hakkı
X-RateLimit-Reset: 1709510400  # Limitin sıfırlanacağı Unix timestamp
Retry-After: 3600              # (Sadece 429 durumunda) Tekrar deneme süresi (saniye)
```
