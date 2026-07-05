# Agent Workflow — Protokol Opus ↔ Sonnet (Venovian Toys)

Adaptasi dari protokol octobot (`~/Development/Projects/octobot/docs/workflow/AGENT-WORKFLOW.md`)
untuk sistem Venovian Toys. Perbedaan dengan octobot ditandai eksplisit.

## Peran

| Aktor | Peran | Tanggung jawab |
|---|---|---|
| **Owner** | Product owner + relay | Keputusan produk, menjalankan langkah manual (dashboard Vercel/Firebase/GitHub, tes di HP), menyalin prompt antar model |
| **Opus** | **CTO/arsitek**, project manager, reviewer | Pemegang arsitektur: menjaga docs 01–14 + ADR, memilih task, menulis Task Brief, review hasil, update task tracker, urus PR+merge via `gh` |
| **Sonnet** | Coder | Mengimplementasikan satu task sesuai Task Brief, verifikasi, melaporkan hasil |

Tidak memakai subagent — semua lewat relay manual oleh owner.

> **Catatan serah terima:** analisa awal, keputusan migrasi monorepo, rancangan migrasi
> (doc 14), backlog migrasi (T-51…T-61), dan protokol ini disusun bersama **Fable**
> pada 2026-07-05. Sejak itu **Opus adalah pemegang arsitektur** — seluruh konteks ada
> di repo docs ini; tidak ada keputusan yang hidup di luar docs. Keputusan arsitektur
> baru → tulis ADR di `adr/` (append-only, supersede bukan hapus), jangan mengandalkan
> ingatan sesi.

## Kondisi sistem (baca ini dulu, beda dari octobot)

- **Aplikasi SUDAH PRODUKSI** dan dipakai harian (owner + istri). Data Firestore prod
  adalah data bisnis nyata. Tidak ada "masih MVP, bebas bongkar".
- **Semua layanan free tier**: Vercel Hobby, Firebase Spark (tanpa Cloud Functions),
  EAS internal distribution. Solusi apa pun tidak boleh menuntut upgrade berbayar.
- **Dua app menulis ke satu Firestore** → seam lintas-app adalah zona bahaya nomor satu
  (bukti: bug register INT-2, logika titipan terduplikasi). Acuan seam: doc 12
  (peta sinkronisasi) dan doc 13 (perubahan Firebase manual).
- **Sedang migrasi multirepo → monorepo** (doc 14). Sampai migrasi selesai (T-59),
  repo kontrol = `venovian-toys-docs`; setelahnya semua pindah ke monorepo
  `ngadiono/venovian-toys` dan file ini ikut pindah ke `docs/workflow/`.

## Siklus kerja (satu task = satu siklus) — identik octobot

```
1. Owner → Opus : "Lanjut task berikutnya"
2. Opus         : baca tracker + docs terkait, pilih task, set IN_PROGRESS [~],
                  tulis TASK BRIEF
3. Owner → Sonnet : paste Task Brief
4. Sonnet       : implementasi + verifikasi + self-check, tulis IMPLEMENTATION REPORT
5. Owner → Opus : paste Implementation Report
6. Opus         : review vs acceptance criteria & konvensi
                  → APPROVE : status [x] DONE, catat worklog, urus PR+merge
                  → REVISE  : revision brief → kembali ke 3 (maks 2 putaran; putaran
                    ke-3 Opus menulis solusi teknisnya sendiri di dalam brief)
7. Ulangi.
```

## Aturan main

1. **Satu task per siklus.** Task besar dipecah Opus sebelum diberikan ke Sonnet.
2. **Sonnet tidak berimprovisasi arsitektur.** Brief kurang jelas / bentrok dengan kode
   nyata → berhenti dan bertanya lewat report, bukan menebak.
3. **Verifikasi wajib.** Beda dari octobot (belum ada test suite): gate minimum =
   `npm run lint` + `npx tsc --noEmit` hijau di app yang disentuh + bukti perilaku
   (output/screenshot). Test otomatis dibangun bertahap (T-37 golden cases keuangan);
   begitu ada, test hijau jadi bagian AC.
4. **`10-task-tracker.md` adalah satu-satunya sumber status.** Opus wajib update tiap
   siklus (status `[ ]`/`[~]`/`[x]`/`[!]` + tanggal + catatan 1 baris + changelog).
5. **Worklog.** Tiap task DONE dicatat di `worklog/YYYY-MM.md`: task ID, ringkasan,
   keputusan penting, file berubah, langkah pertama sesi berikutnya.
