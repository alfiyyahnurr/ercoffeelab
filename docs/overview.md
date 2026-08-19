# ERCoffeeLab — Overview untuk Antigravity AI

Monorepo ini berisi 3 project yang berbagi 1 backend & 1 database. **Baca urutan ini sebelum mulai kerja di project manapun:**

Tiap project (`apps/api`, `apps/admin`, `apps/mobile`) punya folder `docs/` dengan 6 file yang sama polanya:
- `prd.md` — requirement/fitur
- `architecture.md` — stack, struktur folder, pola kode
- `erd.md` — data/entity yang relevan (skema lengkap ada di `apps/api/docs/erd.md`, yang lain cuma ringkasan konsumsi)
- `navigation-flow.md` — alur (halaman untuk admin/mobile, sequence request untuk api)
- `design-system.md` — visual/UI (admin & mobile) atau konvensi API (backend)
- `todo.md` — checklist actionable, urutan pengerjaan

**Urutan baca:**
1. `apps/api/docs/prd.md` — requirement lengkap sistem (semua 21 poin awal)
2. `apps/api/docs/erd.md` — WAJIB paham sebelum nulis query manapun
3. `apps/api/docs/architecture.md` — pola kode backend
4. `apps/api/docs/todo.md` — endpoint mana yang sudah/belum ada (ini kontrak yang dipakai admin & mobile)
5. Lanjut ke `apps/admin/docs/*` kalau kerja di admin, atau `apps/mobile/docs/*` kalau kerja di mobile

## Urutan pengerjaan yang disarankan
Backend (`apps/api`) HARUS lebih dulu dari admin/mobile untuk tiap fitur.
1. Endpoint di `apps/api` (sesuai `apps/api/docs/todo.md`)
2. Test manual pakai curl
3. Baru integrasikan ke `apps/admin` dan/atau `apps/mobile`

## Status sudah dikerjakan (jangan diulang)
- ✅ Fase 1: Schema Neon lengkap + seed dummy (`apps/api/src/db/`)
- ✅ Fase 2: Auth OTP (customer) + SSO Google (staff) + **login email+password (staff, akun demo)** + JWT + role middleware

## Referensi desain (WAJIB dipakai, jangan desain dari nol)
```
apps/admin/_reference/ERCoffeeLabAdmin.tsx   ← ikuti persis untuk admin panel
apps/mobile/_reference/ERCoffeeLab.tsx       ← ikuti persis untuk mobile app
```
Ringkasan token (warna, font, komponen) ada di `apps/admin/docs/design-system.md` dan `apps/mobile/docs/design-system.md` — tapi file `_reference/*.tsx` itu sendiri adalah sumber kebenaran paling detail (struktur komponen, layout persis).

## Login admin panel — DUA jalur (penting)
Admin panel BUKAN cuma SSO. Ada juga login email+password untuk akun demo (bukan email asli):
```
admin@ercoffeelab.com / demo1234           (super_admin)
bandung.admin@ercoffeelab.com / demo1234   (outlet_admin)
```
Detail teknis: `apps/api/docs/architecture.md` § Auth flow, `apps/admin/docs/prd.md` § Auth.

## Belum dikerjakan
- Semua endpoint di `apps/api/docs/todo.md` yang bertanda ⬜
- Admin panel — dibuat ULANG pakai Next.js (project Vite lama di-deprecate, lihat `apps/admin/docs/prd.md`)
- Mobile app — dibuat baru pakai Expo (mockup Vite lama cuma referensi UI)
- Integrasi Midtrans & Fontee

## Aturan lintas project
- Satu sumber kebenaran kontrak API = `apps/api/docs/todo.md` (status endpoint) + shape response didokumentasikan lewat contoh di `erd.md` masing-masing consumer.
- Jangan bikin cara auth baru di admin/mobile — selalu lewat endpoint `/api/auth/*` yang sudah ada (OTP, SSO, atau staff/login).
- Field nama di response API pakai `camelCase` — lihat `apps/api/docs/design-system.md`.

Detail deploy ke Vercel (monorepo, root directory per app): `docs/deployment.md`