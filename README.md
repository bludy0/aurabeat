# 🎵 AuraBeat - AI Müzik Asistanı

> Yapay zeka destekli dijital müzik üretim platformu — Akor tahmini, söz yazımı, ses üretimi ve daha fazlası.

AuraBeat, müzisyenler, besteciler ve oyun geliştiriciler için tasarlanmış yapay zeka destekli kapsamlı bir dijital müzik üretim asistanıdır. Yaratıcı tıkanıklığı (writer's block) aşmak amacıyla, akor zinciri tahmini, şarkı sözü oluşturma, Audio-to-MIDI yetenekleri ve yapay zeka tabanlı gerçek ses üretimini (Prompt-to-Audio) modern bir web arayüzünde birleştirir.

## Özellikler

*   🤖 **AI Müzik İskeleti:** Tür ve Duygu bağlamında detaylı altyapı (Blueprint), BPM, ve Mixing tavsiyeleri oluşturulması.
*   🎵 **Prompt-to-Audio:** Yazı tabanlı veya UI metrikli promptlardan Suno/Udio entegrasyonuyla anında mp3/wav üretimi.
*   🛠️ **Stem Separation & MIDI:** Üretilen sesi katmanlarına ayırma (vokal, davul vb.) ve parçaları saniyeler içinde MIDI'ye dönüştürme.
*   🤝 **Real-time Collaboration:** WebSockets ile aynı projede birden fazla kişinin senkron olarak çalışması (Söz yazımı/Akorlar).
*   🎹 **Toolbox:** Metronom, Tap Tempo, Dijital Piyano Roll ve Akor kütüphanesi içeren gelişmiş stüdyo araç seti.
*   🛡️ **Telife Karşı Similarity Checker:** Üretilen parçaların veya yazılan sözlerin, LLM vektörleri üzerinden bilinen popüler şarkılarla benzerliğini analiz etme.

## Başlangıç (Gereksinimler)

Projeyi yerel ortamınızda (localhost) çalıştırmak için aşağıdaki araçların bilgisayarınızda kurulu olması gerekmektedir:

*   [Node.js](https://nodejs.org/) (v18.x veya daha yeni)
*   [MongoDB](https://www.mongodb.com/) (Yerel veya Atlas bağlantısı)
*   [Git](https://git-scm.com/)

Bu sistemde çalışacak yapay zeka servisleri için şu platformlardan API Key edinmelisiniz:
*   OpenAI API Key (veya Google Gemini)
*   Suno API Key (Desteklenen unofficial/official wrapper)

## Kurulum (Local Development)

**1. Repoyu Klonlayın**
```bash
git clone https://github.com/aurabeat-team/aurabeat-web.git
cd aurabeat-web
```

**2. Bağımlılıkları Yükleyin**
Projeyi hem Frontend (Client) hem de Backend (Server) olarak iki katmanlı kurmanız gerekir.
```bash
# Backend kurulumu
cd backend
npm install

# Frontend kurulumu
cd ../frontend
npm install
```

**3. Çevre Değişkenlerini Ayarlayın (.env)**
Ana dizindeki veya `backend` klasöründeki `.env.example` dosyasını çoğaltarak `.env` olarak yeniden adlandırın ve kendi verilerinizi ekleyin.
```bash
cp .env.example .env
```
*(Bkz. [Örnek Çevre Değişkenleri](#))*

**4. Sunucuları Başlatın**
```bash
# Terminal 1 - Backend
cd backend
npm run dev

# Terminal 2 - Frontend
cd frontend
npm start
```
Frontend sunucusu genellikle `http://localhost:3000`'de, API işlemleri ise `http://localhost:5000`'de çalışacaktır.

## Katkıda Bulunma (Contributing)

Bu proje açık kaynak geliştirmeye uygundur. Katkıda bulunmak için lütfen `CONTRIBUTING.md` dosyamıza göz atın. Temel adımlar:
1. Projeyi forklayın.
2. Feature branch'inizi oluşturun (`git checkout -b feature/MuthisYenilik`).
3. Değişikliklerinizi commit'leyin (`git commit -m 'feat: Yeni AI motoru eklendi'`).
4. Branch'e push'layın (`git push origin feature/MuthisYenilik`).
5. Pull Request (PR) açın.

## Lisans

Bu proje **MIT Lisansı** altında lisanslanmıştır. Detaylar için `LICENSE` dosyasına bakabilirsiniz. 

## İletişim & Proje Sahibi

*   **Proje Ekibi:** AuraBeat Team
*   **İletişim:** hello@aurabeat.com
*   **Demo:** [https://aurabeat.vercel.app](https://aurabeat.vercel.app)
