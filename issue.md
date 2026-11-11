# Issue: Bangun Aplikasi Mini OMS F&B (Laravel 12 + Filament 5) — Showcase Keunggulan Laravel

## 1. Tujuan & Konteks Proyek
Bangun aplikasi **Mini OMS (Order & Inventory Management System)** bertema **Distribusi Produk Makanan & Minuman (F&B)** menggunakan **Laravel 12** dan **Filament 5** sebagai proyek showcase yang mendemonstrasikan keunggulan ekosistem Laravel secara nyata, end-to-end, dan terverifikasi oleh automated tests.

Aplikasi ini menggabungkan tiga antarmuka dalam satu arsitektur bersih:
1. **Filament Admin Panel (`/admin`)**: Antarmuka operasional utama (Dashboard analitik, CRUD relasional, live Repeater order form, Custom Actions, Global Search, Queue Export CSV, dan Database Notifications).
2. **REST API (`/api/v1`)**: API berautentikasi token (**Laravel Sanctum**) dengan validasi `FormRequest`, transformasi `API Resource`, dan otorisasi **Policy yang sama** dengan Filament.
3. **Halaman Panduan Keunggulan (`/`)**: Landing page interaktif berbahasa Indonesia yang menampilkan metrik database live, peta arsitektur keunggulan Laravel ke file kode sumber, kredensial demo `Admin` vs `Staff`, serta cheat-sheet pengujian REST API, Queue, dan Pest.

---

## 2. Batasan Lingkungan & Kompatibilitas (Wajib Dipatuhi)
- **OS & Shell:** Windows / PowerShell.
- **PHP Lokal:** `8.2.12` (ekstensi `pdo_sqlite`, `pdo_mysql`, `intl`, `zip`, `gd`, `mbstring`, `openssl`, `curl` aktif).
- **Kompatibilitas Versi Paket (PHP 8.2):**
  - `laravel/laravel:^12.0` *(JANGAN gunakan Laravel 13 karena butuh PHP 8.3+)*
  - `filament/filament:^5.0` *(mendukung PHP 8.2 & Laravel 12)*
  - `laravel/sanctum:^4.0`
  - `pestphp/pest:^3.8` dan `pestphp/pest-plugin-laravel:^3.0` *(JANGAN gunakan Pest 4/5 karena butuh PHP 8.3+/8.4+)*
- **Node.js & Asset Build:** Karena PowerShell memblokir `npm.ps1` via ExecutionPolicy, **selalu gunakan `npm.cmd`** (contoh: `npm.cmd install`, `npm.cmd run build`).
- **Database & Queue:** Gunakan **SQLite** (`database/database.sqlite`), `QUEUE_CONNECTION=database`, `MAIL_MAILER=log`, dan `APP_LOCALE=id` agar aplikasi berjalan mandiri tanpa dependensi MySQL atau Redis.
- **Konvensi Bahasa:**
  - **UI, Label Filament, Pesan User, Halaman `/`, dan `README.md`:** Bahasa Indonesia.
  - **Nama Class, Tabel, Kolom, Enum, Route API, dan Test:** Standar teknis Bahasa Inggris.

---

## 3. Bootstrap & Setup Awal
1. Periksa apakah `composer.json` dan kerangka Laravel 12 sudah ada di direktori kerja:
   - **Jika sudah ada:** pastikan versi `laravel/framework` adalah `^12.0` dan lanjutkan ke instalasi paket tambahan.
   - **Jika belum ada (direktori hanya berisi `issue.md`):** amankan `issue.md` sementara atau inisialisasi `composer create-project laravel/laravel:^12.0` di direktori sementara lalu pindahkan isinya tanpa menimpa `issue.md`.
2. Install paket inti & dev:
   - `composer require filament/filament:^5.0 laravel/sanctum:^4.0 --no-interaction`
   - `composer require --dev pestphp/pest:^3.8 pestphp/pest-plugin-laravel:^3.0 --with-all-dependencies --no-interaction`
