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
- dashboard.php
```
<h2>Dashboard</h2>
<p>Selamat datang di aplikasi modular. Gunakan menu untuk mengelola data.</p>
```
Penjelasan:
Kode tersebut menampilkan halaman Dashboard dengan judul ```<h2>``` dan pesan sambutan ```<p>``` yang memberi informasi bahwa user sudah berada di halaman utama aplikasi dan dapat menggunakan menu untuk mengelola data.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 134500" src="https://github.com/user-attachments/assets/25956b34-792f-4bbb-962d-d8afa73eabec" />

- footer.php
```
    </main>
    <footer>
        <p>© <?=date('Y');?> - Modular PHP System</p>
    </footer>
</div>
</body>
</html>
```
Penjelasan:
Bagian kode ini adalah penutup tampilan halaman. Tag ```<footer>``` menampilkan teks copyright otomatis dari tahun saat ini menggunakan date('Y'). Setelah itu ditutup dengan tag ```</main>```, ```</div>```, lalu penutup ```</body>``` dan ```</html>``` untuk mengakhiri struktur halaman HTML.

- header.php
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Aplikasi Modular</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
<div class="container">
    <header>
        <h1>Aplikasi Modular - PHP & MySQL</h1>
        <nav>
            <a href="index.php">Dashboard</a>
            <a href="index.php?page=data_barang/list">Data Barang</a>
            <a href="index.php?page=user/list">User</a>
            <?php if(isset($_SESSION['user'])): ?>
                <span class="right">Halo, <?=htmlspecialchars($_SESSION['user']['fullname']);?> |
                  <a href="index.php?page=auth/logout">Logout</a>
                </span>
            <?php else: ?>
                <a class="right" href="index.php?page=auth/login">Login</a>
            <?php endif; ?>
        </nav>
    </header>
    <main>
```
Penjelasan:
Kode ini adalah bagian header halaman web. Di dalamnya terdapat deklarasi HTML, judul halaman, serta pemanggilan CSS untuk styling. Elemen ```<header>``` menampilkan nama aplikasi dan menu navigasi. Menu berubah otomatis berdasarkan status login pengguna—jika sudah login, tampil nama pengguna dan tombol logout; jika belum login, tampil tombol login. Bagian terakhir ```\<main>``` menandai awal konten utama halaman.

### D. MODULES
Auth
- login.php
```
<?php
// modules/auth/login.php
if (isset($_SESSION['user'])) {
    header('Location: index.php');
    exit;
}
$error = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = mysqli_real_escape_string($conn, $_POST['username']);
    $pass = $_POST['password'];

    $sql = "SELECT * FROM users WHERE username = '{$username}' LIMIT 1";
    $res = mysqli_query($conn, $sql);
    if ($res && mysqli_num_rows($res) === 1) {
        $user = mysqli_fetch_assoc($res);
        if (password_verify($pass, $user['password'])) {
            // set session
            $_SESSION['user'] = [
                'id' => $user['id'],
                'username' => $user['username'],
                'fullname' => $user['fullname'],
                'role' => $user['role']
            ];
            header('Location: index.php');
            exit;
        } else {
            $error = "Username atau password salah.";
        }
    } else {
        $error = "Username tidak ditemukan.";
    }
}
?>

<h2>Login</h2>
<?php if($error): ?><div class="alert"><?=htmlspecialchars($error)?></div><?php endif; ?>
<form method="post" action="index.php?page=auth/login">
    <label>Username</label><br>
    <input type="text" name="username" required><br>
    <label>Password</label><br>
    <input type="password" name="password" required><br><br>
    <button type="submit">Login</button>
