# "Audio video fayllarga ishlov berish" tizimi — O'rnatish yo'riqnomasi

## ⚠️ Muhim eslatma (avval o'qing)

Bu paket sizga **to'liq ishlaydigan asosiy tizimni** beradi: server (FastAPI + PostgreSQL),
klient va admin veb-ilovalari, login, fayl yuklash/yuklab olish, tasdiqnomalar,
statistika, korzina, chat va 5 ta mikroxizmat uchun tayyor kod.

Quyidagilarni **o'zingiz bajarishingiz kerak** (chunki bu muhitda internetga ulanish
va Windows'da compile qilish imkoni yo'q edi):
1. Python kutubxonalarini serverga o'rnatish (`pip install -r requirements.txt`)
2. 5 ta AI modelni yuklab olish (quyida havolalar bilan ko'rsatilgan)
3. PostgreSQL'ni Windows Server'ga o'rnatish
4. FFmpeg'ni Windows'ga o'rnatish

Kod tayyor va ishlaydigan holatda — sizga faqat yuqoridagi narsalarni "joyiga qo'yish" qoladi.

---

## 1-QADAM: Server kompyuterini tayyorlash (Windows Server 2016/2019)

### 1.1. Python o'rnatish
- https://www.python.org/downloads/ dan Python 3.10 yoki 3.11 versiyasini yuklab oling
- O'rnatishda "Add Python to PATH" belgisini bosing

### 1.2. PostgreSQL o'rnatish
- https://www.postgresql.org/download/windows/ dan yuklab oling
- O'rnatishda portni **5432** (standart) qoldiring
- O'rnatish jarayonida "postgres" superuser uchun parol so'raladi — eslab qoling
- O'rnatgandan so'ng, "SQL Shell (psql)" dasturini oching va quyidagini bajaring:
  ```
  \i C:\yo'l\server\init_db.sql
  ```
  (bu fayl `avdb` bazasini, `avadmin` foydalanuvchisini va barcha jadvallarni yaratadi)
- **init_db.sql faylidagi parolni** o'zingizniki bilan almashtiring (xavfsizlik uchun)

### 1.3. FFmpeg o'rnatish
- https://www.gyan.dev/ffmpeg/builds/ dan "ffmpeg-release-essentials.zip" yuklab oling
- `C:\ffmpeg` papkasiga chiqaring (ichida `bin\ffmpeg.exe` bo'lishi kerak)
- `C:\ffmpeg\bin` ni Windows PATH ga qo'shing (Sozlamalar → Tizim → Muhit o'zgaruvchilari)

### 1.4. Server kodini joylashtirish
- `server` papkasini, masalan, `C:\AudioVideoServer` ga ko'chiring
- Ichidagi `.env.example` faylini `.env` deb nusxalang va quyidagilarni to'ldiring:
  - `DB_PASSWORD` — PostgreSQL parolingiz
  - `JWT_SECRET_KEY` — istalgan uzun tasodifiy matn
  - `SERVER_PUBLIC_IP=192.168.90.90`
- `run_server.bat` faylini ishga tushiring — u avtomatik ravishda:
  - virtual muhit yaratadi
  - `requirements.txt` dagi kutubxonalarni o'rnatadi
  - serverni `http://192.168.90.90:8000` manzilida ishga tushiradi
- Birinchi ishga tushirishda tizim avtomatik ravishda quyidagi foydalanuvchilarni yaratadi:
  - Admin: login **admin**, parol **123**
  - Klient: login **user**, parol **123**

  **Xavfsizlik uchun ishga tushgach darhol shu parollarni admin panelidan/bazadan o'zgartiring.**

### 1.5. AI modellarni yuklab olish

| # | Xizmat | Kutubxona | Model manbasi |
|---|--------|-----------|----------------|
| 1 | Shovqindan tozalash | DeepFilterNet | `pip install deepfilternet` — model birinchi ishga tushirishda avtomatik yuklanadi (~50MB). Qo'lda: https://github.com/Rikorose/DeepFilterNet/releases |
| 2 | Video → audio ajratish | FFmpeg | Kutubxona emas, dastur (1.3-bandda o'rnatildi) |
| 3 | Speech-to-text | Whisper | `pip install openai-whisper`. Model (`medium`, ~1.5GB) birinchi ishga tushirishda avtomatik yuklanadi. Tezroq variant: https://github.com/ggerganov/whisper.cpp (ggml-medium.bin) |
| 4 | Suhbatdoshlarni ajratish | Pyannote.audio | `pip install pyannote.audio`. Model uchun https://huggingface.co/pyannote/speaker-diarization-3.1 sahifasida ro'yxatdan o'ting, shartlarga rozilik bering, so'ng `huggingface-cli login` orqali token kiriting |
| 5 | Tarjima | NLLB-200 | `pip install transformers sentencepiece torch`. Model: `facebook/nllb-200-distilled-600M` (~2.5GB), birinchi ishlatilganda Hugging Face'dan avtomatik yuklanadi |

> **Eslatma:** Modellar birinchi marta ishlatilganda internetdan yuklab olinadi va
> `.env` faylida ko'rsatilgan papkalarga (`DEEPFILTERNET_MODEL_DIR` va h.k.) saqlanadi.
> Agar serverda internet bo'lmasa, modellarni boshqa (internetli) kompyuterda oldindan
> yuklab, shu papkalarga USB orqali ko'chirishingiz mumkin.

> **GPU haqida:** Whisper, Pyannote va NLLB GPU (NVIDIA CUDA) bo'lsa ancha tezroq ishlaydi.
> GPU bo'lmasa ham ishlайdi, lekin sekinroq. Katta fayllar uchun GPU serverini tavsiya qilamiz.

---

## 2-QADAM: Klient kompyuterlariga o'rnatish

1. `client` papkasini klient kompyuteriga USB yoki tarmoq orqali ko'chiring
2. `client\install\install_client.bat` faylini administrator huquqida ishga tushiring
   - Bu avtomatik ravishda `C:\Audio video denoising\` papkasini va ichidagi
     barcha kerakli papkalarni (kelgan fayllar, yuborilgan fayllar, korzina, media va h.k.) yaratadi
   - Dastur fayllarini `%LOCALAPPDATA%\AudioVideoKlient` ga nusxalaydi
   - Ish stolida yorliq yaratadi
3. `media` papkasiga o'zingizning 3 ta `.wav` ovoz faylingizni joylashtiring
   (masalan: notify1.wav, notify2.wav, notify3.wav)
4. Ish stolidagi "Audio Video Klient" yorlig'ini bosib kiring
5. Login: **user**, Parol: **123** (yoki admin tomonidan yaratilgan boshqa login)

**Server manzilini o'zgartirish kerak bo'lsa:** `client\js\app.js` faylining birinchi
qatoridagi `API_BASE = "http://192.168.90.90:8000"` qiymatini o'zgartiring.

---

## 3-QADAM: Admin kompyuteriga o'rnatish

1. `admin` papkasini admin kompyuteriga ko'chiring
2. `admin\install\install_admin.bat` faylini ishga tushiring
3. Ish stolidagi "Audio Video Admin" yorlig'i orqali kiring
4. Login: **admin**, Parol: **123**

---

## Tizim arxitekturasi (qisqacha)

```
[Klient veb-ilova]  --HTTP/WebSocket-->  [Server: FastAPI, 192.168.90.90:8000]  <--> [PostgreSQL :5432]
[Admin veb-ilova]   --HTTP/WebSocket-->                    |
                                                             v
                                          [5 ta mikroxizmat: DeepFilterNet,
                                           FFmpeg, Whisper, Pyannote, NLLB]
```

- Klient/admin — oddiy HTML/CSS/JS veb-ilovalar (brauzerda ochiladi, `file://` orqali ham ishlaydi)
- Server — barcha so'rovlarni qabul qiladi, fayllarni saqlaydi, mikroxizmatlarni
  navbat asosida (ThreadPoolExecutor) ishga tushiradi, natijani bazaga yozadi
- Bir vaqtning o'zida bir nechta faylga ishlov berish — `pipeline.py` dagi
  `ThreadPoolExecutor(max_workers=4)` orqali (serveringiz quvvatiga qarab sonini oshiring/kamaytiring)

---

## Papka tuzilishi

```
audio_video_system/
├── server/                  → Windows Server'ga o'rnatiladi
│   ├── app/
│   │   ├── main.py          → dastur kirish nuqtasi
│   │   ├── config.py        → sozlamalar (.env dan o'qiydi)
│   │   ├── models.py        → PostgreSQL jadval modellari
│   │   ├── routers/         → API endpointlar (auth, files, chat, admin...)
│   │   └── microservices/   → 5 ta AI xizmat + orkestrator (pipeline.py)
│   ├── init_db.sql          → PostgreSQL sxema
│   ├── requirements.txt
│   ├── .env.example
│   └── run_server.bat       → serverni ishga tushirish
├── client/                  → har bir klient kompyuteriga o'rnatiladi
│   ├── login.html, index.html
│   ├── js/app.js            → barcha menyular logikasi
│   ├── lang/                → uz.json, ru.json, en.json
│   └── install/install_client.bat
└── admin/                   → admin kompyuteriga o'rnatiladi
    ├── login.html, index.html
    ├── js/app.js
    └── install/install_admin.bat
```

---

## Keyingi rivojlantirish bosqichlari (tavsiya)

Ushbu skelet tizimning **ishlaydigan yadrosi** — barcha asosiy funksiyalar (login,
fayl almashish, chat, statistika, korzina, admin boshqaruvi) real API chaqiruvlari
bilan ishlaydi. Lekin quyidagi joylarni ehtiyojingizga qarab kengaytirish tavsiya etiladi:

1. **Mikroxizmatlar sinovi** — har birini alohida test fayl bilan sinab ko'ring
   (`server/app/microservices/` ichidagi funksiyalarni to'g'ridan-to'g'ri chaqirib)
2. **Xavfsizlik** — HTTPS sertifikat qo'shish (hozir oddiy HTTP, faqat lokal tarmoq uchun mos)
3. **Katta fayllar** — juda katta video fayllar uchun yuklashni bo'lib-bo'lib (chunked upload) qilish
4. **Navbat tizimi** — juda ko'p parallel so'rov bo'lsa, ThreadPoolExecutor o'rniga
   Celery + Redis navbatiga o'tish tavsiya etiladi
5. **Xato boshqaruvi** — foydalanuvchiga aniqroq xato xabarlarini ko'rsatish

Men bilan qadamma-qadam davom etishga tayyorman — masalan, keyingi safar
mikroxizmatlarni real fayl bilan sinab ko'rish, yoki chat/statistika bo'limini
yanada boyitish ustida ishlashimiz mumkin.
