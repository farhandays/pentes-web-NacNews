# FLAG [3]: Hidden Service Account Recovery Email

### Description
Apa recovery email dari service account yang tersembunyi di website ini?

### Hint
💡 Hint: IDOR, Out of box

### Analysis
Untuk menyelesaikan tantangan ini, kami memutuskan untuk melakukan *Vulnerability Chaining* (menggabungkan dua celah keamanan). Daripada menebak angka ID secara acak (*blind fuzzing*), kami memanfaatkan celah SQL Injection yang sudah kami temukan sebelumnya pada `search.php?q=` untuk mengekstrak informasi dari tabel `users` di database.

Dari hasil ekstraksi data tersebut, kami menemukan sebuah *Service Account* dengan *username* `sysbot` yang terdaftar pada ID `13`. Meskipun *link* menuju profil `sysbot` ini disembunyikan dari antarmuka web publik, kami menyadari bahwa fitur profil pengguna di *endpoint* `profile.php?id=` rentan terhadap *Insecure Direct Object Reference* (IDOR). Dengan memasukkan parameter `id=13` secara langsung ke URL, kami berhasil mem-*bypass* batasan UI dan membongkar profil rahasia tersebut untuk mendapatkan *Recovery Email*-nya.
