# Data Diri

Nama        : Aditya Fathan Naufaldi<br>
NIM         : H1D023076<br>
Shift Lama  : F<br>
Shift Baru  : B

# Penjelasan

Aplikasi Flutter **TokoKita** adalah aplikasi sederhana yang terhubung ke **REST API CodeIgniter 4** untuk melakukan proses:

* Registrasi
* Login
* CRUD Produk

Modul ini menjelaskan setiap file Flutter yang digunakan dalam project TokoKita.

---

# 🔐 Proses Login

Halaman login merupakan pintu masuk utama pengguna sebelum dapat mengakses fitur CRUD produk. Bagian ini menjelaskan proses login **langkah per langkah**, disertai screenshot dan penjelasan kode berdasarkan file `login_page.dart` dan `login_bloc.dart`.

---

# 📸 1. Tampilan Form Login

<img width="381" height="760" alt="Screenshot 2025-11-24 153335" src="https://github.com/user-attachments/assets/e06f0670-b30d-4d60-b463-e5f7edcda830" />

Pada halaman ini terdapat:

- Input **Email**
- Input **Password**
- Tombol **Login**
- Menu **Registrasi**

Semua input dibungkus dalam `Form` dengan validator untuk memastikan data tidak kosong.

---

# 🧩 2. Pengisian Email & Password

### **a. Input Email**

```dart
TextFormField(
  decoration: InputDecoration(labelText: "Email"),
  keyboardType: TextInputType.emailAddress,
  controller: _emailTextboxController,
  validator: (value) {
    if (value!.isEmpty) {
      return 'Email harus diisi';
    }
    return null;
  },
)
````

**Penjelasan:**

* `controller` menyimpan nilai email yang diketik.
* `validator` memastikan email tidak boleh kosong.

---

### **b. Input Password**

```dart
TextFormField(
  decoration: InputDecoration(labelText: "Password"),
  keyboardType: TextInputType.text,
  obscureText: true,
  controller: _passwordTextboxController,
  validator: (value) {
    if (value!.isEmpty) {
      return "Password harus diisi";
    }
    return null;
  },
)
```

**Penjelasan:**

* `obscureText: true` menyembunyikan password.
* `validator` memastikan password tidak kosong.

---

# 🚪 3. Menekan Tombol Login

Kode tombol login:

```dart
ElevatedButton(
  child: Text("Login"),
  onPressed: () {
    var validate = _formKey.currentState!.validate();
    if (validate) {
      if (!_isLoading) _submit();
    }
  },
)
```

**Penjelasan proses:**

1. Tombol ditekan → lakukan validasi form.
2. Jika valid → panggil fungsi `_submit()` untuk memulai login.
3. Cegah double-click dengan `_isLoading`.

---

# 🔄 4. Proses Login (Fungsi `_submit()`)

```dart
void _submit() {
  _formKey.currentState!.save();
  setState(() {
    _isLoading = true;
  });

  LoginBloc.login(
    email: _emailTextboxController.text,
    password: _passwordTextboxController.text,
  ).then(
    (value) async {
      if (value.code == 200) {
        await UserInfo().setToken(value.token.toString());
        await UserInfo().setUserID(int.parse(value.userID.toString()));

        Navigator.pushReplacement(
          context,
          MaterialPageRoute(builder: (context) => ProdukPage()),
        );
      } else {
        showDialog(
          context: context,
          barrierDismissible: false,
          builder: (_) => WarningDialog(
            description: "Login gagal, silahkan coba lagi",
          ),
        );
      }
    },
    onError: (error) {
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (_) => WarningDialog(
          description: "Login gagal, silahkan coba lagi",
        ),
      );
    },
  );

  setState(() {
    _isLoading = false;
  });
}
```

### **Detail Alur Login:**

1. Ambil email & password dari controller.
2. Kirim request login ke API melalui `LoginBloc`.
3. Ketika API merespons:

   * **Jika kode 200** → login sukses

     * Token disimpan di `UserInfo()`
     * UserID disimpan
     * Navigasi ke halaman `ProdukPage`
   * **Jika gagal** → tampilkan popup error.
4. Loading state dikembalikan ke normal.

---

# 🌐 5. Proses API Login (LoginBloc)

File: `login_bloc.dart`

```dart
class LoginBloc {
  static Future<Login> login({String? email, String? password}) async {
    String apiUrl = ApiUrl.login;

    var body = {"email": email, "password": password};

    var response = await Api().post(apiUrl, body);
    var jsonObj = json.decode(response.body);
    return Login.fromJSON(jsonObj);
  }
}
```

### **Penjelasan:**

* `ApiUrl.login` adalah endpoint login CI4.
* `Api().post()` mengirim request HTTP POST.
* Response JSON dikonversi ke model `Login`.
* Hasilnya dikembalikan ke `_submit()` untuk diproses.

---

# ✅ 6. Login Berhasil

Tanda login berhasil:

* User langsung diarahkan ke halaman **ProdukPage**
* Token & UserID tersimpan di storage lokal

---

# ❌ 7. Login Gagal + Popup Error

<img width="382" height="759" alt="Screenshot 2025-11-24 180450" src="https://github.com/user-attachments/assets/938ce43c-beae-408c-8f5c-150ef9cac386" />

Popup muncul jika:

* Email/password salah
* API mengirim kode gagal
* Terjadi error jaringan

Kode popup:

```dart
WarningDialog(description: "Login gagal, silahkan coba lagi");
```

# 📝 Proses Registrasi 

Halaman registrasi digunakan untuk membuat akun baru sebelum pengguna dapat melakukan login.  
Proses ini mencakup validasi form, pengiriman data ke API, dan menampilkan popup berhasil/gagal.

---

# 📸 1. Tampilan Form Registrasi

<img width="384" height="760" alt="Screenshot 2025-11-24 153253" src="https://github.com/user-attachments/assets/fa8a8a6d-aa0a-4104-aa07-2d68a3cf3211" />

Pada halaman registrasi terdapat 4 input data:

- **Nama**
- **Email**
- **Password**
- **Konfirmasi Password**
- Tombol **Registrasi**

Semua input menggunakan validator untuk memastikan data valid sebelum dikirim ke API.

---

# 🧩 2. Input Data Pengguna

## **a. Input Nama**

```dart
TextFormField(
  decoration: InputDecoration(labelText: "Nama"),
  controller: _namaTextboxController,
  validator: (value) {
    if (value!.length < 3) {
      return "Nama harus diisi minimal 3 karakter";
    }
    return null;
  },
)
````