</form>
```
Penjelasan:
Kode ini menangani proses login pengguna. Pertama, sistem mengecek apakah user sudah login—jika ya, langsung diarahkan kembali ke halaman utama. Jika form login dikirim (method POST), aplikasi mengambil username dan password, lalu mencocokkannya dengan data di database. Jika username ditemukan dan password sesuai menggunakan password_verify(), maka data user disimpan ke session untuk menandakan bahwa ia sudah login. Jika gagal, pesan error ditampilkan. Setelah login berhasil, user diarahkan kembali ke halaman utama aplikasi.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 153707" src="https://github.com/user-attachments/assets/8df184d0-0eea-4931-9605-0becdb218cbc" />

- logout.php
```
<?php
// modules/auth/logout.php
session_destroy();
header('Location: index.php');
exit;
```
Penjelasan:
Kode ini digunakan untuk proses logout. session_destroy() menghapus semua data session sehingga pengguna dianggap tidak lagi login. Setelah itu, halaman diarahkan kembali ke index.php menggunakan header() lalu program dihentikan dengan exit agar tidak ada kode lain yang berjalan.

Data_barang
- hapus.php
```
<?php
$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
if ($id) {
    // optional: delete gambar file
    $res = mysqli_query($conn, "SELECT gambar FROM data_barang WHERE id_barang={$id}");
    if ($res && mysqli_num_rows($res)) {
        $r = mysqli_fetch_assoc($res);
        if (!empty($r['gambar']) && file_exists($r['gambar'])) {
            @unlink($r['gambar']);
        }
    }
    mysqli_query($conn, "DELETE FROM data_barang WHERE id_barang={$id}");
}
header('Location: index.php?page=data_barang/list');
exit;
```
Penjelasan:
Kode ini digunakan untuk menghapus data barang berdasarkan ID yang dikirim melalui URL. Pertama, sistem mengecek apakah ID valid. Jika ada, program mencari data gambarnya di database dan menghapus file gambarnya dari folder jika masih ada. Setelah itu, baris data barang dihapus dari tabel data_barang. Terakhir, pengguna diarahkan kembali ke halaman daftar barang.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 154408" src="https://github.com/user-attachments/assets/73fb3c7f-b2e6-4395-92e4-61a0e0b5aac4" />

- list.php
```
<?php
require_once __DIR__ . "/../../config/database.php";

$sql = "SELECT * FROM data_barang ORDER BY id_barang DESC";
$res = mysqli_query($conn, $sql);
?>
<h2>Data Barang</h2>
<a href="index.php?page=data_barang/tambah">Tambah Barang</a>
<table>
<tr><th>Gambar</th><th>Nama</th><th>Kategori</th><th>Harga Beli</th><th>Harga Jual</th><th>Stok</th><th>Aksi</th></tr>
<?php while ($r = mysqli_fetch_assoc($res)): ?>
<tr>
    <td>
      <?php if(!empty($r['gambar']) && file_exists(__DIR__ . "/../../" . $r['gambar'])): ?>
        <img src="<?= $r['gambar'] ?>" style="max-width:100px">
      <?php else: ?>
        -
      <?php endif; ?>
    </td>
    <td><?= htmlspecialchars($r['nama']) ?></td>
    <td><?= htmlspecialchars($r['kategori']) ?></td>
    <td><?= $r['harga_beli'] ?></td>
    <td><?= $r['harga_jual'] ?></td>
    <td><?= $r['stok'] ?></td>
    <td>
        <a href="index.php?page=data_barang/ubah&id=<?= $r['id_barang'] ?>">Edit</a> |
        <a href="index.php?page=data_barang/hapus&id=<?= $r['id_barang'] ?>" onclick="return confirm('Hapus data ini?')">Hapus</a>
    </td>