3. Jalankan installer resmi:
   - `php artisan filament:install --panels --no-interaction`
   - `php artisan install:api --no-interaction`
   - Siapkan struktur Pest di `tests/Pest.php` (pastikan `RefreshDatabase` & `Tests\TestCase` terkonfigurasi untuk `Feature` tests).
4. Siapkan tabel pendukung Queue, Notifications, dan Filament Exports (pastikan migrasi `jobs`, `notifications`, dan `exports` tersedia melalui `php artisan make:notifications-table` / `php artisan filament:install` / migrasi standar Laravel 12 + `php artisan make:queue-batches-table` atau `php artisan vendor:publish` bila diperlukan oleh Filament ExportAction).
5. Install & build asset frontend via `npm.cmd install` dan `npm.cmd run build`.

---

## 4. Spesifikasi Domain, Database & Enums

### A. PHP Backed Enums (`app/Enums/`)
Buat Backed Enum string yang mengimplementasikan kontrak Filament (`HasLabel`, `HasColor`, `HasIcon` bila relevan) agar badge di tabel/infolist tampil konsisten:
1. `UserRole`: `Admin = 'admin'`, `Staff = 'staff'`
2. `CustomerType`: `B2B = 'b2b'`, `Retail = 'retail'`
3. `OrderStatus`: `Pending = 'pending'`, `Confirmed = 'confirmed'`, `Completed = 'completed'`, `Cancelled = 'cancelled'`
4. `StockMovementType`: `In = 'in'`, `Out = 'out'`, `Adjustment = 'adjustment'`

### B. Skema Tabel & Migrasi (`database/migrations/`)
1. **`users`** (modifikasi/migrasi tambah kolom):
   - Tambah kolom `role` (`string`, default `'staff'`).
2. **`categories`**:
   - `id`, `name` (`string`), `slug` (`string`, unique), `description` (`text`, nullable), `is_active` (`boolean`, default `true`), `timestamps`.
3. **`products`**:
   - `id`, `category_id` (`foreignId` -> `categories`, `restrictOnDelete`), `sku` (`string`, unique), `name` (`string`), `unit` (`string`, mis. `kg`, `dus`, `botol`, `pack`), `cost_price` (`decimal(12,2)`), `sell_price` (`decimal(12,2)`), `stock` (`integer`, default `0`), `min_stock` (`integer`, default `5`), `is_active` (`boolean`, default `true`), `timestamps`, `softDeletes`.
4. **`customers`**:
   - `id`, `name` (`string`), `type` (`string`, default `'retail'`), `phone` (`string`), `email` (`string`, nullable), `address` (`text`, nullable), `notes` (`text`, nullable), `timestamps`, `softDeletes`.
5. **`orders`**:
   - `id`, `order_number` (`string`, unique, format `ORD-YYYYMMDD-XXXX`), `customer_id` (`foreignId` -> `customers`, `restrictOnDelete`), `user_id` (`foreignId` -> `users`, `restrictOnDelete`), `status` (`string`, default `'pending'`), `total_amount` (`decimal(14,2)`, default `0`), `notes` (`text`, nullable), `confirmed_at` (`timestamp`, nullable), `completed_at` (`timestamp`, nullable), `cancelled_at` (`timestamp`, nullable), `timestamps`.
6. **`order_items`**:
   - `id`, `order_id` (`foreignId` -> `orders`, `cascadeOnDelete`), `product_id` (`foreignId` -> `products`, `restrictOnDelete`), `quantity` (`integer`), `unit_price` (`decimal(12,2)`), `subtotal` (`decimal(14,2)`), `timestamps`.
7. **`stock_movements`**:
   - `id`, `product_id` (`foreignId` -> `products`, `restrictOnDelete`), `order_id` (`foreignId` -> `orders`, nullable, `nullOnDelete`), `user_id` (`foreignId` -> `users`, nullable, `nullOnDelete`), `type` (`string`), `quantity_change` (`integer`, signed `+`/`-`), `stock_before` (`integer`), `stock_after` (`integer`), `reason` (`string`), `timestamps`.