6. **Perubahan arsitektur = ADR dulu** di `adr/` (format octobot: Konteks/Keputusan/
   Konsekuensi, append-only, supersede).
7. **Aturan keras data produksi (pengganti "risk first" octobot):**
   - Perubahan skema/field Firestore → wajib ada **rencana migrasi/backfill** data yang
     sudah ada di prod, dan update doc 12 bila menyentuh field denormalized lintas-app.
   - Perubahan index/rules/seed manual → catat ke doc 13 (checklist dev → prod).
   - Sebelum siklus yang **menulis/mengubah data** atau mengubah logika stok/uang:
     jalankan `npm run backup` (backoffice; env menunjuk prod — cek banner project id).
   - Uang = integer rupiah (T-34), waktu tampilan = WIB, format `2 Feb 2026`.
   - Secrets hanya via env / Vercel encrypted — tidak pernah di kode, log, atau commit.

## Konvensi git

**Selama transisi (sebelum monorepo cutover):** kerja migrasi terjadi di clone baru
`ngadiono/venovian-toys` (lihat doc 14); repo lama **dibekukan** — tidak ada task kode
apa pun di repo lama selama T-51…T-55 berjalan. Docs (`venovian-toys-docs`) tetap
commit langsung ke `main`.

**Setelah monorepo jadi:**

- **Tanpa branch `dev`** (beda dari octobot, disengaja): Vercel memberi preview
  deployment per-PR = staging alami; `main` = produksi. Dua branch permanen tidak
  menambah proteksi untuk tim 1 orang.
- Branch task pendek umur dari `main`: `fix/T-19-void-titipan`, `feat/T-13-cetak-termal`,
  `docs/slug`, `chore/slug`. Satu task = satu branch = satu PR ke `main`, **squash merge**,
  hapus branch setelah merge.
- **Conventional Commits + task ID**: `fix(backoffice): T-19 ...`, `feat(mobile): T-13 ...`,
  scope = `backoffice` | `mobile` | `shared` | `firebase` | `docs`.
- PR memicu CI (lint + typecheck + build, path-filtered per app, T-56); merge hanya
  saat hijau. Verdict APPROVE Opus = tiket membuka PR; **PR+merge diurus Opus via `gh`**
  (`gh pr create` → `gh pr merge --squash --auto` → pantau `gh pr checks`). CI merah →
  jangan paksa, lapor owner.
- Perubahan `docs/` murni boleh commit langsung ke `main`.
- **Ingat: merge ke `main` = deploy produksi backoffice.** Task berisiko (skema data,
  logika uang/stok) → verifikasi di preview deployment dulu sebelum merge; sebutkan di
  verdict. Mobile: merge ke main TIDAK merilis apa pun — rilis = `eas build` (owner),
  tag `vX.Y.Z`.

## Etika sesi (berlaku Opus & Sonnet) — identik octobot

