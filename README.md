☁️ Panduan Deployment Cloudflare Worker

Automated · Scalable · Anti-Timeout 🚀

Dokumen ini adalah panduan lengkap untuk melakukan deployment Cloudflare Worker menggunakan GitHub Actions.
Tersedia dua strategi deployment—pilih sesuai kebutuhan dan skala proyek Anda!

🎯 Tujuan Utama

Mengotomatisasi:

Pembuatan rute domain

Deployment Cloudflare Worker

Penanganan skala besar tanpa risiko API Timeout (504)

🤯 Kendala Umum: Cloudflare API Timeout (504)

Saat Worker memiliki banyak rute (domain + subdomain), Cloudflare API sering gagal memproses semua rute sekaligus → menyebabkan timeout.

Solusinya? Kita punya dua pendekatan:

🧩 Perbandingan Strategi
Strategi	Deskripsi	Kapan Digunakan
A. Single Worker (Legacy)	Semua rute masuk ke satu Worker 💥	Rute sangat sedikit (<50). Risiko timeout cukup tinggi.
B. Multi-Domain Chunking	Worker dideploy ke banyak domain menggunakan chunk ✨	Direkomendasikan! Skalabilitas tinggi dan aman dari timeout.
🛠️ File Wajib di Repository
File	Deskripsi	Digunakan oleh
worker.js	Script Cloudflare Worker	A & B
domains/list.txt	Daftar domain	B
.github/workflows/main.yml	Single Worker Deployment	A
.github/workflows/multi.yml	Multi-Domain Deployment	B
⚙️ Strategi A — Single Worker Deployment

File: main.yml

Workflow untuk mendeploy 1 Worker → 1 Domain.

📝 Input yang Dibutuhkan

script_name (misal worker.js)

route_pattern (misal example.com/*)

cloudflare_api_token

cloudflare_account_id

cloudflare_zone_id

📄 File: main.yml
name: Deploy Injektor

on:
  workflow_dispatch:
    inputs:
      script_name:
        description: "Nama file script (misal: worker.js)"
        required: true
      route_pattern:
        description: "Pattern route (misal: example.com/*)"
        required: true
      cloudflare_api_token:
        description: "Cloudflare API Token"
        required: true
      cloudflare_account_id:
        description: "Cloudflare Account ID"
        required: true
      cloudflare_zone_id:
        description: "Cloudflare Zone ID"
        required: true

jobs:
  deploy:
    name: Deploy Worker
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Upload Worker ke Cloudflare
        run: |
          curl -X PUT "https://api.cloudflare.com/client/v4/accounts/${{ inputs.cloudflare_account_id }}/workers/scripts/${{ inputs.script_name }}" \
            -H "Authorization: Bearer ${{ inputs.cloudflare_api_token }}" \
            -H "Content-Type: application/javascript" \
            --data-binary @"${{ inputs.script_name }}"

      - name: Tambah Route
        run: |
          curl -X POST "https://api.cloudflare.com/client/v4/zones/${{ inputs.cloudflare_zone_id }}/workers/routes" \
            -H "Authorization: Bearer ${{ inputs.cloudflare_api_token }}" \
            -H "Content-Type: application/json" \
            --data "{
              \"pattern\": \"${{ inputs.route_pattern }}\",
              \"script\": \"${{ inputs.script_name }}\"
            }"

🚀 Strategi B — Multi-Domain Chunking

File: multi.yml

Deployment otomatis ke banyak domain tanpa risiko timeout dengan membagi domain menjadi beberapa chunk.

📝 Input yang Dibutuhkan

script_name

cloudflare_api_token

cloudflare_account_id

chunk_size (default 20 domain per chunk)

📄 Struktur Folder
domains/
  list.txt

📄 File: multi.yml
name: Deploy Chunked Multi-Domain Worker

on:
  workflow_dispatch:
    inputs:
      script_name:
        description: "Nama file script (misal: worker.js)"
        required: true
      cloudflare_api_token:
        description: "Cloudflare API Token"
        required: true
      cloudflare_account_id:
        description: "Cloudflare Account ID"
        required: true
      chunk_size:
        description: "Jumlah domain per chunk (default 20)"
        required: false
        default: "20"

jobs:
  prepare:
    name: Persiapan Chunking
    runs-on: ubuntu-latest

    outputs:
      chunk_count: ${{ steps.chunk.outputs.count }}

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Buat Folder Chunk
        run: mkdir chunks

      - name: Chunking Domain
        id: chunk
        run: |
          split -l ${{ inputs.chunk_size }} domains/list.txt chunks/domain_chunk_
          COUNT=$(ls chunks | wc -l)
          echo "count=$COUNT" >> $GITHUB_OUTPUT

  deploy:
    name: Deploy Setiap Chunk
    needs: prepare
    runs-on: ubuntu-latest
    strategy:
      matrix:
        chunk_id: ${{ range(0, needs.prepare.outputs.chunk_count) }}

    steps:
      - uses: actions/checkout@v3

      - name: Deploy ke Cloudflare
        run: |
          CHUNK_FILE=$(ls chunks | sed -n "$(( ${{ matrix.chunk_id }} + 1 ))p")
          echo "Processing chunk: $CHUNK_FILE"

          while read DOMAIN; do
            echo "Deploy to $DOMAIN"
            curl -X POST "https://api.cloudflare.com/client/v4/accounts/${{ inputs.cloudflare_account_id }}/workers/scripts/${{ inputs.script_name }}/subdomain" \
              -H "Authorization: Bearer ${{ inputs.cloudflare_api_token }}" \
              -H "Content-Type: application/json" \
              --data "{\"hostname\": \"$DOMAIN\"}"
          done < "chunks/$CHUNK_FILE"

      - name: Selesai
        run: echo "Chunk ${{ matrix.chunk_id }} selesai dideploy"

  finish:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Semua Deployment Selesai
        run: echo "Semua deployment selesai."

🏃 Cara Menjalankan Deployment

Buka tab Actions di GitHub

Pilih workflow sesuai kebutuhan:

Deploy Injektor → Single domain

Deploy Chunked Multi-Domain Worker → Multi-domain

Klik Run workflow

Masukkan input yang diperlukan

Klik Run

Selesai 🎉 Worker sukses dideploy!