### C. Eloquent Models (`app/Models/`)
- Implementasikan `HasFactory`, `SoftDeletes` (pada `Product` dan `Customer`), relasi lengkap (`belongsTo`, `hasMany`), `casts()` ke Enum & numeric/datetime, serta helper methods & query scopes:
  - `User`: implementasi `FilamentUser` (`canAccessPanel`), `HasApiTokens` (Sanctum), helper `isAdmin(): bool`, `isStaff(): bool`.
  - `Product`: scope `active()`, scope `lowStock()` (`whereColumn('stock', '<=', 'min_stock')`), accessor/helper `isLowStock()`.
  - `Order`: auto-generate `order_number` unik jika kosong saat creating, scope `byStatus(OrderStatus $status)`, scope `revenueGenerating()` (`Confirmed` & `Completed`).

---

## 5. Business Logic, Events, Queue Jobs & Console Command

### A. `App\Services\OrderService`
Semua mutasi status order dan pergerakan stok wajib lewat service ini agar konsisten antara **Filament** dan **REST API**:
1. **`createOrder(User $actor, int $customerId, array $items, ?string $notes = null): Order`**
   - Jalankan dalam `DB::transaction()`.
   - Ambil harga jual produk dari database (atau harga item valid jika diizinkan oleh alur admin, pastikan `unit_price >= 0` & `quantity >= 1`) dan hitung `subtotal` serta `total_amount` di server.
   - Simpan order dengan status `OrderStatus::Pending`.
2. **`updatePendingOrder(Order $order, int $customerId, array $items, ?string $notes = null): Order`**
   - Tolak dengan `ValidationException` jika status bukan `Pending`.
   - Sinkronkan `order_items` & hitung ulang `total_amount` dalam `DB::transaction()`.
3. **`confirmOrder(Order $order, User $actor): Order`**
   - Wajib dalam `DB::transaction()`.
   - Reload order dan lakukan validasi status harus `Pending`.
   - Agregasikan total kuantitas per `product_id` (mengantisipasi item produk duplikat dalam satu order), urutkan ID produk (`sort()`), lalu kunci baris produk dengan `Product::query()->whereIn('id', $productIds)->orderBy('id')->lockForUpdate()->get()->keyBy('id')` untuk mencegah race condition & deadlock.
   - Jika ada produk tidak aktif atau `stock < totalQuantity`, lempar `ValidationException` dengan pesan Bahasa Indonesia yang jelas (menyebut nama produk & sisa stok).
   - Kurangi `products.stock`, catat `StockMovement` bertipe `StockMovementType::Out` (`quantity_change` negatif, `stock_before`, `stock_after`, `order_id`, `user_id`), ubah status order ke `Confirmed` dan isi `confirmed_at = now()`.
   - Setelah transaksi berhasil (`DB::afterCommit` atau setelah `transaction()` selesai), dispatch event `OrderConfirmed($order)`.
4. **`completeOrder(Order $order, User $actor): Order`**
   - Validasi status harus `Confirmed`, ubah ke `Completed`, isi `completed_at = now()`.
5. **`cancelOrder(Order $order, User $actor, ?string $reason = null): Order`**
   - Tidak boleh membatalkan order yang sudah `Completed` atau `Cancelled` (lempar `ValidationException`).
   - Jika status `Pending`: ubah ke `Cancelled` (`cancelled_at = now()`) tanpa mengubah stok.
   - Jika status `Confirmed`: dalam `DB::transaction()` + `lockForUpdate()`, kembalikan stok seluruh item ke `products.stock`, catat `StockMovement` bertipe `StockMovementType::In` (`quantity_change` positif), ubah status ke `Cancelled` (`cancelled_at = now()`), dan dispatch event `OrderCancelled($order)`.
