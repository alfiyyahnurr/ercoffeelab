# ERCoffeeLab — Monorepo

📖 **Kalau pakai Antigravity AI atau AI agent lain untuk ngerjain, baca `docs/overview.md` dulu.**

```
apps/
  admin/   Panel admin (Next.js — DIBUAT ULANG, project Vite lama di-deprecate). apps/admin/docs/
  api/     Backend (Next.js API routes + Neon Postgres, raw SQL, Drizzle untuk migrate+seed). apps/api/docs/
  mobile/  React Native Expo (belum di-scaffold). apps/mobile/docs/
docs/      Overview lintas project + panduan deploy
```

Tiap `apps/*/docs/` punya 6 file standar: `prd.md`, `architecture.md`, `erd.md`, `navigation-flow.md`, `design-system.md`, `todo.md`.

## Setup awal

```bash
npm install
cp apps/api/.env.example apps/api/.env   # isi DATABASE_URL, JWT_SECRET, dst
npm run db:push
npm run db:seed
```

## Login admin panel — 2 cara
```
Google SSO, ATAU akun demo:
  admin@ercoffeelab.com / demo1234           (super_admin)
  bandung.admin@ercoffeelab.com / demo1234   (outlet_admin)
```

## Menjalankan tiap app

```bash
npm run dev:api      # backend, port 3000
npm run dev:admin    # admin panel (setelah di-scaffold ulang pakai Next.js — apps/admin/docs/todo.md)
```

## Status migrasi
- [x] Fase 1 — Schema Neon + seed
- [x] Fase 2 — Auth OTP customer + SSO staff + login email/password staff (akun demo)
- [ ] Fase 3 — Order lifecycle, outlet x menu, payment method toggle
- [ ] Fase 4 — Midtrans
- [ ] Fase 5 — Fontee (WA/email) + notification templates
- [ ] Fase 6 — Voucher + loyalty tier otomatis
- [ ] Fase 7 — Admin panel Next.js (dibuat ulang dari nol)
- [ ] Fase 8 — Mobile app (Expo, scaffold dari nol)

Checklist detail per project: `apps/*/docs/todo.md`