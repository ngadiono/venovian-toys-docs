# 14 — Migrasi Monorepo (Rancangan & Rencana Eksekusi)

**Status:** disetujui owner 2026-07-05 · dirancang Fable (CTO) · eksekusi oleh Sonnet
via Task Brief dari Opus (T-51…T-61 di [10 — Task Tracker](10-task-tracker.md)).

## Kenapa monorepo (ringkas — detail alasan ada di ADR-001)

1. Logika domain terduplikasi di 2 app dan sudah menghasilkan bug kembar (INT-2;
   T-42=T-22, T-43=T-21). T-33/T-08 (modul uang & tipe bersama) butuh satu repo.
2. Dua app menulis ke satu Firestore → perubahan seam harus atomik (1 PR menyentuh
   skema + kedua konsumen), bukan 2 PR di 2 repo yang bisa desync.
3. Workflow agent (Opus/Sonnet) jauh lebih sederhana: satu backlog, satu worklog,
   satu CLAUDE.md, satu PR per task lintas-app.

## Kondisi awal (sudah dikerjakan manual, 2026-07-05)

- ✅ Backup Firestore prod: 166 dokumen, 15 koleksi (`backups/venovian-toys/2026-07-05T03-31-08/`
  di laptop owner; skrip `npm run backup` di repo backoffice).
- ✅ Repo GitHub `venovian-toys` di-rename → **`venovian-toys-web`**; Vercel tetap
  ter-deploy dari repo ini (diverifikasi owner).
- ✅ Remote lokal backoffice sudah menunjuk `venovian-toys-web`.
- ✅ Repo baru **kosong**: `https://github.com/ngadiono/venovian-toys.git` (private).

## Struktur target

```
venovian-toys/                        ← repo ngadiono/venovian-toys (monorepo)
├── CLAUDE.md                         ← T-57
├── README.md
├── package.json                      ← npm workspaces: ["apps/*", "packages/*"]
├── package-lock.json                 ← SATU lockfile root (per-app lockfile dihapus)
├── .gitignore  /  .easignore
├── .github/workflows/ci.yml          ← T-56
├── .claude/ (settings.json, skills/) ← T-57
├── firebase/
│   ├── firestore.rules               ← pindahan dari apps/backoffice
│   └── firestore.indexes.json        ← menyusul (T-12)
├── apps/
│   ├── backoffice/                   ← isi repo venovian-toys-web, utuh + histori
│   └── mobile/                       ← isi repo venovian-toys-mobile, utuh + histori
├── packages/
│   └── shared/                       ← T-58: kerangka KOSONG (kode pindah nanti, T-33/T-08)
└── docs/                             ← isi repo venovian-toys-docs, utuh + histori
    ├── 01…14, README
    ├── workflow/AGENT-WORKFLOW.md
    ├── adr/
    └── worklog/
```

## Aturan keras migrasi (masuk ke setiap Task Brief terkait)

1. **Pindah, bukan tulis ulang.** Kode `src/` kedua app TIDAK berubah. Setiap
   penyimpangan dari "murni pindah" (mis. config workspace) harus disebut eksplisit
   di Implementation Report dengan alasannya. Target diff `src/`: **nol**.
2. **Repo lama read-only** sejak T-51 dimulai: tidak ada commit task apa pun ke
   `venovian-toys-web` / `-mobile` / `-docs` (kecuali hotfix produksi darurat — bila
   terjadi, migrasi di-rebase/ulang subtree-nya, lapor Opus).
3. **Histori wajib ikut** (`git subtree add`) — git log/blame ketiga repo harus tetap
   bisa ditelusuri di monorepo.
4. **Gerbang verifikasi berurutan** (lihat per-task): build lokal → Vercel cutover →
   EAS APK → baru archive repo lama. Selama gerbang belum lolos, jalan pulang =
   repo lama yang masih utuh.
5. Kerja dilakukan di clone terpisah `~/Development/Projects/venovian-toys-mono`
   (di LUAR folder payung, supaya tiga repo lama tak tersentuh). Swap folder terjadi
   di T-59.

## Rencana per task

