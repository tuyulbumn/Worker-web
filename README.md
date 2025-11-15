# ☁️ Panduan Deployment Cloudflare Worker  
**Automated · Scalable · Anti-Timeout 🚀**

Dokumen ini adalah panduan lengkap untuk melakukan deployment Cloudflare Worker menggunakan GitHub Actions.  
Tersedia **dua strategi deployment**—pilih sesuai kebutuhan dan skala proyek Anda!

---

## 🎯 Tujuan Utama
Mengotomatisasi:

- Pembuatan rute domain  
- Deployment Cloudflare Worker  
- Penanganan skala besar tanpa risiko *API Timeout (504)*

---

## 🤯 Kendala Umum: *Cloudflare API Timeout (504)*
Saat Worker memiliki **banyak rute** (domain + subdomain), Cloudflare API sering gagal memproses semua rute sekaligus → menyebabkan *timeout*.

Solusinya? Kita punya **dua pendekatan**:

---

## 🧩 Perbandingan Strategi

| Strategi | Deskripsi | Kapan Digunakan |
|---------|-----------|------------------|
| **A. Single Worker (Legacy)** | Semua rute masuk ke **satu Worker** 💥 | Rute **sangat sedikit (<50)**. Risiko timeout cukup tinggi. |
| **B. Multi-Worker Sharding** | **1 Domain = 1 Worker unik** ✨ | **Direkomendasikan!** Skalabilitas tinggi dan aman dari timeout. |

---

## 🛠️ Persiapan File Wajib  
Pastikan file berikut ada di **root** repo:

| File | Deskripsi | Untuk Strategi |
|------|-----------|----------------|
| `worker.js` | Kode utama Worker | A & B |
| `customdomain.txt` | Daftar prefix subdomain (ex: `api`, `blog`) | A & B |
| `main_domains.txt` | Daftar domain utama | B |
| `deploy_chunked.yml` | Workflow sharding | B |
| `[Deploy Injektor].yml` | Workflow legacy | A |

---

# ⚙️ Strategi A — **Single Worker (Legacy Deployment)**  
**File:** `[Deploy Injektor].yml`

Pendekatan tradisional untuk proyek kecil tanpa banyak perkembangan domain.

### 📝 Input yang Dibutuhkan
- `worker_name` → Nama Worker  
- `main_domain` → Domain utama (ex: `nzr2805.my.id`)  
- `cloudflare_account_id` / `cloudflare_api_token`

### 🔄 Alur Kerja
1. Workflow membaca `main_domain` + semua prefix dalam `customdomain.txt`  
2. Semua rute digabung dalam **satu `wrangler.toml`**  
3. Satu Worker dideploy dengan seluruh rute tersebut  

Cocok jika jumlah rute sangat terbatas.

---

# 🚀 Strategi B — **Multi-Worker Sharding (Highly Recommended!)**  
**File:** `deploy_chunked.yml`

Solusi modern untuk menghindari timeout dan menangani banyak domain.

### 📝 Input yang Dibutuhkan
Hanya kredensial:

- `cloudflare_account_id`  
- `cloudflare_api_token`

Tidak perlu memasukkan nama worker atau domain — **semuanya otomatis!**

### 🔮 Logika Otomatis
| Langkah | Deskripsi | Tujuan |
|--------|-----------|---------|
| **1. Chunking** | Setiap domain di `main_domains.txt` dipecah menjadi 1 domain per proses | Mengurangi beban API |
| **2. Penamaan Otomatis** | `blueivy.qzz.io` → Worker bernama `blueivy` | Worker unik per domain |
| **3. Serial Deployment** | Deploy satu per satu, tidak paralel | Mencegah konflik API |
| **4. Cooldown 20 detik** | `sleep 20` antar deployment | Hindari 504 Timeout |

Dengan strategi ini, Anda bisa mendeploy bahkan **ratusan domain** secara stabil.

---

# 🏃 Cara Menjalankan Deployment

1. Buka tab **Actions** di GitHub repo Anda  
2. Pilih workflow:
   - **[Deploy Injektor]** → Strategi A  
   - **Deploy Chunked Multi-Domain Worker** → Strategi B  
3. Klik **“Run workflow”**  
4. Masukkan kredensial Cloudflare  
5. Tekan **Run** → Deployment berjalan otomatis 🎉

---

