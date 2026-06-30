# 02 — Arsitektur

## Stack ringkas

| Lapisan | Backoffice | Mobile |
|---------|-----------|--------|
| Framework | Next.js 16.2 (App Router) | Expo ~56 / React Native 0.85 |
| Bahasa | TypeScript, React 19.2 | TypeScript, React 19.2 |
| Routing | App Router + Route Groups | expo-router (file-based) |
| UI | Tailwind v4 + shadcn/ui | Komponen RN custom + tema sendiri |
| Tabel | TanStack Table v8 | mobile-card / list custom |
| Form | React Hook Form + Zod | — (form RN langsung) |
| Peta | Leaflet / react-leaflet (OSM) | — (link ke Google Maps) |
| Chart | Recharts (shadcn chart) | victory-native |
| State | Zustand | Zustand |
| Akses Firebase | **firebase-admin** (server) + firebase (client auth) | **firebase** JS SDK (client) |

## Backoffice: lapisan akses data

Backoffice memakai pola **Server Actions + Firebase Admin SDK**. Mutasi penting tidak
dilakukan langsung dari komponen client, melainkan melalui:

```
Komponen (client)
   │  panggil server action
   ▼
src/actions/*.ts           ← validasi Zod, requireSession(), assertCanWrite()
   │
   ▼
src/services/firebase/*.ts ← query Firestore via Admin SDK (adminDb)
   │
   ▼
Firestore
```

- `src/actions/` — Server Actions, gerbang mutasi. Selalu cek sesi & role di sini.
- `src/services/firebase/` — query/akses Firestore (Admin SDK), plus `admin.ts`
  (`adminAuth`, `adminDb`, `getSessionUser`, `requireSession`).
- `src/validations/` — skema Zod terpusat.
- `src/store/` — Zustand (UI state).
- `src/components/features/<domain>/` — komponen per domain (tabel, dialog, form).
- `src/components/ui/` — shadcn/ui generik. `src/components/layout/` — sidebar, header.

**God nodes** (paling banyak terhubung, dari graphify): `cn()`, `requireSession()`,
`formatCapitalizedText()`, lalu primitive UI (`Button`, `Input`, `Dialog*`). `requireSession()`
dengan 70 edge menandakan hampir semua server action lewat gerbang sesi — pola yang sehat.

### Proteksi route (Next.js 16)

- `proxy.ts` (bukan `middleware.ts`) untuk pre-filter route publik vs terproteksi.
- Verifikasi sesi server-side di layout/route terproteksi.
- Sesi disimpan sebagai **session cookie HTTP-only** (Firebase Admin), bukan ID token di
  `localStorage`. Detail: `venovian-toys/docs/authentication.md`.

## Mobile: lapisan akses data

Mobile **tidak punya server**. Ia memakai Firebase JS SDK langsung dari device:

```
Layar (src/app/*.tsx)
   │
   ▼
src/services/firebase/*.ts ← query/transaksi Firestore via client SDK (db)
   │
   ▼
Firestore  (Security Rules = satu-satunya penjaga)
```

Service mobile yang ada (`src/services/firebase/`):

| File | Fungsi | Catatan |
|------|--------|---------|
| `client.ts` | init app + Auth dgn AsyncStorage persistence | fallback `getAuth` bila RN persistence tak ada |
| `config.ts` | baca env `EXPO_PUBLIC_FIREBASE_*` | `getMissingFirebaseConfigKeys()` untuk diagnosa |
| `auth.ts` | login email, logout, subscribe auth state, profil | |
| `customers.ts` | baca pelanggan (`onSnapshot`/`getDocs`) | tipe `Customer`, status `active/follow_up/inactive` |
| `customer-stocks.ts` | stok titipan per pelanggan | pakai `runTransaction` untuk mutasi |
| `consignment-notes.ts` | buat surat titipan | `runTransaction`, kuantitas selalu dalam pcs |
| `invoices.ts` | tagihan tertunda | status: draft/issued/paid/partial/unpaid/overdue/void |
| `products.ts` | katalog produk | `subscribeToProducts` (realtime) |
| `prospects.ts` | prospek scan area | `updateDoc` untuk update kunjungan |
| `dashboard.ts` | ringkasan keuangan | agregasi sisi-klien dari beberapa koleksi |

**Penting:** beberapa service menulis ke Firestore langsung dengan `runTransaction`
(`consignment-notes`, `customer-stocks`). Artinya logika konsistensi stok berjalan di
device — Security Rules harus memvalidasi, dan ada potensi divergensi dengan logika yang
sama di backoffice. Lihat [07 — Bug Register](07-bug-register.md).

### Struktur layar mobile (expo-router)

```
src/app/
├── _layout.tsx   ← root layout, auth gate, tema
├── login.tsx     ← login email/password
├── index.tsx     ← home / dashboard ringkas
├── customers.tsx ← daftar pelanggan + kunjungan
├── catalog.tsx   ← katalog produk
├── prospects.tsx ← daftar prospek scan area + update kunjungan
└── settings.tsx  ← pengaturan, tema, logout
```

## Firestore: model bersama

Kedua app berbagi koleksi Firestore yang sama. Sumber kebenaran skema:
`venovian-toys/docs/database-design.md`. Ringkasan: lihat [05 — Model Data](05-model-data.md).

Catatan keselarasan tipe: definisi tipe **didefinisikan ulang secara terpisah** di tiap
app (mis. `Customer`, `Product` di service mobile vs tipe backoffice). Tidak ada paket
tipe bersama. Ini sumber risiko drift skema — bila satu app menambah/ubah field, app lain
tidak ikut otomatis. Lihat [06 — Kontrak Integrasi](06-kontrak-integrasi.md).