**Penjelasan:**

* Nama minimal 3 karakter.
* Digunakan untuk menyimpan identitas pengguna.

---

## **b. Input Email**

```dart
TextFormField(
  decoration: InputDecoration(labelText: "Email"),
  controller: _emailTextboxController,
  validator: (value) {
    if (value!.isEmpty) {
      return 'Email harus diisi';
    }

    Pattern pattern =
        r'^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$';
    RegExp regex = RegExp(pattern.toString());
    if (!regex.hasMatch(value)) {
      return "Email tidak valid";
    }
    return null;
  },
)
```

**Penjelasan:**

* Validasi email menggunakan regex.
* Email kosong atau format salah → form gagal submit.

---

## **c. Input Password**

```dart
TextFormField(
  decoration: InputDecoration(labelText: "Password"),
  obscureText: true,
  controller: _passwordTextboxController,
  validator: (value) {
    if (value!.length < 6) {
      return "Password harus diisi minimal 6 karakter";
    }
    return null;
  },
)
```

**Penjelasan:**

* Password harus lebih dari 6 karakter.
* `obscureText: true` untuk menyembunyikan input.

---

## **d. Konfirmasi Password**

```dart
TextFormField(
  decoration: InputDecoration(labelText: "Konfirmasi Password"),
  obscureText: true,
  validator: (value) {
    if (value != _passwordTextboxController.text) {
      return "Konfirmasi Password tidak sama";
    }
    return null;
  },
)
```

**Penjelasan:**

* Memastikan password sama dengan konfirmasi password.
* Login hanya dapat dilakukan jika keduanya cocok.

---

# 🚪 3. Menekan Tombol Registrasi

Kode tombol:

```dart
ElevatedButton(
  child: Text("Registrasi"),
  onPressed: () {
    var validate = _formKey.currentState!.validate();
    if (validate) {
      if (!_isLoading) _submit();
    }
  },
)
```

**Penjelasan proses:**

1. Tombol ditekan → form divalidasi.
2. Jika valid → jalankan fungsi `_submit()`.
3. Cegah double-submit dengan `_isLoading`.

---

# 🔄 4. Proses Registrasi (Fungsi `_submit()`)