### T-51 — Rakit monorepo: subtree import 3 repo + workspaces (size L, pelaksana Sonnet)

Garis besar teknis:

```bash
cd ~/Development/Projects
git clone https://github.com/ngadiono/venovian-toys.git venovian-toys-mono
cd venovian-toys-mono
# commit awal: README singkat + .gitignore root (agar subtree punya parent)
git subtree add --prefix=apps/backoffice ~/Development/Projects/venovian-toys/venovian-toys main
git subtree add --prefix=apps/mobile     ~/Development/Projects/venovian-toys/venovian-toys-mobile main
git subtree add --prefix=docs            ~/Development/Projects/venovian-toys/venovian-toys-docs main
```

Lalu: root `package.json` (`"workspaces": ["apps/*", "packages/*"]`, `private: true`);
beri nama workspace `@venovian/backoffice` dan `@venovian/mobile` di package.json
masing-masing (satu-satunya edit file app yang diizinkan di task ini); hapus
`package-lock.json` per-app; `npm install` di root → satu lockfile; gabungkan/naikkan
`.gitignore` yang perlu ke root. Push ke `origin main`.

AC: (a) `git log --follow` file lama tetap berhistori; (b) `diff -r` isi
`apps/backoffice` vs repo lama = hanya file yang memang diubah (package.json name,
lockfile) — sisanya identik; (c) `npm install` root sukses tanpa error resolusi;
(d) belum ada perubahan `src/` sama sekali.

### T-52 — Backoffice hijau di monorepo + `firebase/` (size M, Sonnet)

- `next build` dari `apps/backoffice` hijau (env: copy `.env.local` owner ke
  `apps/backoffice/.env.local`).
- `git mv apps/backoffice/firestore.rules firebase/firestore.rules`; cari referensi
  path-nya (README/docs/scripts) dan perbarui.
- `npm run lint` + `npx tsc --noEmit` hijau.
- AC: build+lint+tsc hijau; tidak ada perubahan `src/` selain yang mutlak (target nol).

### T-53 — Mobile hijau di monorepo (size M, Sonnet + owner cek device)

- `npx tsc --noEmit` + `npm run lint` hijau dari `apps/mobile`.
- `npx expo start` → app jalan di device/simulator (owner yang memegang HP).
- Expo SDK 56 mendeteksi workspace otomatis; **hanya bila** ada modul gagal resolve,
  tambah `metro.config.js` minimal (watchFolders root) — sebut di report.
- Tambah `.easignore` root: exclude `apps/backoffice`, `docs`, `backups`, artefak berat;
  `packages/` dan `apps/mobile` tetap ikut.
- AC: tsc+lint hijau; owner konfirmasi app jalan; diff `src/` nol.

### T-54 — EAS build preview APK dari monorepo (size S, Sonnet memandu + owner eksekusi)

- Dari `apps/mobile`: `eas build --profile preview --platform android`.
- **Tidak ada setting cloud EAS yang diubah** (projectId/credentials tetap).
- AC: build sukses; owner install APK di HP; fitur utama jalan (login, titipan, invoice).

### T-55 — Cutover Vercel (size S, owner manual dipandu Opus)

- Vercel → Settings → Git → connected repo = `ngadiono/venovian-toys`;
  Build & Deployment → Root Directory = `apps/backoffice`. Env vars tidak disentuh.
- Deploy → Ready → owner cek app produksi (login + beberapa halaman + satu aksi tulis).
- Rollback bila gagal: kembalikan connected repo ke `venovian-toys-web` (≈2 menit).
- AC: produksi jalan dari monorepo.

### T-56 — CI + branch protection (size M, Sonnet)

- `.github/workflows/ci.yml`: job per app dengan path filter
  (`apps/backoffice/**` → lint+tsc+build backoffice; `apps/mobile/**` → lint+tsc mobile;
  `packages/**` → keduanya). Node 22, cache npm.
- Branch protection `main` (via `gh api`): require PR, require status checks, block
  force-push. Baru dinyalakan SETELAH T-55 (selama perakitan, push langsung ke main
  monorepo diizinkan karena belum melayani produksi).
- AC: PR dummy menunjukkan check jalan & path filter benar.