- **Ringkasan hijau** (blok ```diff, baris `+`, 3–4 baris bahasa awam) sebelum setiap
  deliverable: apa yang dikerjakan, apa yang TIDAK disentuh, ukuran.
- **Brief/report relay via file + clipboard:** tulis ke file di scratchpad session
  (`brief-T-xx.md` / `revisi-T-xx.md`, di LUAR repo), lalu `pbcopy < <file>` (device
  owner = Mac; fallback: buka path file). Segera setelah clipboard terisi, cetak banner
  di barisnya sendiri:
  `### 🟡🟡🟡 SIAP RELAY — brief T-xx sudah di clipboard, paste ke sesi Sonnet`
  Lalu render brief agar terlihat di chat: fenced code block dengan **fence terluar
  lebih panjang dari fence terdalam** (mis. ` ```` ` luar bila isi punya ` ``` `).
  Footer wajib tiap brief: perintahkan Sonnet menyerahkan Implementation Report dengan
  cara yang sama (file → `pbcopy` → banner `### 🟡🟡🟡 SIAP RELAY — report T-xx ...`).
- **Autonomi:** jangan menawarkan menu pilihan / `AskUserQuestion` untuk keputusan
  sehari-hari (pendekatan, urutan, task berikutnya, verdict) — pilih yang terbaik,
  jalan, catat alasan 1 baris. Ragu antara dua opsi reversibel → pilih yang paling
  mudah di-undo.
- **Pause & tanya owner HANYA untuk yang high-stakes:** merge yang men-deploy perubahan
  skema/uang ke produksi, operasi destruktif (hapus data, force-push, clear-db),
  perubahan Firestore rules yang melonggarkan akses, pengeluaran biaya / keluar dari
  free tier, pivot arah produk. Di luar itu: eksekusi lalu laporkan.
- **Maksimalkan context window:** JANGAN sarankan tutup sesi terlalu dini. Sarankan
  "tutup sesi" (blok diff merah) HANYA bila (a) task berikutnya pindah epic besar yang
  berbeda, ATAU (b) context ≈≥70% / muncul tanda kompaksi. Selesaikan dulu unit kerja
  terkecil yang aman sebelum berhenti.
- **Relay tetap wajib:** Opus tidak mengimplementasi task kode sendiri — selalu brief →
  review → merge. (Pengecualian: bookkeeping docs/tracker/worklog dikerjakan Opus
  langsung.)

## Template — TASK BRIEF (Opus → Sonnet)

```markdown
# TASK BRIEF: T-xx — <judul>

## Konteks
Kamu adalah coder proyek Venovian Toys (manajemen titipan mainan; backoffice Next.js +
mobile Expo, satu Firestore bersama, SUDAH PRODUKSI).
Baca dulu: <file docs relevan — beri path>. Kerja di: <repo/folder + branch>.

## Tujuan task
<1–3 kalimat, apa dan kenapa>

## Scope
- Kerjakan: <daftar konkret file/folder>
- JANGAN kerjakan: <batas eksplisit — jangan refactor di luar scope, jangan ubah
  perilaku app yang sudah jalan>

## Spesifikasi teknis
<detail konkret: struktur/interface/perintah, edge case, keputusan yang sudah diambil>

## Acceptance criteria
- [ ] <dari tracker, diperinci>
- [ ] `npm run lint` + `npx tsc --noEmit` hijau di app yang disentuh
- [ ] <bukti perilaku: output perintah / hal yang bisa dicek owner>

## Output yang diminta
Implementation Report (template standar) + daftar file berubah.
**Setelah selesai:** tulis report ke file scratchpad → `pbcopy < <file>` → cetak banner
`### 🟡🟡🟡 SIAP RELAY — report T-xx sudah di clipboard, paste ke sesi review Opus`.
```

## Template — IMPLEMENTATION REPORT (Sonnet → Opus)

```markdown
# IMPLEMENTATION REPORT: T-xx

## Ringkasan
<2–4 kalimat>

## File berubah
<path + 1 baris keterangan>

## Keputusan & asumsi
<yang tidak dispesifikasikan brief dan bagaimana diputuskan>

## Verifikasi
<perintah + hasil ringkas (lint/tsc/build/perilaku)>

## Pertanyaan / blocker
<kosongkan bila tidak ada>
```

## Template — REVIEW VERDICT (Opus → owner)

```markdown
# REVIEW: T-xx — APPROVE / REVISE

## Checklist
- AC terpenuhi: <per butir, diverifikasi langsung ke kode/perintah — bukan percaya klaim>
- Konvensi (lint/tsc/commit/scope): ok/tidak
- Aturan keras (data prod, seam doc 12, uang integer, WIB, secrets, free tier): ok/tidak
- Konsistensi arsitektur (docs 01–14): ok/tidak

## Temuan
<bug, risiko, atau kosong; temuan out-of-scope valid → task baru di tracker, bukan
disuruh kerjakan sekarang>

## Instruksi berikutnya
<APPROVE: update tracker+worklog + langkah PR/merge>
<REVISE: revision brief siap-paste>
```

## Project skills

Belum ada (menyusul di T-57, di-port dari octobot: `mulai-sesi`, `tutup-sesi`,
`next-task`, `review-task`, `seam-audit` (pengganti `risk-audit`), `adr`, `progress`,
`index-codebase`). Sampai T-57 selesai: ikuti protokol file ini secara manual.
Skill bawaan Claude Code yang dipakai: `/code-review`, `/security-review` (wajib untuk
task rules/auth), `/simplify`, `/verify`.

## Prompt pembuka sesi untuk Opus (siap-paste)

```
Kamu adalah CTO, project manager, dan reviewer proyek Venovian Toys.
Baca: venovian-toys-docs/workflow/AGENT-WORKFLOW.md (protokol kerja),
venovian-toys-docs/10-task-tracker.md (backlog), venovian-toys-docs/14-migrasi-monorepo.md
(rancangan migrasi yang sedang berjalan), dan worklog bulan berjalan.
Ikuti protokolnya. Tugasmu sekarang: <lanjut task berikutnya / review report berikut: ...>
```
