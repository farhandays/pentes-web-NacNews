# FLAG [8]: Decrypting Admin Secrets

### Description
Apa nilai m yang berhasil didekripsi dari tabel admin_secrets, menggunakan kunci enkripsi yang terdapat di site_settings?

### Hint
💡 Hint Flag 8: xor

### Analysis
Tantangan ini mengharuskan kami untuk mendekripsi sebuah nilai rahasia (message/m) yang disimpan di dalam *database*. Menggunakan kerentanan SQL Injection yang telah dikonfirmasi sebelumnya, kami mengekstrak kunci enkripsi dari tabel `site_settings` dan data *ciphertext* dari tabel `admin_secrets`. Berdasarkan *hint* yang diberikan dan analisis metode enkripsi "internal", kami mengidentifikasi bahwa aplikasi menggunakan algoritma kriptografi kustom berbasis **Repeating-Key XOR Cipher** yang digabungkan dengan **Base64**. Dengan membalikkan proses tersebut menggunakan kunci yang ditemukan, pesan asli berhasil didekripsi.

### Solution

**1. Ekstraksi Kunci Enkripsi (Encryption Key)**
Kami menggunakan `sqlmap` untuk melakukan *dumping* pada tabel `site_settings` guna mencari kunci enkripsi yang digunakan oleh sistem.
```bash
sqlmap -u "[http://160.25.222.15:8070/search.php?q=test](http://160.25.222.15:8070/search.php?q=test)" \
--cookie="PHPSESSID=a64b2b04ca168600434f5d7f48a7cdb5" \
-D nac_news -T site_settings --dump --batch
```
![Scanning Site Settings](scanning_sitesetting.png)

Dari hasil *dump* tersebut, kami menemukan entri `encryption_key` dengan nilai **`PTOLEMY`**. Kunci inilah yang akan digunakan untuk proses dekripsi.
![Hasil Site Settings](hasil_sitesetting.png)

**2. Ekstraksi Pesan Terenkripsi (Ciphertext)**
Selanjutnya, kami mengekstrak isi tabel `admin_secrets` menggunakan perintah SQLMap yang sama:
```bash
sqlmap -u "[http://160.25.222.15:8070/search.php?q=test](http://160.25.222.15:8070/search.php?q=test)" \
--cookie="PHPSESSID=a64b2b04ca168600434f5d7f48a7cdb5" \
-D nac_news -T admin_secrets --dump --batch
```
![Scanning Admin Secrets](scanning_adminsecret.png)

Tabel tersebut berisi 3 entri rahasia. Kami memfokuskan analisis pada `encrypted_value` dari `vault_access_code`, yaitu: `JDwqEyI/PDEgECAsLysxJjYTKygvNSYQLjA/NzUw`.
![Hasil Admin Secrets](hasil_adminsecret.png)

**3. Proses Dekripsi (Base64 & XOR)**
Kami menggunakan alat analisis kriptografi **CyberChef** untuk melakukan dekripsi dengan alur operasi (Recipe) sebagai berikut:
1. **From Base64**: Mengubah *ciphertext* kembali ke bentuk *byte array*.
2. **XOR**: Menggunakan skema *Standard* dengan parameter Key berupa *string* UTF8 **`PTOLEMY`**.

Setelah memasukkan *ciphertext* ke dalam input, CyberChef langsung memunculkan teks terang (*plaintext*) dari kode rahasia tersebut.
![Proses Dekripsi CyberChef](Decrypting.png)

**Bukti Flag Benar:**
`the_great_library_never_burned`
![Bukti Flag 8](image_874126.png)

---

### Vulnerability Assessment
* **Vulnerability:** Weak/Custom Cryptography & Information Disclosure via SQLi
* **Severity:** High (Data kredensial dan rahasia internal dapat dibaca dalam bentuk teks terang)
* **CVSS v4.0 Score:** **7.1 (High)**
* **CVSS Vector:** `CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:L/VA:N/SC:N/SI:N/SA:N`

### Saran Rekomendasi Mitigasi
1. **Gunakan Standar Enkripsi Industri (Avoid "Roll Your Own Crypto")**
   Jangan pernah menggunakan algoritma enkripsi buatan sendiri (seperti kombinasi Base64 dan XOR) untuk menyimpan data sensitif. XOR Cipher dengan kunci yang berulang sangat mudah dipecahkan. Gunakan standar enkripsi yang sudah teruji seperti **AES-256-GCM** dengan pustaka kriptografi bawaan bahasa pemrograman (misal: `openssl_encrypt` pada PHP).
2. **Pisahkan Kunci Enkripsi dari Database**
   Menyimpan kunci enkripsi di *database* yang sama dengan data yang dienkripsi (seperti di tabel `site_settings`) adalah praktik yang buruk. Jika *database* bocor (misal melalui SQLi), penyerang akan mendapatkan "gembok beserta kuncinya". Simpan kunci enkripsi di *Environment Variables* (`.env`), *Key Management Service* (KMS), atau *file system* dengan hak akses sangat ketat yang terpisah dari *database web*.
