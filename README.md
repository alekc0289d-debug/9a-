# eMaktab Worker, API va Telegram bot

Python backend: eMaktab'dan baholar/jadvalni olib Firebase'ga yozadi,
ro'yxatdan o'tish API'sini (FastAPI) va Telegram botni (aiogram v3) ishga tushiradi.

## O'rnatish

    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt
    cp .env.example .env        # va to'ldiring

Kalit yaratish (EMAKTAB_ENC_KEY):

    python -c "import secrets; print(secrets.token_hex(32))"

API tokeni (API_ADMIN_TOKEN):

    python -c "import secrets; print(secrets.token_urlsafe(32))"

## Ishga tushirish

| Nima | Buyruq |
|------|--------|
| Bitta o'quvchi (test) | `python main.py --test --student-id student_abc123` |
| Navbatni bir marta | `python main.py --once` |
| Soatlik worker + captcha tekshiruvi | `python main.py` |
| Faqat javob berilgan captchalar | `python main.py --captcha` |
| API server | `python api_server.py` (yoki `uvicorn api_server:app`) |
| Telegram bot | `python run_bot.py` |

Eslatma: `api_server.py` ham soatlik avto-sync qiladi. `main.py` bilan bir vaqtda
ishlatsangiz, sync ikki marta bajariladi — bittasini tanlang.

## API

- `POST /api/register`, `GET /api/register/status/{id}` — ochiq (ro'yxatdan o'tish)
- `POST /api/sync-now`, `POST /api/cleanup-orphans` — faqat `X-Admin-Token` bilan

    curl -X POST -H "X-Admin-Token: $API_ADMIN_TOKEN" http://localhost:8000/api/sync-now

## Captcha oqimi

1. Worker captcha ko'rsa, rasm `emaktab_captcha_queue` ga yoziladi.
2. Admin panel -> "Captcha navbati" da javob kiritiladi (`status: answered`).
3. Worker (`main.py`, har 3 daqiqada yoki `--captcha`) saqlangan cookie va token bilan qayta kiradi.
   3 marta xato bo'lsa `failed` bo'ladi.

## Xavfsizlik

`.env`, `credentials.json`, `firebase-service-account.json` hech qachon git'ga yoki
arxivga qo'shilmasin (`.gitignore` da bor).

## Hosting: Render yoki Railway?

Ikkalasi ham to'liq bepul emas — bot uzluksiz ishlashi (polling +
schedulerlar) kerak, bu esa "doim yoniq" server talab qiladi:

| | Railway | Render |
|---|---|---|
| Bepul sinov | $5 kredit, **30 kun** — tugagach to'lov talab qiladi | Web Service bepul, lekin 15 daqiqa harakatsizlikdan keyin "uxlaydi" |
| Doimiy ishlaydigan jarayon (bot) | Hobby $5/oy dan | Background Worker $7/oy dan |
| Narx turi | Foydalanishga qarab (oldindan bilish qiyinroq) | Belgilangan oylik narx (bashorat qilish oson) |

**Tavsiya:** botni (`run_bot.py`, schedulerlar bilan) Render'ning
**Background Worker** xizmatida ishga tushiring (~$7/oy, hech qachon
uxlamaydi, "31 kunlik limit" yo'q). Ro'yxatdan o'tish API'sini
(`api_server.py`) esa Render'ning **bepul Web Service**ida qoldirsa ham
bo'ladi — u faqat ro'yxatdan o'tishda ishlatiladi, uxlab qolsa ham birinchi
so'rov ~30 soniya kechikishi mumkin, xolos.

Tayyor konfiguratsiya: repo tagidagi `render.yaml`. Render dashboardida
**New + → Blueprint** orqali shu repo'ni ulang, so'ng har ikkala xizmat
uchun Environment Variables'ni (`.env.example`dagi kabi) qo'lda kiriting va
`firebase-service-account.json`ni "Secret Files" bo'limiga yuklang.

Railway ham ishlaydi (ayniqsa boshlang'ich sinov uchun qulay), lekin 30
kundan keyin Hobby ($5/oy) rejasiga o'tish shart bo'ladi.
