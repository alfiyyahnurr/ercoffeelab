# Deployment — Vercel (Monorepo)

Satu repo Git, dipetakan ke beberapa **Vercel Project** terpisah lewat "Root Directory".

## Setup
1. Push seluruh folder `ercoffeelab/` ke satu repo GitHub
2. Di Vercel: "Add New Project" → pilih repo ini → set **Root Directory** = `apps/api` → nama project `ercoffeelab-api`
3. "Add New Project" lagi → repo yang SAMA → **Root Directory** = `apps/admin` → nama project `ercoffeelab-admin`
4. (Nanti) ulangi untuk `apps/mobile` KALAU pakai Expo web build — biasanya mobile di-deploy lewat EAS Build, bukan Vercel

## Environment variables
Diisi terpisah per Vercel Project (Settings → Environment Variables):
- `ercoffeelab-api`: semua isi `apps/api/.env.example`
- `ercoffeelab-admin`: `NEXT_PUBLIC_API_URL` = URL project `ercoffeelab-api` yang sudah live

## Batasi rebuild tidak perlu (opsional)
Karena 2 project baca repo yang sama, push ke `apps/admin` bisa trigger rebuild `apps/api` juga kalau tidak diatur. Di Settings → Git → "Ignored Build Step" tiap project:
```bash
git diff --quiet HEAD^ HEAD -- apps/api
```
(ganti path sesuai project masing-masing — build di-skip kalau tidak ada perubahan di folder itu)

## Update dari SSO callback
`APP_URL` (env di `apps/api`) dan `ADMIN_PANEL_URL` harus diupdate ke domain production setelah pertama kali deploy (bukan lagi `localhost`). Juga update "Authorized redirect URIs" di Google Cloud Console jadi `https://ercoffeelab-api.vercel.app/api/auth/sso/callback`.