6. **`adjustStock(Product $product, User $actor, StockMovementType $type, int $quantity, string $reason): StockMovement`**
   - Gunakan `DB::transaction()` + `lockForUpdate()`.
   - Dukung penambahan stok (`In`, `$quantity > 0`) atau penyesuaian (`Adjustment`, bisa bertanda `+` atau `-` sesuai input modal).
   - Validasi `stock_after >= 0` (tolak stok negatif).
   - Simpan `StockMovement`, lalu jika `stock_after <= min_stock`, dispatch `NotifyLowStockJob`.

### B. Events, Listeners, Queue Jobs & Scheduler
1. **`App\Events\OrderConfirmed` & `App\Events\OrderCancelled`**
2. **`App\Listeners\DispatchLowStockCheckListener`**: Mendengarkan `OrderConfirmed` dan men-dispatch `NotifyLowStockJob` ke antrean (`database`).
3. **`App\Jobs\NotifyLowStockJob` (`implements ShouldQueue`)**:
   - Menerima array `$productIds` (atau opsional cek semua `Product::active()->lowStock()`).
   - Mengirim **Filament Database Notification** (`Notification::make()->title(...)->body(...)->warning()->sendToDatabase($adminUsers)`) kepada seluruh user ber-role `Admin`.
   - Mencatat ringkasan peringatan stok menipis ke Laravel log (`Log::warning(...)`).
4. **`App\Console\Commands\CheckLowStockCommand` (`oms:check-low-stock`)**:
   - Mengaudit produk aktif dengan `stock <= min_stock`, menampilkan tabel ringkas di CLI, dan men-dispatch `NotifyLowStockJob` bila ada opsi `--notify` atau secara default saat ditemukan stok menipis.
   - Daftarkan jadwal harian di `routes/console.php` (`Schedule::command('oms:check-low-stock')->dailyAt('08:00')`).

---

## 6. Otorisasi Native Laravel (`app/Policies/`)

Implementasikan Policy berikut menggunakan `UserRole` enum, dan pastikan didaftarkan/terdeteksi otomatis oleh Laravel serta dipatuhi oleh Filament dan REST API:

| Model / Aksi | `Admin` | `Staff` |
| :--- | :---: | :---: |
| `CategoryPolicy`: `viewAny`, `view` | Ya | Ya |
| `CategoryPolicy`: `create`, `update`, `delete` | Ya | Tidak |
| `ProductPolicy`: `viewAny`, `view` | Ya | Ya |
| `ProductPolicy`: `create`, `update`, `delete`, `restore`, `forceDelete`, `adjustStock` | Ya | Tidak |
| `CustomerPolicy`: `viewAny`, `view`, `create`, `update` | Ya | Ya |
| `CustomerPolicy`: `delete`, `restore`, `forceDelete` | Ya | Tidak |
| `OrderPolicy`: `viewAny`, `view`, `create`, `confirm`, `complete` | Ya | Ya |
| `OrderPolicy`: `update` | Ya (hanya jika `Pending`) | Ya (hanya jika `Pending`) |
| `OrderPolicy`: `cancel` | Ya (jika `Pending` atau `Confirmed`) | Ya (hanya jika `Pending`, tidak boleh batalkan `Confirmed`) |
| `OrderPolicy`: `delete` | Ya (hanya jika `Pending` atau `Cancelled`) | Tidak |
| `StockMovementPolicy`: `viewAny`, `view` | Ya | Ya |
| `StockMovementPolicy`: `create`, `update`, `delete` | Tidak (Audit trail immutable) | Tidak |

---

## 7. Spesifikasi Filament Admin Panel (`/admin`)

Konfigurasi di `AdminPanelProvider`:
- Path: `/admin`, aktifkan `->login()`, `->databaseNotifications()`, `->globalSearchKeyBindings(['command+k', 'ctrl+k'])`, brand name `"TBS Mini OMS"`.
- Kelompokkan navigasi dalam Bahasa Indonesia: **Penjualan**, **Master Data**, **Inventori**.

