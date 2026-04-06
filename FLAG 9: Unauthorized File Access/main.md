# FLAG [9]
### LFI - Admin Actions Log Passphrase
Severity: HIGH

### Description
Apa passphrase yang tercatat di dalam file /var/log/nac/admin_actions.log?

### Hint
💡 Hint: LFI (Local File Inclusion)


### Analysis
Kami memulai analisis dengan melakukan enumerasi pada aplikasi web NAC NEWS untuk menemukan vektor serangan yang tersedia. Setelah mengamati pola URL pada halaman-halaman web, kami menemukan parameter 

Setelah mengamati pola URL pada halaman web, kami menemukan parameter ?page= yang digunakan untuk memuat konten halaman secara dinamis. Parameter ini mengindikasikan adanya potensi kerentanan Local File Inclusion (LFI), di mana penyerang dapat memanipulasi nilai parameter tersebut untuk membaca file-file sensitif di luar direktori web root.

Dengan menggunakan teknik path traversal (../../../../), kami berhasil menavigasi ke luar dari direktori web root dan mengakses file log admin di /var/log/nac/admin_actions.log. Di dalam file tersebut, kami menemukan backup encryption passphrase yang menjadi target flag ke-9.


### Solution
1. Identifikasi Parameter LFI
Kami mengamati URL pada halaman web dan menemukan parameter page yang memuat konten secara dinamis:

<img width="1072" height="52" alt="{24407B7D-CC5E-4A58-B886-4265E8DDC3D8}" src="https://github.com/user-attachments/assets/3da53481-e0c8-498c-8c00-642fa25458a9" />

2. Uji Coba Path Traversal
Kami melakukan uji coba dengan menggunakan teknik path traversal untuk mencoba mengakses file di luar web root. Dengan menambahkan karakter ../../../../ pada nilai parameter, kami menavigasi ke direktori sistem:

<img width="1919" height="946" alt="{F55EF36D-DB08-4E49-937B-BD0324501EB2}" src="https://github.com/user-attachments/assets/22042f5b-e2b7-4f3e-93ab-88631d50de2c" />

3. Akses File Log Admin
Setelah mengonfirmasi bahwa LFI berhasil, kami menargetkan file log admin actions sesuai dengan petunjuk soal:

http://[TARGET]:8070/page.php?page=../../../../var/log/nac/admin_actions.log


4. Ekstraksi Passphrase dari Log
File log berhasil terbaca dan menampilkan aktivitas admin secara lengkap. Di dalam log tersebut, kami menemukan baris yang mencatat penggunaan backup encryption passphrase:

[2025-01-03 10:14:07] [INFO] Backup encryption passphrase used: ptolemy_dynasty_305bc

<img width="1920" height="951" alt="{F78D7573-7BAC-4C5C-B6D1-461960EF2839}" src="https://github.com/user-attachments/assets/08783562-638c-413d-958e-5afa699b6490" />


5. Submit Flag
Flag berhasil ditemukan. Format jawaban menggunakan pola key | url parameter, di mana url parameter adalah query string lengkap beserta tanda ? yang digunakan untuk mengeksploitasi LFI:

🚩 FLAG 9 ANSWER:
ptolemy_dynasty_305bc | ?page=../../../../var/log/nac/admin_actions.log

<img width="382" height="148" alt="{C8C736A9-F277-44A2-919F-873325A72FA4}" src="https://github.com/user-attachments/assets/64cc44e7-7468-4bd1-870c-4090e88af88c" />

Vulnerability Assessment
Vulnerability	Local File Inclusion (LFI)
Severity	High
CVSS v4.0 Score	8.7 (High)
CVSS Vector	CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:N/VA:N/SC:N/SI:N/SA:N

Saran Rekomendasi Mitigasi
1.	Validasi dan Sanitasi Input: Terapkan whitelist validasi untuk parameter page. Hanya izinkan nama halaman yang sudah ditentukan sebelumnya, dan tolak semua input yang mengandung karakter path traversal seperti ../, ..\ atau URL encoding-nya.
2.	Nonaktifkan allow_url_include: Pastikan konfigurasi PHP (php.ini) menonaktifkan direktif allow_url_include = Off dan allow_url_fopen = Off untuk mencegah remote file inclusion.
3.	Batasi Akses File Sistem: Gunakan fungsi PHP seperti realpath() dan basename() untuk memvalidasi bahwa path file yang diakses berada di dalam direktori yang diizinkan. Terapkan open_basedir untuk membatasi akses PHP hanya ke direktori web root.
4.	Perkuat Izin File Log: File log sensitif seperti /var/log/nac/admin_actions.log seharusnya tidak dapat dibaca oleh proses web server. Atur permission file log dengan mode 640 (rw-r-----) dan pastikan hanya user/group yang berwenang yang dapat mengaksesnya.
5.	Hindari Penyimpanan Kredensial di Log: Jangan pernah mencatat passphrase, password, atau kredensial sensitif lainnya di dalam file log, meskipun dalam kondisi terenkripsi. Gunakan sistem manajemen rahasia (secret management) yang terpisah.

