# AGENTS.md — ERCoffeeLab Monorepo

Instruksi operasional untuk AI coding agent (Antigravity) yang mengerjakan repo ini. Baca file ini PERTAMA KALI sebelum menyentuh kode apapun.

## 1. Apa proyek ini
Sistem coffee shop ERCoffeeLab: backend tunggal + 2 aplikasi (admin panel, mobile app) yang berbagi database yang sama. Sedang migrasi total dari Supabase ke Neon Postgres dengan arsitektur custom (raw SQL, JWT auth sendiri, tanpa RLS).

```
apps/
  api/     Backend — Next.js API Routes + Neon Postgres + raw SQL query layer
  admin/   Panel admin — Next.js (App Router), role super_admin/outlet_admin
  mobile/  App pelanggan — React Native + Expo
```

## 2. Urutan baca dokumen (WAJIB, jangan skip)
1. **File ini (AGENTS.md)** — aturan umum lintas project
2. `docs/overview.md` — peta lengkap semua dokumen + referensi desain
3. `apps/api/docs/prd.md` → `erd.md` → `architecture.md` → `todo.md` — sebelum sentuh backend
4. `apps/admin/docs/*` — sebelum sentuh admin panel (baca 6 file: prd, architecture, erd, navigation-flow, design-system, todo)
5. `apps/mobile/docs/*` — sebelum sentuh mobile app (6 file sama)

Tiap `apps/*/docs/` punya struktur sama: `prd.md`, `architecture.md`, `erd.md`, `navigation-flow.md`, `design-system.md`, `todo.md`. `todo.md` adalah checklist actionable — kerjakan urut dari atas ke bawah, centang yang sudah selesai & sudah dites.

## 3. Urutan pengerjaan (cross-project)
**Backend dulu, selalu.** Admin & mobile adalah consumer API — jangan bangun UI yang manggil endpoint yang belum ada.
```
1. Cek apps/api/docs/todo.md → endpoint yang dibutuhkan sudah ✅ atau masih ⬜?
2. Kalau ⬜: buat endpoint di apps/api dulu, test pakai curl, centang di todo.md
3. Baru kerjakan UI admin/mobile yang konsumsi endpoint itu
```

## 4. Aturan teknis non-negotiable

### Backend (`apps/api`)
- Query database pakai **raw SQL** via `sql` dari `src/db/client.ts` (`@neondatabase/serverless`). **JANGAN** pakai Drizzle query builder di route handler — Drizzle cuma untuk `db:push` (migration) dan `seed.ts`.
- Auth selalu lewat `requireCustomer()` / `requireStaff()` dari `lib/auth-middleware.ts`. Jangan bikin cara cek token baru.
- **outlet_admin WAJIB difilter `outlet_id`** di setiap query yang scoped-outlet (pengganti RLS Supabase yang sudah tidak ada). Lupa filter ini = celah keamanan.
- Response sukses: object langsung (single) atau `{ data: [...] }` (list). Error: `{ error: "pesan" }` + HTTP status yang tepat.
- Field response **camelCase**, walau kolom database `snake_case` — transform manual di route handler.
- Uang: integer rupiah tanpa desimal. Tanggal: ISO 8601 string.
- Semua panggilan ke Midtrans/Fonnte WAJIB dicatat ke `payment_logs`/`notification_logs` (request + response).

- **Hitung ulang harga di server** saat `POST /api/orders` — jangan pernah percaya harga dari client.
- Detail lengkap: `apps/api/docs/architecture.md` + `design-system.md` (konvensi API).

### Admin panel (`apps/admin`)
- **Project Vite lama sudah DEPRECATED** — dibangun ulang dari nol pakai Next.js App Router. Jangan coba convert project lama.
- Dua jalur login: SSO Google DAN email+password (akun demo). Keduanya hasilkan JWT shape sama.
- Desain **WAJIB** ikuti `apps/admin/_reference/ERCoffeeLabAdmin.tsx` sepersis mungkin (warna, layout sidebar, drawer, komponen). Token warna & font ada di `apps/admin/docs/design-system.md`.

### Mobile app (`apps/mobile`)
- Belum di-scaffold — mulai dari nol pakai Expo + Expo Router.
- Desain **WAJIB** ikuti `apps/mobile/_reference/ERCoffeeLab.tsx` sepersis mungkin. Artifact itu React web (Tailwind + lucide-react) — konversi ke React Native pakai **NativeWind** (lihat catatan konversi di `apps/mobile/docs/architecture.md`), JANGAN tulis ulang semua style manual ke StyleSheet dari nol.
- `localhost` tidak bisa diakses dari HP fisik lewat Expo Go — pakai IP address laptop di `EXPO_PUBLIC_API_URL`.

## 5. Akun demo & testing

**Login admin panel:**
```
admin@ercoffeelab.com / demo1234           (super_admin)
bandung.admin@ercoffeelab.com / demo1234   (outlet_admin)
```

**Payment (Midtrans) — dua cara test:**
```bash
# A. Simulasi lokal, tanpa perlu akun Midtrans sama sekali (dev-only):
curl -X POST http://localhost:3000/api/payments/midtrans/simulate \
  -H "Content-Type: application/json" -d '{"orderId":"<id>","result":"success"}'

# B. Sandbox Midtrans beneran — lihat apps/api/docs/navigation-flow.md § 6
```

## 6. Status saat ini (jangan kerjakan ulang)
- ✅ Schema Neon lengkap + seed (`apps/api/src/db/`)
- ✅ Auth: OTP customer, SSO Google staff, email+password staff, JWT, role middleware
- ✅ Payment: Midtrans charge + webhook + simulate endpoint, loyalty auto-calc, notification stub
- ⬜ Semua endpoint lain di `apps/api/docs/todo.md` (outlet, menu, order, voucher, dst)
- ⬜ Admin panel Next.js (belum di-scaffold)
- ⬜ Mobile app Expo (belum di-scaffold)

Detail checklist per project: `apps/api/docs/todo.md`, `apps/admin/docs/todo.md`, `apps/mobile/docs/todo.md`.

## 7. Setup lokal cepat
```bash
npm install
cp apps/api/.env.example apps/api/.env   # isi DATABASE_URL (Neon), JWT_SECRET, dst
npm run db:push
npm run db:seed
npm run dev:api      # port 3000
```

## 8. Kalau ragu
- Kontrak API tidak jelas? → cek `apps/api/docs/erd.md` (skema data) dan `apps/api/docs/architecture.md` (contoh pola route handler)
- Desain tidak jelas? → buka langsung file `_reference/*.tsx`, itu sumber kebenaran paling detail
- Ubah shape response API? → update juga `apps/api/docs/todo.md`/`erd.md` di commit yang sama, jangan biarkan dokumen basi