### A. Resources & Relation Managers
1. **`CategoryResource`**: Tabel + form sederhana (auto-slug dari `name`, hitung jumlah produk `products_count`).
2. **`ProductResource`**:
   - Global search: `sku`, `name`.
   - Filter: `category`, `is_active`, `low_stock` (custom filter), dan `TrashedFilter`.
   - Table Actions: **Sesuaikan Stok** (modal form memilih tipe `In`/`Adjustment`, jumlah, alasan -> panggil `OrderService::adjustStock`, otorisasi `adjustStock`), `Edit`, `Delete`, `Restore`.
   - Relation Manager: `StockMovementsRelationManager` (read-only kartu stok).
3. **`CustomerResource`**:
   - Global search: `name`, `phone`, `email`.
   - Filter: `type` (`B2B` vs `Retail`), `TrashedFilter`.
   - Relation Manager: `OrdersRelationManager`.
4. **`OrderResource`**:
   - Global search: `order_number`, `customer.name`.
   - **Form Interaktif**:
     - Pilih `customer_id` (searchable + modal create option inline).
     - `Repeater` untuk `items`: saat `product_id` dipilih (`->live()`), otomatis isi `unit_price` dari `sell_price` produk dan hitung `subtotal = quantity * unit_price`. Saat `quantity` atau `items` berubah (`->live()`), tampilkan ringkasan `Placeholder` / field `total_amount` secara real-time.
     - Catatan: Saat menyimpan di halaman Create/Edit Filament, delegasikan pembuatan/pembaruan item & kalkulasi total yang aman ke `OrderService` (atau pastikan server-side calculation konsisten).
   - **Infolist (ViewOrder)**: Tampilkan ringkasan transaksi, status badge, timestamp workflow, tabel item, dan catatan.
   - **Custom Actions (Table & View Header)** sesuai `OrderPolicy`:
     - `Konfirmasi Order` (tampil saat `Pending`, konfirmasi modal, tangkap `ValidationException` -> tampilkan `Notification::danger()` jika stok tidak cukup).
     - `Selesaikan Order` (tampil saat `Confirmed`).
     - `Batalkan Order` (tampil saat diperbolehkan oleh `OrderPolicy::cancel`).
   - **Header Export Action**: `ExportAction` menggunakan `App\Filament\Exports\OrderExporter` (berjalan lewat Queue + kirim notifikasi download di bell icon).
5. **`StockMovementResource`**:
   - Read-only (`canCreate() => false`), filter berdasarkan `type` dan `product_id`, menampilkan badge `+`/`-` yang jelas.

### B. Dashboard Widgets
1. **`OmsStatsOverview`**:
   - Omzet Bulan Ini (Rp format Indonesia, dari order `Confirmed` & `Completed`)
   - Order Pending (butuh konfirmasi)
   - Produk Aktif & Estimasi Nilai Inventori
   - Produk Stok Menipis (`stock <= min_stock`, warna danger/warning)
2. **`MonthlySalesChart`**: Line/Bar chart tren omzet harian/bulanan (minimal 14 hari atau 6 bulan terakhir dari data seeder).
3. **`LowStockProductsTable`**: Tabel produk dengan `stock <= min_stock` + quick action `Sesuaikan Stok` (hanya muncul untuk `Admin`).

---

## 8. Spesifikasi REST API (`/api/v1`)

Gunakan `routes/api.php` dengan prefix `v1`:
- **Publik (Rate-limited `throttle:10,1`):**
  - `POST /api/v1/login` -> validasi `LoginRequest` (`email`, `password`, optional `device_name`), kembalikan `{ token, user: { id, name, email, role } }`.
