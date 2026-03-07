# FLAG [5]: Internal API Version

### Description
Berapa versi (version) dari internal API service yang tidak dapat diakses dari luar?

### Hint
💡 Hint Flag 5: Editor, SSRF

### Analysis
Tantangan ini membutuhkan eksekusi serangan *Server-Side Request Forgery* (SSRF). Kami memanfaatkan petunjuk yang didapat dari celah SQL Injection sebelumnya, di mana kami menemukan riwayat *payload* dari *database* yang mengarah ke *endpoint* internal `http://internal-api:5000/status`. Dengan menggunakan hak akses Editor yang didapat pada tantangan sebelumnya, kami mengeksploitasi fitur **URL Preview Tool** di dalam *dashboard* untuk mem-*bypass* *firewall* eksternal dan memaksa *server* web mengambil data rahasia dari *service* API internal tersebut.

### Solution

**1. Menemukan Target Internal (via SQLi Dump)**
Sebelumnya, kami melakukan *dumping* pada tabel `comments` menggunakan `sqlmap` untuk menganalisis riwayat injeksi.
```bash
sqlmap -u "[http://160.25.222.15:8070/search.php?q=test](http://160.25.222.15:8070/search.php?q=test)" \
--cookie="PHPSESSID=a64b2b04ca168600434f5d7f48a7cdb5" \
-D nac_news -T comments --dump --batch
```
![Scanning SQLMap Comment](scanning_sqlmap_comment.png)

Dari hasil *dump* tersebut, pada entri ID 22, kami menemukan rekam jejak *payload* berupa URL internal: `http://internal-api:5000/status`. Ini mengonfirmasi adanya *service* yang berjalan di *port* 5000 pada jaringan internal yang terisolasi.
![Hasil Scanning Database](hasil_scanning.png)

**2. Eksploitasi SSRF pada URL Preview**
Berbekal akun `sarah_editor`, kami masuk ke menu **URL Preview Tool** di *dashboard* admin. Fitur ini berfungsi untuk melakukan *fetch metadata* dari URL eksternal, namun ternyata aplikasi tidak memfilter *request* ke IP atau *domain* internal.
Kami memasukkan URL target yang ditemukan dari *database* ke dalam kolom masukan:
`http://internal-api:5000/status`

**3. Ekstraksi Versi API**
Setelah menekan tombol **Preview**, *server* NacNews meneruskan permintaan kita ke jaringan internal dan mengembalikan respons HTTP 200 OK dengan format `application/json`. Di dalam *Response Body* tersebut, kami menemukan versi dari API internal yang menjadi jawaban dari tantangan ini: `"version":"v3.7.2-ptolemy"`.
![SSRF URL Preview](url_preview.png)

**Bukti Flag Benar:**
`v3.7.2-ptolemy`

---

### Vulnerability Assessment
* **Vulnerability:** Server-Side Request Forgery (SSRF)
* **Severity:** High (Penyerang dapat mengakses dan memetakan layanan di jaringan internal yang terisolasi)
* **CVSS v4.0 Score:** **7.7 (High)**
* **CVSS Vector:** `CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:N/VA:N/SC:L/SI:N/SA:N`

### Saran Rekomendasi Mitigasi
1. **Validasi dan Filter Input URL (Allowlisting/Denylisting)**
   Terapkan validasi ketat pada input URL. Blokir secara eksplisit resolusi ke alamat IP privat/internal (seperti `127.0.0.1`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) dan *hostname* internal (seperti `localhost` atau nama kontainer internal).
2. **Network Segmentation**
   Isolasi *server* web yang menghadap publik (*public-facing*) dari layanan API internal di level jaringan. Pastikan aturan *firewall* melarang web *server* untuk mengakses *port* layanan internal secara bebas jika tidak diperlukan.
3. **Gunakan SSRF Protection Library**
   Gunakan pustaka keamanan khusus (*safe curl wrapper* atau setara) yang secara otomatis menangani proteksi SSRF, termasuk mencegah serangan *DNS Rebinding* saat melakukan *HTTP fetch* ke pihak ketiga.
