# FLAG [4]: Articles Draft

### Description
Apa judul draft yang belum dipublikasikan di akun editor sarah?

### Hint
💡 Hint Flag 4: Brute Force

### Analysis
Untuk menyelesaikan tantangan ini, kami melakukan *Vulnerability Chaining* (penggabungan kerentanan). Berdasarkan *hint* yang mengarah pada *Brute Force*, kami memanfaatkan celah SQL Injection yang telah ditemukan sebelumnya untuk mengekstrak *hash password* milik editor dari *database*. 

Karena antarmuka *login* web rentan terhadap pemblokiran jika di-*brute-force* secara langsung, kami memilih jalur eksploitasi yang lebih sunyi: **Offline Hash Cracking**. Kami membuat *custom wordlist* spesifik menggunakan teknik OSINT (nama target, tahun, dan karakter spesial), lalu menggunakan Hashcat untuk memecahkan *hash* Bcrypt tersebut. Setelah mendapatkan kata sandi *plaintext*, kami masuk ke dalam *dashboard* editor dan menemukan informasi sensitif berupa draf artikel rahasia.

### Solution

**1. Ekstraksi Hash Password (via SQLi)**
Melalui titik injeksi SQL sebelumnya, kami menggunakan `sqlmap` untuk melakukan *dump* pada tabel `users` guna mendapatkan kredensial *login*.
```bash
sqlmap -u "[http://160.25.222.15:8070/search.php?q=test](http://160.25.222.15:8070/search.php?q=test)" \
--cookie="PHPSESSID=a64b2b04ca168600434f5d7f48a7cdb5" \
-D nac_news -T users -C id,username,email,password \
--dump --threads=10 --keep-alive -o --batch
```
![Scanning Tabel Users](scanning_sqlmap.png)

Dari hasil *dumping* tersebut, kami menemukan akun `sarah_editor` beserta *hash* Bcrypt-nya: `$2y$10$u2PyyCEmJGa9jf5UkLaG9uUBJENeHFGDkPeH2o5Oee6DrfUdg7BF6`.
![Hasil Scanning](hasil_scanning_sqlmap.png)

Kami kemudian menyimpan *hash* tersebut ke dalam sebuah *file* bernama `sarah_editor_hash.txt`.
![Hash Sarah](hash_sarah.png)

**2. Pembuatan Custom Wordlist**
Untuk mengoptimalkan proses *brute-force* yang memakan banyak *resource* CPU/GPU, kami meracik *custom wordlist* bernama `generate_wordlist.txt`. *Wordlist* ini di- *generate* berdasarkan parameter yang relevan dengan target, seperti `sarah`, `editor`, `nac`, dan `2024`.
![Membuat Wordlist](membuat_wordlist.png)
![Wordlist](wordlist.png)

**3. Offline Cracking menggunakan Hashcat**
Kami menjalankan **Hashcat** dengan mode algoritma Bcrypt (`-m 3200`) untuk menghantam *hash* tersebut menggunakan *wordlist* yang telah kami buat.
```bash
hashcat -m 3200 -a 0 sarah_editor_hash.txt generate_wordlist.txt
```
Dalam hitungan detik, Hashcat berhasil mengungkap kata sandi di balik *hash* tersebut, yaitu `sarah2024!`.
![Hashcat Cracking](hashcat_cracking.png)

**4. Autentikasi ke Dashboard Administrator**
Berbekal kredensial `sarah_editor`:`sarah2024!`, kami berhasil melakukan *login* melalui pintu depan aplikasi web di *endpoint* `/login.php`.
![Login UI](login_ui.png)

**5. Ekstraksi Artikel Rahasia**
Setelah masuk ke dalam *Dashboard*, kami memeriksa fitur manajemen konten dan menavigasi ke menu **Articles $\rightarrow$ Drafts**. Di sana, kami menemukan artikel rahasia (ID 13) yang belum dipublikasikan, yang merupakan target dari tantangan ini.
![Menemukan Hidden Articles](menemukan_hidden_articles.png)

**Bukti Flag Benar:**
`The Hidden Archive of Digital Artifacts`
![Bukti Flag](flag.jpeg)

### Vulnerability Assessment
* **Vulnerability:** Broken Authentication (Weak Password Policy) & Information Disclosure
* **Severity:** High (Penyerang dapat mengambil alih akun dengan hak akses tinggi)
* **CVSS v4.0 Score:** **7.1 (High)**
* **CVSS Vector:** `CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:L/VA:N/SC:N/SI:N/SA:N`

### Saran Rekomendasi Mitigasi
1. **Penerapan Kebijakan Sandi yang Kuat (Strong Password Policy)**
   Wajibkan seluruh pengguna, khususnya pengguna dengan *role* administratif, untuk menggunakan kata sandi dengan kompleksitas tinggi. Tolak secara otomatis kata sandi yang mudah ditebak, seperti kombinasi nama pengguna dan tahun berjalan.

2. **Tingkatkan Cost Factor pada Hashing Algoritma**
   Meskipun Bcrypt sudah cukup aman, *cost factor* dapat ditingkatkan secara berkala seiring perkembangan teknologi perangkat keras. Peningkatan nilai iterasi ini akan membuat serangan *offline brute-force* (seperti menggunakan Hashcat) menjadi sangat lambat dan tidak efisien.
