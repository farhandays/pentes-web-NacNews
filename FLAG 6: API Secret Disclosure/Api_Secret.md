

# FLAG [6]: API Secret Disclosure

### Description

Berapa nilai **API_SECRET** yang ditemukan setelah berhasil melewati *upload filter* dan membaca file kredensial di server?

### Hint

💡 Hint Flag 6: Upload File, Case Sensitive

---

## Analysis

Untuk menyelesaikan tantangan ini, kami mengeksploitasi kelemahan pada mekanisme **file upload validation**. Berdasarkan *hint*, kami menduga adanya celah pada pengecekan ekstensi file yang bersifat *case sensitive*.

Sistem hanya melakukan validasi sederhana terhadap ekstensi dan *Content-Type*, tanpa memverifikasi isi file secara menyeluruh. Hal ini membuka peluang untuk melakukan **Upload Filter Bypass**.

Strategi yang digunakan:

* Membuat file *web shell* sederhana berbasis PHP.
* Mengubah ekstensi menjadi kombinasi huruf besar-kecil (misalnya `.PhP`) untuk melewati filter.
* Memodifikasi header `Content-Type` menjadi `image/png`.
* Mengunggah ulang file menggunakan manipulasi request melalui Burp Suite.

Setelah file berhasil diunggah dan tersimpan di direktori `/uploads/`, kami dapat mengeksekusi perintah sistem menggunakan parameter `cmd` dan melakukan enumerasi direktori server.

Dari proses ini ditemukan folder sensitif yang berisi file kredensial internal, termasuk nilai **API_SECRET**.

---

## Solution

### 1. Pembuatan Backdoor Sederhana

Kami membuat file PHP sederhana yang dapat menjalankan perintah sistem melalui parameter URL.

Contoh akses setelah file berhasil diunggah:

```
http://target/uploads/upload_69a7bf050497e_PintuBelakang.PhP?cmd=ls
```

<img width="1135" height="582" alt="image" src="https://github.com/user-attachments/assets/a89ff481-4bf8-4dc9-93f2-19d592807d57" />


---

### 2. Analisis Request Menggunakan Burp Suite

Proses upload dianalisis menggunakan Burp Suite untuk melihat struktur *HTTP request* yang dikirim ke server.

Ditemukan bahwa:

* Validasi ekstensi tidak konsisten.
* Server hanya memeriksa `Content-Type`.
* Tidak ada pembatasan eksekusi file di folder `/uploads/`.

Kami kemudian:

* Mengubah ekstensi file menjadi `.PhP`
* Mengatur `Content-Type: image/png`
* Mengirim ulang request yang telah dimodifikasi
<img width="973" height="84" alt="image" src="https://github.com/user-attachments/assets/cdc800db-6e6b-4123-8dfc-07d9ba1da656" />
<img width="900" height="125" alt="image" src="https://github.com/user-attachments/assets/79bcb130-30eb-4663-b3a5-8dda0ee8362e" />
<img width="1069" height="117" alt="{F773B142-1B11-48BF-B283-106D875AFADC}" src="https://github.com/user-attachments/assets/8b56d0cb-635a-4b94-8970-98c5f7a4d6e0" />

---

### 3. Enumerasi Direktori Server

Setelah file dapat dieksekusi, kami melakukan enumerasi direktori menggunakan:

```
?cmd=ls ../../
```

Dari hasil tersebut ditemukan folder:

```
Secrets
```
<img width="120" height="70" alt="image" src="https://github.com/user-attachments/assets/ced04709-abf8-453b-801b-b871ab2b3d9e" />


Kemudian dilakukan eksplorasi lebih lanjut hingga menemukan file:

```
api_credentials.txt
```

<img width="120" height="70" alt="image" src="https://github.com/user-attachments/assets/18d43570-45a7-426b-b55c-7d170a9d7d25" />
<img width="591" height="256" alt="image" src="https://github.com/user-attachments/assets/420c51a6-7d37-46e4-b6e2-96f8eb135604" />

---

### 4. Flag Ditemukan

Nilai **API_SECRET** yang ditemukan dalam file tersebut adalah:

```
sk_live_pharos_9x7k2m
```

**Bukti Flag Benar:**
`sk_live_pharos_9x7k2m`
<img width="511" height="128" alt="image" src="https://github.com/user-attachments/assets/af42069b-7675-4897-b2ed-f0c55709c35f" />


---

## Vulnerability Assessment

* **Vulnerability:** Unrestricted File Upload & Remote Code Execution (RCE)
* **Severity:** Critical
* **Impact:** Penyerang dapat mengeksekusi perintah sistem, membaca file sensitif, dan memperoleh kredensial internal.
* **Root Cause:** Validasi ekstensi berbasis *case-sensitive* dan tidak adanya pembatasan eksekusi pada direktori upload.

---

## Saran Rekomendasi Mitigasi

1. **Gunakan Whitelist Extension Validation**
   Validasi ekstensi harus dilakukan setelah dinormalisasi ke lowercase.

2. **Validasi File Signature (Magic Bytes)**
   Jangan hanya mengandalkan ekstensi atau `Content-Type`.

3. **Nonaktifkan Eksekusi di Folder Upload**
   Konfigurasi server agar direktori `/uploads/` tidak dapat mengeksekusi skrip.

4. **Simpan File di Luar Web Root**
   Gunakan mekanisme file handler untuk menampilkan file.

5. **Implementasi Web Application Firewall (WAF)**
   Deteksi dan blokir pola upload mencurigakan.