- **Terproteksi (`middleware('auth:sanctum')`):**
  - `POST /api/v1/logout` -> hapus token saat ini.
  - `GET /api/v1/me` -> profil user aktif.
  - `GET /api/v1/products` -> filter `?search=`, `?category_id=`, `?low_stock=1`, paginated (`ProductResource`), cek `ProductPolicy@viewAny`.
  - `GET /api/v1/customers` -> daftar pelanggan aktif (`CustomerResource`), cek `CustomerPolicy@viewAny`.
  - `GET /api/v1/orders` -> daftar order berpaginasi + filter `?status=`, eager load `customer` & `items.product` (`OrderResource`), cek `OrderPolicy@viewAny`.
  - `POST /api/v1/orders` -> validasi `StoreOrderRequest` (`customer_id`, `items.*.product_id`, `items.*.quantity`, `notes`), buat order `Pending` via `OrderService::createOrder()`, kembalikan `201 Created`, cek `OrderPolicy@create`.
  - `GET /api/v1/orders/{order}` -> detail order (`OrderResource`), cek `OrderPolicy@view`.
  - `POST /api/v1/orders/{order}/confirm` -> cek `OrderPolicy@confirm`, jalankan `OrderService::confirmOrder()`, kembalikan `OrderResource` (atau `422` jika stok kurang / status tidak valid).
  - `POST /api/v1/orders/{order}/cancel` -> cek `OrderPolicy@cancel`, jalankan `OrderService::cancelOrder()`, kembalikan `OrderResource` (atau `403` jika `Staff` mencoba membatalkan order `Confirmed`).

---

## 9. Halaman Panduan Keunggulan (`/`) & Seeder Data F&B

### A. Halaman Showcase (`/` via `ShowcaseController`)
Bangun halaman `resources/views/welcome.blade.php` yang rapi, modern, dan responsif menggunakan Tailwind CSS (dikompilasi via Vite):
- **Hero + Navigasi Cepat:** Tombol menuju `/admin` (Filament Admin Panel) dan ringkasan arsitektur aplikasi.
- **Live Database Snapshot:** Metrik real-time dari SQLite (Total Produk Aktif, Stok Menipis, Order Pending, Total Omzet Terkonfirmasi, Pending Queue Jobs).
- **Peta Keunggulan Laravel & Filament:** Kartu interaktif/terstruktur untuk setiap keunggulan (Filament UI, Eloquent & Scopes, `OrderService` + `lockForUpdate`, Queue & Database Notifications, Native Policy, Sanctum API, Pest Testing) lengkap dengan nama file kode sumber terkait.
- **Panel Uji Akun Demo (`Admin` vs `Staff`):** Kredensial siap salin (`admin@tbsoms.test` dan `staff@tbsoms.test`, password `password`) serta ringkasan apa yang membedakan hak akses keduanya.
- **Cheat Sheet Uji Coba:** Contoh perintah PowerShell (`Invoke-RestMethod` / `curl.exe`) untuk mengetes API Sanctum, perintah `php artisan queue:work`, `php artisan oms:check-low-stock`, dan `php artisan test`.

### B. Seeder Realistis F&B (`DatabaseSeeder`)
Pastikan `php artisan migrate:fresh --seed` bersifat idempoten/bersih dan menghasilkan:
- **2 User Demo:** `Admin` (`admin@tbsoms.test` / `password`) dan `Staff` (`staff@tbsoms.test` / `password`).
- **5 Kategori F&B:** *Biji Kopi & Teh*, *Produk Susu & Krimer*, *Sirup & Topping*, *Makanan Ringan & Bakery*, *Kemasan & Perlengkapan*.
- **14 Produk F&B** dengan harga modal/jual Rupiah realistis dan beberapa SKU sengaja memiliki `stock <= min_stock` (mis. *Kopi Arabika Gayo 1kg*, *Susu UHT Full Cream 1L*, *Sirup Karamel 750ml*, *Cup Sealer 16oz*, *Bubuk Matcha Premium 500g*, dll.).
- **6 Pelanggan F&B** (kombinasi B2B outlet seperti *Kopi Senja Kemang*, *Cafe Ruang Teduh*, *Resto Nusantara* dan pelanggan Retail).
- **10 Order Sample** lintas status (`Pending`, `Confirmed`, `Completed`, `Cancelled`) yang dibuat melalui `OrderService` atau dengan histori `StockMovement` yang 100% konsisten terhadap stok produk saat ini.