</tr>
<?php endwhile; ?>
</table>
```
Penjelasan:
Kode ini menampilkan daftar data barang dari database. Pertama, file koneksi database di-load, lalu data diambil menggunakan query SELECT * FROM data_barang ORDER BY id_barang DESC. Data ditampilkan dalam bentuk tabel, termasuk gambar jika tersedia. Pada setiap baris terdapat tombol Edit dan Hapus untuk mengelola data. Jika gambar tidak ditemukan, tanda strip (-) ditampilkan sebagai pengganti.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 154320" src="https://github.com/user-attachments/assets/61b3709c-9e6f-49db-a00f-2908556f685c" />

- tambah.php
```
<?php
$err = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $nama = mysqli_real_escape_string($conn, $_POST['nama']);
    $kategori = mysqli_real_escape_string($conn, $_POST['kategori']);
    $harga_beli = (float)$_POST['harga_beli'];
    $harga_jual = (float)$_POST['harga_jual'];
    $stok = (int)$_POST['stok'];

    $gambarPath = '';
    if (!empty($_FILES['file_gambar']['name'])) {
        $fn = basename($_FILES['file_gambar']['name']);
        $target = 'assets/uploads/'.time()."_".$fn;
        if (move_uploaded_file($_FILES['file_gambar']['tmp_name'], $target)) {
            $gambarPath = $target;
        }
    }

    $sql = "INSERT INTO data_barang (kategori,nama,gambar,harga_beli,harga_jual,stok)
            VALUES ('$kategori','$nama','$gambarPath',$harga_beli,$harga_jual,$stok)";
    if (mysqli_query($conn,$sql)) {
        header('Location: index.php?page=data_barang/list');
        exit;
    } else {
        $err = mysqli_error($conn);
    }
}
?>
<h2>Tambah Barang</h2>
<?php if($err) echo "<div class='alert'>$err</div>"; ?>
<form method="post" enctype="multipart/form-data" action="index.php?page=data_barang/tambah">
    <label>Nama</label><br><input type="text" name="nama" required><br>
    <label>Kategori</label><br><input type="text" name="kategori" required><br>
    <label>Harga Beli</label><br><input type="number" name="harga_beli" required><br>
    <label>Harga Jual</label><br><input type="number" name="harga_jual" required><br>
    <label>Stok</label><br><input type="number" name="stok" required><br>
    <label>Gambar</label><br><input type="file" name="file_gambar"><br><br>
    <button type="submit">Simpan</button>
</form>
```
Penjelasan:
Kode ini digunakan untuk menambah data barang baru ke database. Saat form disubmit (metode POST), sistem mengambil input seperti nama, kategori, harga, stok, lalu mengecek apakah ada file gambar yang diunggah. Jika ada, file dipindahkan ke folder assets/uploads dan path-nya disimpan. Setelah itu, semua data dimasukkan ke tabel data_barang menggunakan query INSERT. Jika berhasil, user diarahkan kembali ke daftar barang; jika gagal, pesan error ditampilkan.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 154332" src="https://github.com/user-attachments/assets/20cb07f4-a217-44c4-8b4f-ff2fd5e486b0" />

- ubah.php
```
<?php
$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
if (!$id) { echo "<p>ID tidak valid</p>"; return; }

$err = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $id = (int)$_POST['id'];
    $nama = mysqli_real_escape_string($conn, $_POST['nama']);
    $kategori = mysqli_real_escape_string($conn, $_POST['kategori']);
    $harga_beli = (float)$_POST['harga_beli'];
    $harga_jual = (float)$_POST['harga_jual'];
    $stok = (int)$_POST['stok'];

    $gambarPath = '';
    if (!empty($_FILES['file_gambar']['name'])) {
        $fn = basename($_FILES['file_gambar']['name']);
        $target = 'assets/uploads/'.time()."_".$fn;
        if (move_uploaded_file($_FILES['file_gambar']['tmp_name'], $target)) {
            $gambarPath = $target;
        }
    }

    $sql = "UPDATE data_barang SET nama='$nama', kategori='$kategori', harga_beli=$harga_beli, harga_jual=$harga_jual, stok=$stok";
    if ($gambarPath) $sql .= ", gambar='$gambarPath'";
    $sql .= " WHERE id_barang=$id";
    if (mysqli_query($conn,$sql)) {
        header('Location: index.php?page=data_barang/list');
        exit;
    } else {
        $err = mysqli_error($conn);
    }
}

