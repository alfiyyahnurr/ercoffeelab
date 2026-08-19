# CLAUDE.md

> Baca **`AGENTS.md`** di root repo dulu — itu sumber utama untuk aturan proyek (stack, konvensi kode & response API, skema DB, sequence antar sistem, dokumentasi wajib). File ini cuma tambahan khusus untuk Claude Code.

## Konteks singkat

ERCoffeeLab — monorepo 3 aplikasi. `apps/api` (Next.js 15 + Neon Postgres, raw SQL, JWT auth, Midtrans, Fontee, tanpa RLS) sudah terdokumentasi lengkap (`prd.md`, `erd.md`, `architecture.md`, `design-system.md`, `navigation-flow.md`, `todo.md`). `apps/admin` dan `apps/mobile` belum.

## Cara Claude harus kerja di repo ini

- **Cek `apps/api/todo.md` di awal sesi** untuk tahu task berikutnya. Jangan lompat urutan tanpa konfirmasi user.
- **Ikuti pola route handler dan konvensi response API persis** (lihat `AGENTS.md` § Konvensi kode). Poin yang paling sering kelupaan:
  - Transform `snake_case` → `camelCase` sebelum `NextResponse.json(...)` — raw SQL return snake_case apa adanya.
  - HTTP status baku (400/401/403/404/409/500), bukan asal pilih.
  - Response list selalu dibungkus `{ data: [...] }`, single resource object langsung.
- **Jangan pakai Drizzle ORM query builder** untuk query runtime — proyek ini sengaja pakai raw SQL via `@neondatabase/serverless`. Drizzle cuma untuk `db:push` dan `src/db/seed.ts`.
- **Outlet scoping**: setiap endpoint yang menyentuh data outlet-spesifik (orders, alerts, product_outlets, dsb), double-check filter `outlet_id` untuk role `outlet_admin` — tidak ada RLS di DB, jadi ini murni tanggung jawab kode. Ini titik paling rawan bocor data lintas outlet.
- **Ikuti sequence yang sudah didefinisikan** di `docs/navigation-flow.md` untuk alur multi-step (checkout→charge→webhook→loyalty→notifikasi, login staff dua jalur, dsb) — jangan improvisasi urutan langkah sendiri.
- **Logging third-party wajib**: setiap panggilan Midtrans/Fontee dicatat ke `payment_logs`/`notification_logs`, termasuk kasus error/gagal.
- **Setelah endpoint selesai**: update checkbox di `apps/api/todo.md`, tunjukkan contoh test manual (curl) — proyek ini belum punya test runner otomatis.
- **File yang tidak boleh disentuh/dibaca ulang tanpa perlu**: `node_modules/`, `.next/`, `.env` (baca hanya kalau eksplisit diminta debug env, jangan pernah print isinya ke chat/commit).

## Saat mengerjakan `apps/admin` atau `apps/mobile`

Dokumentasi untuk dua app ini (navigasi UI, design system visual) masih kosong. Sebelum menulis kode:
1. Baca langsung `apps/admin/package.json` atau `apps/mobile/package.json` untuk konfirmasi stack.
2. Untuk konvensi response API yang dikonsumsi dari sisi client, tetap acu ke `docs/design-system.md` (camelCase, format error, dsb) — itu kontrak dari sisi API yang harus dipatuhi consumer manapun.
3. Laporkan ke user kalau ada keputusan UI yang butuh dokumen yang belum ada.

## Testing

Belum ada test runner otomatis. Pola yang dipakai: **test manual via curl** per endpoint. Kalau user minta test otomatis (Jest/Vitest dsb), tanya dulu preferensi mereka sebelum setup dari nol.