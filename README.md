# Venovian Toys — Dokumentasi Sistem

Repo ini adalah **dokumentasi payung** untuk seluruh sistem Venovian Toys: aplikasi
manajemen bisnis titipan mainan ke warung/toko. Sistem terdiri dari **dua aplikasi**
yang berbagi satu database Firebase yang sama.

| Aplikasi | Repo | Stack | Untuk siapa |
|----------|------|-------|-------------|
| **Backoffice** | `venovian-toys` | Next.js 16 (App Router) + Firebase | Operator kantor (admin) |
| **Mobile (lapangan)** | `venovian-toys-mobile` | Expo / React Native 0.85 + Firebase | Sales di lapangan |

> Repo ini **tidak menyalin** dokumentasi internal tiap aplikasi. Dokumen rinci per-app
> tetap tinggal di app masing-masing (`<app>/docs/`). Repo ini menyatukan pandangan
> lintas-app, kontrak integrasi, daftar bug, dan roadmap. Lihat
> [Peta Dokumen](#peta-dokumen) untuk tautan ke dokumen sumber.

## Mulai dari sini

1. Baru di proyek ini? Baca **[01 — Ringkasan Sistem](01-ringkasan-sistem.md)**.
2. Mau paham bagaimana 2 app saling terhubung? **[06 — Kontrak Integrasi](06-kontrak-integrasi.md)**.
3. Mau kembangkan/perbaiki? **[07 — Bug & Issue Register](07-bug-register.md)** dan **[08 — Roadmap](08-roadmap.md)**.

## Peta Dokumen

### Di repo ini (lintas-app)
- [01 — Ringkasan Sistem](01-ringkasan-sistem.md) — bisnis, dua app, hosting, alur besar
- [02 — Arsitektur](02-arsitektur.md) — struktur teknis kedua app + Firestore bersama
- [03 — Backoffice](03-backoffice.md) — peta fitur, route, server actions, services
- [04 — Mobile Lapangan](04-mobile.md) — peta layar, services, alur kerja sales
- [05 — Model Data](05-model-data.md) — ringkasan koleksi Firestore + tautan kanonik
- [06 — Kontrak Integrasi](06-kontrak-integrasi.md) — bagaimana 2 app berbagi Firestore
- [07 — Bug & Issue Register](07-bug-register.md) — temuan bug, risiko, status
- [08 — Roadmap](08-roadmap.md) — rencana pengembangan
- [09 — Deployment](09-deployment.md) — Vercel, Firebase, EAS, environment
- [10 — Task Tracker](10-task-tracker.md) — **backlog pengembangan yang dilacak** (T-01…)
- [11 — Fokus: Keuangan & Scan Area](11-fokus-keuangan-scan-area.md) — penguatan dua area vital (tetap gratis)

### Sumber kanonik di repo aplikasi
| Topik | Lokasi |
|-------|--------|
| Rancangan database Firestore (lengkap, 930 baris) | `venovian-toys/docs/database-design.md` |
| Autentikasi Firebase | `venovian-toys/docs/authentication.md` |
| Strategi Scan Area | `venovian-toys/docs/scan-area-strategy.md` |
| Strategi dokumen titipan & invoice | `venovian-toys/docs/invoice-strategy.md` |
| Arsitektur mobile | `venovian-toys-mobile/docs/mobile-architecture.md` |
| Brief produk mobile | `venovian-toys-mobile/docs/mobile-product-brief.md` |
| Strategi Spark plan (free tier) | `venovian-toys-mobile/docs/mobile-firebase-free-plan.md` |
| Strategi cetak termal | `venovian-toys-mobile/docs/mobile-printing-strategy.md` |
| Roadmap mobile | `venovian-toys-mobile/docs/mobile-roadmap.md` |

## Status dokumen

Dokumentasi ini disusun **2026-06-30** dengan membaca kode dan graphify-out kedua app.
Bagian yang ditandai `[VERIFIKASI]` belum dikonfirmasi penuh terhadap kode dan perlu
dicek saat audit. Bagian bug register adalah hasil pembacaan sepintas, **bukan** audit
keamanan/korektif menyeluruh — audit mendalam dijadwalkan sebagai langkah berikutnya.