$res = mysqli_query($conn, "SELECT * FROM data_barang WHERE id_barang={$id}");
if (!$res || mysqli_num_rows($res) == 0) { echo "<p>Data tidak ditemukan</p>"; return; }
$data = mysqli_fetch_assoc($res);
?>
<h2>Ubah Barang</h2>
<?php if($err) echo "<div class='alert'>$err</div>"; ?>
<form method="post" enctype="multipart/form-data" action="index.php?page=data_barang/ubah&id=<?=$id?>">
    <input type="hidden" name="id" value="<?=$id?>">
    <label>Nama</label><br><input type="text" name="nama" value="<?=htmlspecialchars($data['nama'])?>" required><br>
    <label>Kategori</label><br><input type="text" name="kategori" value="<?=htmlspecialchars($data['kategori'])?>" required><br>
    <label>Harga Beli</label><br><input type="number" name="harga_beli" value="<?= $data['harga_beli'] ?>" required><br>
    <label>Harga Jual</label><br><input type="number" name="harga_jual" value="<?= $data['harga_jual'] ?>" required><br>
    <label>Stok</label><br><input type="number" name="stok" value="<?= $data['stok'] ?>" required><br>
    <label>Gambar (kosongkan jika tidak ingin ganti)</label><br><input type="file" name="file_gambar"><br>
    <?php if(!empty($data['gambar']) && file_exists($data['gambar'])): ?>
      <img src="<?= $data['gambar'] ?>" style="max-width:120px"><br>
    <?php endif; ?>
    <br><button type="submit">Update</button>
</form>
```
Penjelasan:
Kode ini digunakan untuk mengedit data barang yang sudah ada. Pertama, sistem mengambil ID barang dari URL lalu memuat datanya dari database. Jika form dikirim (POST), data yang diinput akan digunakan untuk memperbarui tabel data_barang. Jika user mengunggah gambar baru, gambar tersebut disimpan dan path-nya ikut diperbarui. Jika tidak, gambar lama tetap digunakan. Setelah update berhasil, user dikembalikan ke halaman daftar barang.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 154348" src="https://github.com/user-attachments/assets/20e3bfa1-e20b-4a7f-942e-87367af3f320" />

User
- add.php
```
<?php
// modules/user/add.php
$err = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = mysqli_real_escape_string($conn, $_POST['username']);
    $fullname = mysqli_real_escape_string($conn, $_POST['fullname']);
    $password = password_hash($_POST['password'], PASSWORD_DEFAULT);
    $role = mysqli_real_escape_string($conn, $_POST['role']);

    $sql = "INSERT INTO users (username,password,fullname,role) VALUES ('$username','$password','$fullname','$role')";
    if (mysqli_query($conn, $sql)) {
        header('Location: index.php?page=user/list');
        exit;
    } else {
        $err = "Gagal menambah user: ".mysqli_error($conn);
    }
}
?>
<h2>Tambah User</h2>
<?php if($err) echo "<div class='alert'>$err</div>"; ?>
<form method="post" action="index.php?page=user/add">
    <label>Username</label><br>
    <input type="text" name="username" required><br>
    <label>Nama Lengkap</label><br>
    <input type="text" name="fullname" required><br>
    <label>Password</label><br>
    <input type="password" name="password" required><br>
    <label>Role</label><br>
    <select name="role">
        <option value="user">user</option>
        <option value="admin">admin</option>
    </select><br><br>
    <button type="submit">Simpan</button>
