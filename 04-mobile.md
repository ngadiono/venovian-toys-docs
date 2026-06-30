# 04 — Mobile Lapangan

Repo: `venovian-toys-mobile` · Expo ~56 / React Native 0.85 · expo-router · EAS Build.

- Bundle id: `com.venovian.toysmobile` (iOS & Android)
- Scheme: `venoviantoys` · Owner EAS: `ngadiono7s-organization`
- EAS projectId: `3166a4d7-3242-4af7-b5b8-69b9d8d68849`

## Peran

Mobile adalah **companion lapangan** untuk sales: subset backoffice yang dioptimalkan
untuk kerja di lokasi warung. Bukan pengganti backoffice. Sumber kebenaran konfigurasi
(status, kategori, tipe) tetap di backoffice.

## Peta layar (expo-router)

| Route | Layar | Fungsi |
|-------|-------|--------|
| `login.tsx` | Login | Email/password Firebase |
| `index.tsx` | Home | Dashboard ringkas (ringkasan keuangan, sinyal) |
| `customers.tsx` | Pelanggan | Daftar mitra + detail + kunjungan |
| `catalog.tsx` | Katalog | Daftar produk (baca) |
| `prospects.tsx` | Prospek | Daftar prospek scan area + update status kunjungan |
| `settings.tsx` | Pengaturan | Tema, profil, logout |

Komponen modal utama: `AddConsignmentModal` (tambah titipan), `CustomerStockDetailModal`,
`CustomerStockReceiptModal` (struk titipan — arah ke cetak termal).

## Alur kerja lapangan (field workflow)

```
1. Buka prospek (prospects.tsx)
      └─ tandai "sudah dikunjungi" + catatan  →  Firestore scanProspects
2. Deal warung baru  →  konversi prospek jadi pelanggan
3. Titip stok (AddConsignmentModal)
      └─ buat consignment note (runTransaction)  →  Firestore
4. Kunjungan rutin (customers.tsx)
      └─ catat penjualan / tarik sisa stok
5. Tagih / setor  →  lihat invoice (invoices.ts)
6. Cetak struk (target: printer termal portable)
```

Brief lengkap: `venovian-toys-mobile/docs/mobile-product-brief.md`.
Roadmap fase: `venovian-toys-mobile/docs/mobile-roadmap.md`.

## Akses data

Mobile memakai **Firebase JS SDK langsung** (tanpa server perantara). Detail service di
[02 — Arsitektur](02-arsitektur.md#mobile-lapisan-akses-data). Poin penting:

- `client.ts` meng-init Auth dengan **AsyncStorage persistence** agar sesi bertahan.
- Konfigurasi via env `EXPO_PUBLIC_FIREBASE_*` (lihat [09 — Deployment](09-deployment.md)).
- Mutasi stok (`consignment-notes`, `customer-stocks`) pakai `runTransaction` di device.
- Pembacaan realtime via `onSnapshot` (customers, products, prospects).

### Strategi hemat Spark plan

Karena Firebase Spark (gratis, tanpa Cloud Functions), pembacaan dirancang hemat:
hindari full-collection scan, batasi query, agregasi seperlunya. Aturan & guardrail:
`venovian-toys-mobile/docs/mobile-firebase-free-plan.md`. Saat menambah fitur baca,
**cek dulu** dokumen ini agar tidak meledakkan kuota baca.

## Cetak termal

Target: printer termal portable (Bluetooth) untuk struk titipan/tagihan. Memerlukan
**Expo development build** (bukan Expo Go). Abstraksi printing service + prinsip template:
`venovian-toys-mobile/docs/mobile-printing-strategy.md`.

## Tema & UI

Tema terang/gelap dikelola Zustand (`theme-store.ts`, `use-theme.ts`). Komponen tema:
`themed-text`, `themed-view`. Tab bar bisa sembunyi saat scroll (`use-hide-tab-on-scroll`).
Ada varian `.web.tsx` untuk beberapa komponen (dukungan `expo start --web`).

## Catatan dokumentasi yang perlu dibetulkan

- **`README.md` mobile basi**: masih menyebut "web application" dan "Next.js (App Router)"
  — disalin dari backoffice. Faktanya ini Expo/React Native. Lihat
  [07 — Bug Register](07-bug-register.md) #DOC-1.
- **`AGENTS.md` mobile** masih menyatakan fase "strictly UI-first / mock data, jangan
  integrasi Firebase", padahal service Firestore nyata sudah ada & dipakai. Aturan ini
  basi dan bisa menyesatkan sesi AI/agent berikutnya. Lihat #DOC-2.
