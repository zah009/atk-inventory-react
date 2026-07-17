# Sistem Inventaris ATK — Frontend (React + Vite)

Frontend untuk **Sistem Informasi Pengadaan & Pengolahan ATK**: mengelola data barang, supplier, barang masuk/keluar, permintaan barang antar-divisi (dengan alur persetujuan), pengadaan, dan pembayaran. Dibangun dengan **React 19 + Vite + Tailwind CSS**, terhubung ke backend REST API [**SistemInventory_ATK_API_ASPdotNet**](https://github.com/zah009/SistemInventory_ATK_API_ASPdotNet) (ASP.NET Core).

---

## Daftar Isi

1. [Tampilan Aplikasi](#1-tampilan-aplikasi)
2. [Fitur Utama](#2-fitur-utama)
3. [Teknologi yang Digunakan](#3-teknologi-yang-digunakan)
4. [Arsitektur & Alur Data](#4-arsitektur--alur-data)
5. [Routing & Role Akses](#5-routing--role-akses)
6. [Instalasi](#6-instalasi)
7. [Konfigurasi Environment](#7-konfigurasi-environment)
8. [Menjalankan Aplikasi](#8-menjalankan-aplikasi)
9. [Menjalankan Bersama Backend (Full Stack)](#9-menjalankan-bersama-backend-full-stack)
10. [Struktur Folder](#10-struktur-folder)
11. [Deployment (GitHub Pages)](#11-deployment-github-pages)

---

## 1. Tampilan Aplikasi

| Login | Dashboard Admin |
|---|---|
| ![Login](./public/Login.png) | ![Dashboard Admin](./public/DashboardAdmin.png) |

| Dashboard Divisi |
|---|
| ![Dashboard Divisi](./public/DashboardDivisi.png) |

---

## 2. Fitur Utama

**Login**
- Autentikasi Admin & Divisi ke API (`/api/Auth/login`)
- Token JWT disimpan di `localStorage`, dikirim otomatis lewat header `Authorization: Bearer` di setiap request
- Redirect otomatis sesuai role setelah login (`/admin` atau `/divisi`), lewat `ProtectedRoute`

**Untuk Admin** (`/admin/*`)
- **Dashboard** — ringkasan data dari `AdminDashboard` API
- **Manajemen Barang** — tambah (bulk), edit, hapus barang, tabel interaktif
- **Supplier** — CRUD data supplier
- **Barang Masuk** — input penerimaan barang dari supplier (modal create/edit)
- **Barang Keluar** — riwayat barang keluar hasil approval, dengan detail per transaksi
- **Permintaan** — meninjau & memproses (approve/reject) permintaan barang dari divisi, lengkap modal detail
- **Pengadaan** — input pengajuan pengadaan barang ke supplier (modal create/edit)
- **Payment** — kelola pembayaran ke supplier, detail transaksi
- **User Divisi** — kelola akun user per divisi

**Untuk Divisi** (`/divisi/*`)
- **Dashboard** ringkas
- **Ajukan Permintaan Barang** — form pengajuan baru
- **Status Permintaan** — memantau status permintaan yang sudah diajukan
- **Lihat Stok** — melihat stok barang yang tersedia

**Komponen UI Reusable** (`src/components`) — `Button`, `Input`, `Select`, `Table`, `Modal`, `ConfirmModal`, `Card`, `Badge`, `Alert`, `Loading`, `Sidebar`, dipakai konsisten di seluruh halaman.

---

## 3. Teknologi yang Digunakan

| Komponen | Teknologi |
|---|---|
| Library UI | React 19 |
| Build Tool | Vite 7 |
| Routing | React Router DOM 7 |
| Styling | Tailwind CSS 3 |
| Icon | react-icons |
| HTTP Client | `fetch` native (helper `apiCall` di `src/services/api.jsx`), **bukan** Axios |
| Linting | ESLint 9 |
| Deployment | GitHub Pages (`gh-pages`) |

> Catatan: README versi sebelumnya menyebut Axios sebagai HTTP client — setelah dicek `src/services/api.jsx` dan `package.json`, project ini sebenarnya memakai `fetch` bawaan browser lewat helper `apiCall`, tanpa dependency Axios.

---

## 4. Arsitektur & Alur Data

```
Komponen Halaman (src/pages)
        |
        v
Service Layer (src/services/api.jsx) --> fetch() ke Backend API
        |
        v
Backend ASP.NET Core API (default http://localhost:8080)
```

Semua pemanggilan API terpusat di satu file `src/services/api.jsx`, dikelompokkan per modul (`authAPI`, `barangAPI`, `supplierAPI`, `barangMasukAPI`, `barangKeluarAPI`, `permintaanBarangAPI`, `pengadaanAPI`, `paymentAPI`, `divisiAPI`, `userDivisiAPI`, `dashboardAPI`) — satu-satu berkorespondensi langsung dengan controller di backend.

Helper `apiCall()`:
- Otomatis menambahkan base URL (`VITE_API_URL`)
- Otomatis menyisipkan header `Authorization: Bearer {token}` dari `localStorage`
- Menangani response non-JSON (mis. `204 No Content`)
- Melempar error dengan pesan dari body response bila status bukan 2xx

Konstanta enum status (`StatusPermintaan`, `PaymentStatus`) disamakan nilainya dengan enum di backend (`Pending: 0`, `Disetujui/Lunas: 1`, `Ditolak: 2`) supaya konsisten saat mengirim/membaca data.

---

## 5. Routing & Role Akses

Routing diatur di `src/App.jsx` menggunakan React Router, dengan proteksi lewat komponen `ProtectedRoute`:

| Path | Role | Halaman |
|---|---|---|
| `/login` | Publik | Login |
| `/admin` | Admin | Dashboard Admin |
| `/admin/barang` | Admin | Manajemen Barang |
| `/admin/supplier` | Admin | Manajemen Supplier |
| `/admin/barang-masuk` | Admin | Barang Masuk |
| `/admin/barang-keluar` | Admin | Barang Keluar |
| `/admin/permintaan` | Admin | Approval Permintaan |
| `/admin/pengadaan` | Admin | Pengadaan Barang |
| `/admin/payment` | Admin | Payment |
| `/admin/user-divisi` | Admin | Manajemen User Divisi |
| `/divisi` | Divisi | Dashboard Divisi |
| `/divisi/permintaan/create` | Divisi | Ajukan Permintaan |
| `/divisi/permintaan/status` | Divisi | Status Permintaan |
| `/divisi/stok` | Divisi | Lihat Stok |
| `*` | — | Halaman 404 |

Role ditentukan dari data `user` (hasil login) yang disimpan di `localStorage`. Jika role tidak sesuai dengan `requiredRole` pada route, `ProtectedRoute` akan mengalihkan pengguna.

---

## 6. Instalasi

### Prasyarat

- **Node.js** (disarankan versi LTS terbaru, ≥ 18) & npm
- Backend API sudah berjalan (lihat [SistemInventory_ATK_API_ASPdotNet](https://github.com/zah009/SistemInventory_ATK_API_ASPdotNet)) — frontend ini tidak bisa dipakai sendirian tanpa API

### Langkah

```bash
git clone https://github.com/zah009/atk-inventory-react.git
cd atk-inventory-react
npm install
```

---

## 7. Konfigurasi Environment

Base URL API diatur lewat environment variable `VITE_API_URL`. Buat file `.env` di root project:

```env
VITE_API_URL=http://localhost:8080
```

- Jika backend dijalankan lewat **Docker Compose** → default port `8080`, tidak perlu diubah.
- Jika backend dijalankan **manual** (`dotnet run`) → biasanya port `5034`, sesuaikan:
  ```env
  VITE_API_URL=http://localhost:5034
  ```
- Jika `.env` tidak dibuat, aplikasi otomatis fallback ke `http://localhost:8080` (lihat `src/services/api.jsx`).

---

## 8. Menjalankan Aplikasi

**Mode development:**
```bash
npm run go
```
Aplikasi berjalan di `http://localhost:3000` (port ini sudah dikonfigurasi tetap di `vite.config.js`, dan harus sama dengan origin yang diizinkan CORS di backend).

**Build production:**
```bash
npm run build
```

**Preview hasil build:**
```bash
npm run preview
```

**Lint kode:**
```bash
npm run lint
```

---

## 9. Menjalankan Bersama Backend (Full Stack)

```bash
# 1. Jalankan backend (API + SQL Server) via Docker Compose
git clone https://github.com/zah009/SistemInventory_ATK_API_ASPdotNet.git
cd SistemInventory_ATK_API_ASPdotNet
docker compose up --build
cd Atk && dotnet ef database update && cd ..
# API tersedia di http://localhost:8080

# 2. Jalankan frontend ini
cd ..
git clone https://github.com/zah009/atk-inventory-react.git
cd atk-inventory-react
npm install
npm run go
# Frontend tersedia di http://localhost:3000
```

Buka `http://localhost:3000` — frontend otomatis memanggil API di `http://localhost:8080`.

---

## 10. Struktur Folder

```
atk-inventory-react/
├── public/                      # Asset statis + screenshot aplikasi
├── src/
│   ├── components/              # Komponen UI reusable (Button, Table, Modal, dst)
│   ├── layouts/
│   │   └── MainLayout.jsx       # Layout utama (sidebar + konten) untuk halaman ter-autentikasi
│   ├── pages/
│   │   ├── Login/
│   │   ├── Admin/                # Halaman & modal khusus role Admin, per modul
│   │   └── Divisi/               # Halaman khusus role Divisi
│   ├── services/
│   │   └── api.jsx               # Semua pemanggilan API terpusat di sini
│   ├── App.jsx                   # Routing utama + logika autentikasi/proteksi role
│   └── main.jsx                  # Entry point React
├── vite.config.js                # Konfigurasi Vite (port dev server: 3000)
└── package.json
```

---

## 11. Deployment (GitHub Pages)

Project sudah dikonfigurasi untuk deploy ke GitHub Pages lewat `gh-pages`:

```bash
npm run deploy
```

Ini akan menjalankan `npm run build` lalu mem-publish folder `dist/` ke branch `gh-pages`. URL publikasi mengikuti field `homepage` di `package.json`:

```
https://zah009.github.io/atk-inventory-react/
```

> **Catatan penting:** karena frontend statis di GitHub Pages tidak bisa memanggil `http://localhost:8080`, `VITE_API_URL` perlu diarahkan ke URL backend yang sudah di-deploy publik (mis. instance AWS/EC2/cloud lain) sebelum menjalankan `npm run build` untuk production, dan pastikan origin GitHub Pages tersebut ditambahkan ke kebijakan CORS backend (`AllowReactApp` di `Program.cs`).