### T-57 — CLAUDE.md + skills + settings (size M, Opus boleh kerjakan sendiri — ini docs/tooling, bukan kode app)

- `CLAUDE.md` root: konteks proyek, aturan keras (lihat AGENT-WORKFLOW § aturan keras),
  perintah umum, konvensi git.
- Port skills octobot → `.claude/skills/`: `mulai-sesi`, `tutup-sesi`, `next-task`,
  `review-task`, `adr`, `progress`, `index-codebase`, dan **`seam-audit`** (baru,
  pengganti `risk-audit`): checklist = auth/`requireSession` di server actions, uang
  integer rupiah, WIB, field denormalized vs doc 12, Firestore rules tidak melonggar,
  kuota Spark (onSnapshot/read amplification), rencana migrasi data prod, update doc 13.
- `.claude/settings.json`: allow lint/tsc/build/git-read; deny Read .env*, deny
  `npm run clear:db` dan seed ke prod.
- AC: skill terbaca di sesi baru; referensi path di dalamnya valid untuk monorepo.

### T-58 — Kerangka `packages/shared` (size S, Sonnet)

- `packages/shared/package.json` (`@venovian/shared`) + `src/index.ts` kosong +
  tsconfig; `transpilePackages: ['@venovian/shared']` di next.config backoffice;
  smoke-import di kedua app lalu dihapus ATAU dibiarkan minimal (1 konstanta versi).
- **JANGAN memindahkan logika apa pun ke shared di task ini** — itu T-33/T-08.
- AC: kedua app build hijau dengan dependency `@venovian/shared` terpasang.

### T-59 — Beres-beres: archive repo lama + swap folder lokal (size S, owner dipandu)

- Archive di GitHub (JANGAN delete): `venovian-toys-web`, `venovian-toys-mobile`,
  `venovian-toys-docs`. Boleh ditunda beberapa hari setelah T-55 stabil.
- Lokal: folder payung `~/Development/Projects/venovian-toys/` diganti clone monorepo
  (rename `venovian-toys-mono` → `venovian-toys`); tiga folder repo lama dipindah ke
  arsip lokal dulu (mis. `~/Development/Projects/_arsip-venovian/`), dihapus nanti
  kalau sudah yakin.
- Update link lintas-repo di docs (kini path relatif dalam monorepo).
- AC: sesi kerja baru dibuka di monorepo; `git pull/push` normal; docs tidak punya
  link mati.

### T-60 — Revisi AGENTS.md kedua app (size S, Sonnet)

- Hapus/ubah aturan "jangan jalankan tsc/lint/build" (bertentangan dengan workflow);
  ganti: verifikasi lint+tsc wajib sebelum lapor selesai; dev server tetap urusan owner.
- Tambah rujukan: protokol kerja di `docs/workflow/AGENT-WORKFLOW.md`, backlog di
  `docs/10-task-tracker.md`. Aturan desain UI yang ada DIPERTAHANKAN.

### T-61 — Env preview Vercel → Firebase dev (size S, owner manual dipandu Opus)

- Di Vercel: environment variables scope **Preview** diarahkan ke project Firebase dev
  (`venovian-toys-dev`), scope Production tetap prod. Hasil: preview deployment aman
  untuk uji tulis-data tanpa menyentuh data nyata.
- Catat di doc 09 + doc 13.

## Risiko & mitigasi

| Risiko | Mitigasi |
|---|---|
| Metro/Expo gagal resolve dep karena hoisting | Gerbang T-53/T-54 sebelum archive; fallback metro.config standar; repo lama utuh |
| Vercel build beda perilaku di subdirectory | Preview deploy dulu di T-55; rollback = ganti connected repo kembali |
| Hotfix produksi dibutuhkan di tengah migrasi | Hotfix di repo lama → selesai → ulang subtree add (histori linear, murah) |
| Lockfile tunggal mengubah versi resolved | `npm install` tanpa update: pertahankan versi dari lockfile lama sebisanya; `npm ls` diff dicek di review T-51 |
| Link antar-docs putus | T-59 menyisir semua link; sebelum itu docs lama tetap sumber kebenaran |