```dart
void _submit() {
  _formKey.currentState!.save();
  setState(() {
    _isLoading = true;
  });

  RegistrasiBloc.registrasi(
    nama: _namaTextboxController.text,
    email: _emailTextboxController.text,
    password: _passwordTextboxController.text,
  ).then(
    (value) {
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (_) => SuccessDialog(
          description: "Registrasi berhasil, silahkan login",
          okClick: () {
            Navigator.pop(context);
          },
        ),
      );
    },
    onError: (error) {
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (_) => WarningDialog(
          description: "Registrasi gagal, silahkan coba lagi",
        ),
      );
    },
  );

  setState(() {
    _isLoading = false;
  });
}
```

### **Detail Alur Registrasi:**

1. Ambil data dari controller (nama, email, password).
2. Kirim request registrasi ke API melalui `RegistrasiBloc`.
3. Jika API berhasil memproses:

   * Tampilkan popup **SuccessDialog**
   * User diminta kembali ke halaman login
4. Jika API gagal:

   * Tampilkan popup **WarningDialog**
   * User diminta mencoba ulang

---

# 🌐 5. Proses API Registrasi (RegistrasiBloc)

```dart
class RegistrasiBloc {
  static Future<Registrasi> registrasi({
    String? nama,
    String? email,
    String? password,
  }) async {
    String apiUrl = ApiUrl.registrasi;

    var body = {"nama": nama, "email": email, "password": password};

    var response = await Api().post(apiUrl, body);
    var jsonObj = json.decode(response.body);
    return Registrasi.fromJSON(jsonObj);
  }
}
```

### **Penjelasan:**

* Mengambil URL endpoint registrasi.
* Mengirim request POST ke backend CI4.
* Response JSON diubah menjadi model `Registrasi`.
* Hasilnya digunakan untuk menentukan popup yang ditampilkan.

---

# ✅ 6. Registrasi Berhasil

<img width="382" height="760" alt="Screenshot 2025-11-24 153308" src="https://github.com/user-attachments/assets/25fe17f2-b260-4285-a46f-5a4bc4960d8f" />

Popup muncul:

```
Registrasi berhasil, silahkan login
```

User dapat menekan **OK** untuk kembali ke halaman login.

---

# ❌ 7. Registrasi Gagal + Popup Error

Popup gagal muncul apabila:

* Email sudah digunakan
* Validasi server gagal
* Terjadi error koneksi

Kode popup:

```dart
WarningDialog(description: "Registrasi gagal, silahkan coba lagi");
```

# 📦 CRUD Produk

Dokumentasi ini menjelaskan **alur lengkap CRUD Produk** pada aplikasi Flutter yang terhubung dengan backend **CodeIgniter 4 (CI4)**. Semua fungsi CRUD menggunakan `ProdukBloc` sebagai penghubung antara UI Flutter dan REST API CI4.

---

# 🚀 1. GET Produk (Read/List)

<img width="383" height="758" alt="Screenshot 2025-11-24 153702" src="https://github.com/user-attachments/assets/41279a3a-46a5-4ff1-9f1d-5a9380d48fd2" />

Frontend mengambil daftar produk dari endpoint CodeIgniter 4 melalui `ProdukBloc.getProduks()`.

### 🔍 Alur Kerja
1. Flutter memanggil endpoint GET `listProduk`.
2. API mengembalikan JSON berisi daftar produk.
3. JSON di-decode lalu dikonversi menjadi list objek `Produk`.