</form>
```
Penjelasan:
Form ini mengambil data user dari form, mengamankan input dan password, menyimpannya ke database, lalu redirect ke daftar user atau tampilkan error jika gagal.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 154929" src="https://github.com/user-attachments/assets/830cef81-ec95-4036-b00c-493da12896e7" />

- delete.php
```
<?php
// modules/user/delete.php
$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
if ($id) {
    mysqli_query($conn, "DELETE FROM users WHERE id={$id}");
}
header('Location: index.php?page=user/list');
exit;
```
Penjelasan:
Kode ini menghapus user berdasarkan id dari URL, lalu langsung redirect ke daftar user.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 155016" src="https://github.com/user-attachments/assets/cc801e07-cad0-4bb6-96f9-df7efa90d469" />

- edit.php
```
<?php
// modules/user/edit.php
$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
$err = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $id = (int)$_POST['id'];
    $fullname = mysqli_real_escape_string($conn, $_POST['fullname']);
    $role = mysqli_real_escape_string($conn, $_POST['role']);
    $sql = "UPDATE users SET fullname='{$fullname}', role='{$role}' WHERE id={$id}";
    if (mysqli_query($conn, $sql)) {
        header('Location: index.php?page=user/list');
        exit;
    } else {
        $err = mysqli_error($conn);
    }
}

$res = mysqli_query($conn, "SELECT * FROM users WHERE id={$id}");
if (!$res || mysqli_num_rows($res) === 0) {
    echo "<p>User tidak ditemukan.</p>";
    return;
}
$user = mysqli_fetch_assoc($res);
?>
<h2>Edit User</h2>
<?php if($err) echo "<div class='alert'>$err</div>"; ?>
<form method="post" action="index.php?page=user/edit">
    <input type="hidden" name="id" value="<?= $user['id'] ?>">
    <label>Username</label><br>
    <input type="text" value="<?= htmlspecialchars($user['username']) ?>" disabled><br>
    <label>Nama Lengkap</label><br>
    <input type="text" name="fullname" value="<?= htmlspecialchars($user['fullname']) ?>" required><br>
    <label>Role</label><br>
    <select name="role">
        <option value="user" <?= $user['role']=='user' ? 'selected':'' ?>>user</option>
        <option value="admin" <?= $user['role']=='admin' ? 'selected':'' ?>>admin</option>
    </select><br><br>
    <button type="submit">Update</button>
</form>
```
Penjelasan:
Kode ini mengambil data user berdasarkan id, menampilkan form untuk edit nama lengkap dan role, menyimpan perubahan ke database, lalu redirect ke daftar user atau tampilkan error jika gagal.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 154947" src="https://github.com/user-attachments/assets/ec5efd2f-730b-4d31-9a91-2da5f50ecfe1" />

- list.php
```
<?php
// Pastikan koneksi sudah ada
require_once "config/database.php";

// Query data
$sql = "SELECT id, username, fullname, role FROM users";
$res = mysqli_query($conn, $sql);

?>

<h2>Daftar User</h2>
<a class="btn" href="index.php?page=user/add">Tambah User</a>

<table>
    <tr>
        <th>ID</th>
        <th>Username</th>
        <th>Nama Lengkap</th>
        <th>Role</th>
        <th>Aksi</th>
    </tr>

    <?php if (mysqli_num_rows($res) > 0): ?>
        <?php while ($row = mysqli_fetch_assoc($res)): ?>
        <tr>
            <td><?= $row['id'] ?></td>
            <td><?= htmlspecialchars($row['username']) ?></td>
            <td><?= htmlspecialchars($row['fullname']) ?></td>
            <td><?= htmlspecialchars($row['role']) ?></td>
            <td>
                <a href="index.php?page=user/edit&id=<?= $row['id'] ?>">Edit</a> |
                <a href="index.php?page=user/delete&id=<?= $row['id'] ?>" onclick="return confirm('Yakin hapus user ini?')">Hapus</a>
            </td>
        </tr>
        <?php endwhile; ?>
    <?php else: ?>
        <tr><td colspan="5">Tidak ada data user.</td></tr>
    <?php endif; ?>