---

## 10. Automated Testing (Pest PHP v3) & Dokumentasi

### A. Suite Pengujian (`tests/`)
1. **`tests/Unit/OrderServiceTest.php`**:
   - `createOrder` menghitung subtotal & total di server dengan status `Pending` tanpa mengubah stok.
   - `confirmOrder` memotong stok, mencatat `StockMovement::Out` dengan `stock_before`/`stock_after` akurat, dan mengagregasi item produk yang sama dalam satu order.
   - `confirmOrder` gagal secara atomik (`ValidationException`) bila stok salah satu item tidak mencukupi (stok item lain tidak ikut berkurang).
   - `cancelOrder` pada order `Confirmed` mengembalikan stok dan mencatat `StockMovement::In`.
   - `cancelOrder` pada order `Pending` tidak mengubah stok.
   - `adjustStock` menolak pengurangan yang membuat stok negatif.
2. **`tests/Feature/PolicyAuthorizationTest.php`**:
   - Memverifikasi seluruh matriks hak akses `Admin` vs `Staff` (`ProductPolicy`, `CategoryPolicy`, `CustomerPolicy`, `OrderPolicy`, `StockMovementPolicy`).
3. **`tests/Feature/OrderWorkflowTest.php`**:
   - Mengonfirmasi order yang menurunkan stok hingga `<= min_stock` memicu `OrderConfirmed`, menjalankan `NotifyLowStockJob`, dan menyimpan notifikasi database untuk user `Admin`.
   - Menjalankan `php artisan oms:check-low-stock` sukses (exit code 0) dan mendeteksi produk stok menipis.
4. **`tests/Feature/Api/OrderApiTest.php`**:
   - Login Sanctum berhasil mendapatkan token & `GET /api/v1/me` sesuai role.
   - Request tanpa token ditolak `401`.
   - Buat order (`POST /api/v1/orders`) -> konfirmasi (`POST /api/v1/orders/{order}/confirm`) memotong stok.
   - `Staff` yang mencoba `POST /api/v1/orders/{order}/cancel` pada order `Confirmed` ditolak `403`.
   - Validasi input salah mengembalikan `422`.
5. **`tests/Feature/Filament/AdminPanelSmokeTest.php`**:
   - `Admin` dan `Staff` dapat mengakses halaman `/` dan `/admin`.
   - Smoke test Livewire untuk `ProductResource` & `OrderResource` (render list, create form, tombol aksi sesuai role).

### B. `README.md`
Buat `README.md` ringkas dalam Bahasa Indonesia berisi penjelasan proyek, arsitektur keunggulan Laravel yang didemonstrasikan, kredensial demo, serta cara menjalankan server (`php artisan serve`), queue worker (`php artisan queue:work`), dan test (`php artisan test`).

---

## 11. Definition of Done & Verifikasi Wajib
Agent AI dinyatakan selesai **hanya setelah** semua perintah verifikasi berikut dijalankan setelah perubahan kode terakhir dan seluruhnya lulus tanpa error:
- [ ] `npm.cmd run build` berhasil membangun asset Vite tanpa error.
- [ ] `php artisan migrate:fresh --seed` berjalan bersih dan menghasilkan data F&B yang konsisten.
- [ ] `php artisan test` lulus 100% seluruh test Pest (tanpa skipped atau relaxed assertions).
- [ ] `php artisan oms:check-low-stock` berjalan normal dan menampilkan daftar produk stok menipis.
- [ ] `php artisan queue:work --stop-when-empty` memproses job antrean tanpa masuk ke `failed_jobs`.
- [ ] `php artisan route:list` memastikan route `/`, `/admin`, dan `/api/v1/*` terdaftar lengkap.