### 🧩 Code Utama
```dart
static Future<List<Produk>> getProduks() async {
  String apiUrl = ApiUrl.listProduk;
  var response = await Api().get(apiUrl);
  var jsonObj = json.decode(response.body);
  List<dynamic> listProduk = jsonObj['data'];

  return listProduk.map((item) => Produk.fromJSON(item)).toList();
}
````

---

# 📝 2. CREATE Produk (Tambah Produk)

<img width="383" height="761" alt="Screenshot 2025-11-24 153601" src="https://github.com/user-attachments/assets/066cd38a-ad41-451c-9e84-f071e6d7dbd1" />

Fitur ini digunakan pada form tambah produk.

### 🔍 Alur Kerja

1. User mengisi form: kode, nama, harga.
2. Flutter mengirim data ke endpoint POST `createProduk`.
3. Backend CI4 menyimpan data dan mengembalikan status sukses.
4. Jika sukses → redirect kembali ke halaman Produk.

### 🧩 Code Utama

```dart
static Future addProduk({Produk? produk}) async {
  String apiUrl = ApiUrl.createProduk;
  var body = {
    "kode_produk": produk!.kodeProduk,
    "nama_produk": produk.namaProduk,
    "harga": produk.hargaProduk.toString(),
  };

  var response = await Api().post(apiUrl, body);
  return json.decode(response.body)['status'];
}
```

### 🧩 Proses Simpan di Form

```dart
simpan() {
  Produk createProduk = Produk(id: null);
  createProduk.kodeProduk = _kodeProdukTextboxController.text;
  createProduk.namaProduk = _namaProdukTextboxController.text;
  createProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);

  ProdukBloc.addProduk(produk: createProduk).then((value) {
    Navigator.push(context, MaterialPageRoute(builder: (_) => ProdukPage()));
  });
}
```

---

# ✏️ 3. UPDATE Produk (Ubah Produk)

<img width="383" height="756" alt="Screenshot 2025-11-24 181055" src="https://github.com/user-attachments/assets/ca296ee1-e549-4029-bd1c-dc64c953f805" />

Dipanggil saat user membuka form dengan data produk yang ingin di-edit.

### 🔍 Alur Kerja

1. Data produk lama ditampilkan ke form.
2. User melakukan perubahan.
3. Flutter mengirim PUT request ke endpoint `updateProduk/{id}`.
4. Backend menyimpan perubahan.

### 🧩 Code Utama

```dart
static Future updateProduk({required Produk produk}) async {
  String apiUrl = ApiUrl.updateProduk(int.parse(produk.id!));
  var body = {
    "kode_produk": produk.kodeProduk,
    "nama_produk": produk.namaProduk,
    "harga": produk.hargaProduk.toString(),
  };
  var response = await Api().put(apiUrl, jsonEncode(body));
  return json.decode(response.body)['status'];
}
```

### 🧩 Proses Update di Form

<img width="382" height="757" alt="Screenshot 2025-11-24 153726" src="https://github.com/user-attachments/assets/9edc3e52-d566-4b06-9964-105e33663dcd" />

```dart
ubah() {
  Produk updateProduk = Produk(id: widget.produk!.id!);
  updateProduk.kodeProduk = _kodeProdukTextboxController.text;
  updateProduk.namaProduk = _namaProdukTextboxController.text;
  updateProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);

  ProdukBloc.updateProduk(produk: updateProduk).then((value) {
    Navigator.push(context, MaterialPageRoute(builder: (_) => ProdukPage()));
  });
}
```

---

# 🗑️ 4. DELETE Produk

<img width="382" height="761" alt="Screenshot 2025-11-24 153903" src="https://github.com/user-attachments/assets/abfc3102-fbf6-4820-9259-57b0f3a1b5e8" />

Dilakukan dari halaman detail produk.

### 🔍 Alur Kerja

1. User klik tombol DELETE.
2. Muncul dialog konfirmasi.
3. Flutter mengirim DELETE request ke endpoint `deleteProduk/{id}`.
4. Backend menghapus data dan mengembalikan status sukses.
5. Flutter redirect ke halaman Produk.

### 🧩 Code Utama

```dart
static Future<bool> deleteProduk({int? id}) async {
  String apiUrl = ApiUrl.deleteProduk(id!);
  var response = await Api().delete(apiUrl);
  return json.decode(response.body)['data'];
}
```

### 🧩 Aksi Delete di UI

```dart
ProdukBloc.deleteProduk(id: int.parse(widget.produk!.id!)).then((value) {
  Navigator.push(context, MaterialPageRoute(builder: (_) => ProdukPage()));
});
```

---

# 🖥️ 5. UI Pendukung CRUD

### 🧾 Form Tambah/Ubah Produk (`ProdukForm`)

Menangani input user untuk **CREATE** dan **UPDATE**.

* Menampilkan form dengan 3 input:

  * kode produk
  * nama produk
  * harga produk
* Mode otomatis berubah:

  * Jika ada data → mode update
  * Jika kosong → mode tambah

### 📘 Halaman Detail Produk (`ProdukDetail`)

Menampilkan informasi produk + tombol:

* **EDIT** → menuju `ProdukForm`
* **DELETE** → konfirmasi & hapus