</table>
```
Penjelasan:
Kode ini menampilkan daftar semua user dari database dalam tabel, dengan tombol untuk tambah, edit, atau hapus user.

<img width="1920" height="1080" alt="Cuplikan layar 2025-11-24 154915" src="https://github.com/user-attachments/assets/5ccabec8-c244-4695-918e-b3b51a5bb27e" />

### E. ASSETS
CSS
- style.css
```
/* assets/css/style.css */
body{ font-family: Arial, sans-serif; background:#f4f4f4; color:#333; }
.container{ width: 95%; max-width:1100px; margin:20px auto; background:#fff; padding:20px; box-shadow: 0 0 6px rgba(0,0,0,0.1);}
header h1{ margin:0; }
nav{ margin-top:10px; }
nav a{ margin-right:10px; text-decoration:none; color:#0073aa; }
nav .right{ float:right; }
table{ width:100%; border-collapse:collapse; margin-top:15px; }
table, th, td{ border:1px solid #ddd; }
th, td{ padding:8px; text-align:left; }
.alert{ background:#ffdddd; padding:8px; margin-bottom:10px; border:1px solid #ffbcbc; }
footer{ margin-top:20px; text-align:center; color:#666; font-size:14px; }
img{ max-width:100%; height:auto; }
b/* ======= GLOBAL ======= */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: "Segoe UI", Arial, sans-serif;
}

body {
    background: #f5f7fa;
    color: #333;
}

/* ======= LAYOUT ======= */
.container {
    width: 90%;
    max-width: 1200px;
    margin: 30px auto;
    background: #fff;
    padding: 25px;
    border-radius: 10px;
    box-shadow: 0px 5px 20px rgba(0,0,0,0.08);
}

/* ======= HEADER ======= */
header h1 {
    font-size: 26px;
    margin-bottom: 15px;
    font-weight: 600;
    color: #007bff;
}

/* ======= NAVIGATION ======= */
nav {
    display: flex;
    gap: 15px;
    background: #007bff;
    padding: 10px;
    border-radius: 8px;
}

nav a {
    color: white;
    text-decoration: none;
    font-size: 15px;
    padding: 8px 12px;
    background: rgba(255,255,255,0.15);
    border-radius: 6px;
    transition: 0.2s;
}

nav a:hover {
    background: rgba(255,255,255,0.3);
}

/* ======= TABLE ======= */
table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
}

table th {
    background: #007bff;
    color: white;
}

table th, table td {
    border: 1px solid #ddd;
    padding: 10px;
    text-align: center;
}

table tr:nth-child(even) {
    background: #f1f5ff;
}

/* ===== BUTTON ===== */
button, input[type="submit"], a.btn {
    background: #007bff;
    padding: 10px 15px;
    color: white;
    text-decoration: none;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: 0.3s;
}

button:hover, input[type="submit"]:hover, a.btn:hover {
    background: #0056d2;
}

/* ===== FORM ===== */
input, select {
    width: 100%;
    padding: 10px;
    margin-top: 6px;
    border: 1px solid #ccc;
    border-radius: 6px;
}

/* ===== FOOTER ===== */
footer {
    margin-top: 25px;
    text-align: center;
    padding: 10px;
    font-size: 14px;
    color: #666;
}
utton, input[type="submit"]{ background:#0073aa; color:#fff; padding:8px 12px; border:none; border-radius:4px; cursor:pointer;}
input[type="text"], input[type="password"], input[type="number"], select{ padding:6px; width:100%; max-width:400px; margin-bottom:8px;}
```
Penjelasan:
CSS ini mengatur tampilan halaman web:
- Global: font, margin, padding, box-sizing, warna background dan teks.
- Container: lebar maksimal, padding, latar putih, bayangan, border-radius.
- Header & Nav: ukuran judul, warna, navigasi fleksibel dengan link yang diberi efek hover.
- Table: border, padding, warna header biru, baris genap beda warna, teks rata tengah.
- Form & Input: padding, border, border-radius, lebar penuh untuk input/select.
- Button & Link: warna biru, teks putih, efek hover, border-radius, cursor pointer.
- Alert & Footer: background/warna teks untuk alert, footer center dengan warna abu-abu.
Singkatnya: CSS ini membuat halaman user list, form, dan tombol terlihat modern, rapi, dan responsif.
