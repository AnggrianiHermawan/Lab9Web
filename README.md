# Praktikum 9: PHP Modular

> **Nama:** Anggriani Hermawan
> **NIM:** 312410175  
> **Mata Kuliah:** Pemrograman Web  
> **Dosen:** Agung Nugroho, S.Kom., M.Kom

---

## Tujuan Praktikum
1. Mahasiswa mampu memahami konsep dasar Modularisasi Program.
2. Mahasiswa mampu memahami konsep dasar Fungsi pada PHP.
3. Mahasiswa mampu membuat program Modular sederhana menggunakan PHP.
4. Mahasiswa mampu mengimplementasikan penggunaan fungsi pada PHP 

## 1. Membuat Folder baru
### A. CONFIG (database.php)
```
<?php
// config/database.php
$host = "localhost";
$user = "root";
$pass = "";
$db   = "latihan1";

$conn = mysqli_connect($host, $user, $pass, $db);
if (!$conn) {
    die("Koneksi gagal: " . mysqli_connect_error());
}
// optional: set charset
mysqli_set_charset($conn, "utf8mb4");
?>
```
Penjelasan:
Kode database.php digunakan untuk menghubungkan aplikasi PHP dengan database MySQL. Bagian atas berisi data koneksi seperti host, username, password, dan nama database. mysqli_connect() digunakan untuk membuat koneksi, lalu dicek apakah berhasil. Jika gagal, program berhenti dan menampilkan pesan error. Terakhir, mysqli_set_charset() mengatur encoding supaya data (termasuk karakter khusus) bisa dibaca dengan benar.

### B. index.php
```
<?php
session_start();
echo "<pre>";
print_r($_SESSION);
echo "</pre>";
require_once "config/database.php";

// simple auth check helper
function is_logged() {
    return isset($_SESSION['user']);
}

$page = isset($_GET['page']) ? $_GET['page'] : 'dashboard';
$page = trim($page, '/'); // sanitize

// include header
include "views/header.php";

// map page to file safely
$file = "modules/". $page .".php";

// allow dashboard and modules only
$allowed = [
    'dashboard',
    'auth/login',
    'auth/logout',
    'user/list','user/add','user/edit','user/delete',
    'data_barang/list','data_barang/tambah','data_barang/ubah','data_barang/hapus'
];

if ($page === 'dashboard') {
    include "views/dashboard.php";
} elseif (in_array($page, $allowed) && file_exists($file)) {
    include $file;
} else {
    echo "<h2>Halaman tidak ditemukan</h2>";
}

include "views/footer.php";
?>
```
Penjelasan:
Kode tersebut berfungsi sebagai router utama yang menentukan halaman mana yang ditampilkan berdasarkan URL. session_start() digunakan untuk mengelola login. Variabel page membaca parameter dari URL, lalu dicek apakah halaman tersebut ada di daftar $allowed. Jika sesuai dan file-nya ada, halaman dimuat. Jika tidak, tampil pesan "Halaman tidak ditemukan". Bagian header dan footer selalu dimasukkan agar tampilan halaman konsisten.
<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 134448" src="https://github.com/user-attachments/assets/2e5966fa-b123-4888-9c9c-16620dd5a393" />

### C. VIEWS
dashboard.php
```
<h2>Dashboard</h2>
<p>Selamat datang di aplikasi modular. Gunakan menu untuk mengelola data.</p>
```
<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 134500" src="https://github.com/user-attachments/assets/25956b34-792f-4bbb-962d-d8afa73eabec" />
Penjelasan:
