# ADR-001 — Migrasi multirepo → monorepo

**Tanggal:** 2026-07-05 · **Status:** Accepted · **Diputuskan oleh:** owner, atas rekomendasi Fable (CTO)

## Konteks

Sistem terdiri dari 3 repo (`venovian-toys` backoffice, `venovian-toys-mobile`,
`venovian-toys-docs`) dengan dua app yang berbagi satu Firestore. Masalah yang terbukti:

- Logika domain (titipan/stok/uang) ditulis dua kali dan menghasilkan bug kembar
  (bug register INT-2; T-42=T-22, T-43=T-21).
- Backlog T-33 (modul uang/stok bersama) dan T-08 (tipe/skema bersama) tidak praktis
  di multirepo tanpa publish package privat.
- Perubahan seam lintas-app butuh 2 PR di 2 repo yang bisa desync; kontrak field
  denormalized (doc 12) rapuh.
- Workflow agent (Opus PM ↔ Sonnet coder, adopsi octobot) jauh lebih sederhana di
  satu repo: satu backlog, satu worklog, satu CLAUDE.md, PR atomik lintas-app.

Alasan-alasan umum memilih multirepo (tim terpisah, access control, versioning
independen) tidak ada di proyek ini (solo, satu produk, satu database).

## Keputusan

1. Satu monorepo `ngadiono/venovian-toys`: `apps/backoffice`, `apps/mobile`,
   `packages/shared`, `firebase/`, `docs/` — struktur lengkap di doc 14.
2. Repo lama di-rename/diarsip (bukan dihapus): backoffice → `venovian-toys-web`,
   plus `-mobile` dan `-docs` diarsip setelah cutover stabil (T-59).
3. Histori git ketiga repo dibawa masuk via `git subtree add`.
4. Migrasi = pindah, bukan tulis ulang: diff `src/` kedua app ditarget nol.
5. Model branch monorepo: tanpa `dev` permanen (beda octobot) — branch task → PR →
   squash ke `main`; Vercel preview per-PR berperan sebagai staging.
6. npm workspaces + satu lockfile root; shared dikonsumsi sebagai source TS
   (Metro langsung; Next via `transpilePackages`).

Alternatif yang ditolak:

- **Tetap multirepo + workflow disesuaikan** — bisa jalan, tapi menunda (bukan
  menghindari) tagihan T-33/T-08 dan melanggengkan bug kembar.
- **Monorepo di repo backoffice lama (git mv in-place)** — ditolak demi rollback
  bersih: repo lama tak tersentuh sampai semua gerbang verifikasi lolos.
- **Branch `dev` permanen ala octobot** — upacara ekstra tanpa proteksi tambahan
  untuk tim 1 orang; preview deployment Vercel sudah menjadi staging.

## Konsekuensi

- Backlog: T-51…T-61 (doc 14 = rancangan eksekusi); FREEZE task kode di repo lama
  selama T-51…T-55.
- Docs: doc 14 baru; setelah T-59 semua link lintas-repo jadi path relatif monorepo;
  README docs diperbarui.
- Tooling: CI path-filtered (T-56), skills + CLAUDE.md di monorepo (T-57).
- Trade-off diterima: upload EAS lebih besar (dimitigasi `.easignore`); risiko
  hoisting Metro (dimitigasi gerbang T-53/T-54 sebelum archive).
